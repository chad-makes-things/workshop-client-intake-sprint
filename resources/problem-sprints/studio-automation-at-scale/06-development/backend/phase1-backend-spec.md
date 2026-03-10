# Phase 1 Backend Specification

**Backend Engineer:** Development Team — Backend Role
**Sprint source:** Studio Automation at Scale
**Date:** 2026-03-10
**Scope:** Phase 1 pilot — 15 Gen4 practices, email + Teams intake, 8-template library

---

## Overview

The backend is the core of the Studio Agent system. It handles: email ingestion, Teams message handling, Claude API conversation management, Canva asset generation, CD review notification, Weave patient email scheduling, Gelato print order submission, and the studio analytics data pipeline. The frontend (dashboard) is a separate service consuming the internal API.

**Runtime:** Node.js 20 (LTS) with TypeScript
**Framework:** Fastify (preferred over Express for performance and schema validation)
**Database ORM:** Drizzle ORM with PostgreSQL
**Job queue:** BullMQ with Redis for async generation jobs
**Hosting:** Azure App Service

---

## Module: Email Adapter

### Inbound Email Processing

**Trigger:** Microsoft Graph API change notification webhook — fires when a new email arrives in `studio@sgadental.com`

**Implementation:**

```typescript
// POST /webhooks/email/inbound
export async function handleInboundEmail(
  notification: GraphChangeNotification
): Promise<void> {
  // 1. Fetch the email message from Graph API
  const message = await graphClient
    .api(`/users/studio@sgadental.com/messages/${notification.resourceData.id}`)
    .select('id,subject,from,body,toRecipients,receivedDateTime,conversationId,conversationIndex')
    .get();

  // 2. Identify practice from sender email domain
  const senderEmail = message.from.emailAddress.address;
  const domain = senderEmail.split('@')[1];
  const practice = await practiceRepo.findByEmailDomain(domain);

  // 3. Create or continue request record
  const existingRequest = await requestRepo.findByConversationId(message.conversationId);

  if (existingRequest) {
    // Continue existing conversation
    await conversationRepo.append({
      requestId: existingRequest.id,
      role: 'user',
      content: stripHtml(message.body.content),
      channel: 'email',
    });
    await agentOrchestrator.continue(existingRequest.id);
  } else {
    // New request
    const request = await requestRepo.create({
      practiceId: practice?.id ?? null,
      channel: 'email',
      channelThreadId: message.conversationId,
      senderEmail,
      status: 'intake',
    });
    await conversationRepo.append({
      requestId: request.id,
      role: 'user',
      content: stripHtml(message.body.content),
      channel: 'email',
    });
    await agentOrchestrator.start(request.id, { unknownPractice: !practice });
  }
}
```

**Email reply:**

```typescript
export async function sendEmailReply(
  request: Request,
  replyContent: string,
  attachments?: EmailAttachment[]
): Promise<void> {
  await graphClient
    .api(`/users/studio@sgadental.com/messages/${originalMessageId}/reply`)
    .post({
      message: {
        body: {
          contentType: 'text',
          content: replyContent,
        },
        attachments: attachments?.map(a => ({
          '@odata.type': '#microsoft.graph.fileAttachment',
          name: a.filename,
          contentType: a.mimeType,
          contentBytes: a.base64Content,
        })) ?? [],
      },
    });
}
```

**Inline preview images:** Convert Canva preview JPGs to base64 and embed in email body as `<img src="data:image/jpeg;base64,...">` — ensures previews display without requiring the recipient to click "Display Images."

---

## Module: Teams Adapter

### Inbound Teams Message

**Trigger:** Bot Framework webhook POST to `/webhooks/teams/message`

```typescript
export async function handleTeamsActivity(
  activity: Activity
): Promise<void> {
  if (activity.type !== ActivityTypes.Message) return;

  const senderEmail = activity.from.aadObjectId
    ? await resolveTeamsUserEmail(activity.from.aadObjectId)
    : null;

  // Same practice lookup + request routing as email adapter
  const practice = senderEmail
    ? await practiceRepo.findByEmailDomain(senderEmail.split('@')[1])
    : null;

  // Thread ID = conversation.id (unique per 1:1 conversation or channel thread)
  const threadId = activity.conversation.id;

  // ... same create/continue logic as email
}
```

**CD Review Adaptive Card:**

```typescript
export async function sendCDReviewCard(
  request: Request,
  previews: PreviewImage[]
): Promise<void> {
  const card = {
    type: 'AdaptiveCard',
    version: '1.4',
    body: [
      {
        type: 'TextBlock',
        text: `${request.practice.name} — ${request.collateralTypes.join(', ')}`,
        weight: 'Bolder',
        size: 'Medium',
      },
      {
        type: 'TextBlock',
        text: `Requested by: ${request.primaryContactName}`,
        color: 'Accent',
      },
      {
        type: 'ColumnSet',
        columns: previews.map(p => ({
          type: 'Column',
          width: 'auto',
          items: [{
            type: 'Image',
            url: p.thumbnailUrl, // Azure Blob Storage URL (not base64 for Adaptive Cards)
            size: 'Medium',
          }],
        })),
      },
    ],
    actions: [
      { type: 'Action.Submit', title: 'Approve All', data: { action: 'approve', requestId: request.id } },
      { type: 'Action.Submit', title: 'Request Change', data: { action: 'change', requestId: request.id } },
      { type: 'Action.Submit', title: 'Reject', style: 'destructive', data: { action: 'reject', requestId: request.id } },
    ],
  };

  await botClient.sendActivity(cdTeamsConversationRef, { attachments: [CardFactory.adaptiveCard(card)] });
}
```

---

## Module: Agent Orchestrator

### Claude API Integration

```typescript
const SYSTEM_PROMPT = `
You are the SGA Studio Agent — the creative request handler for SGA Dental Partners.
You help practice managers request, preview, and receive on-brand creative assets.

RULES:
1. Never ask more than 3 clarifying questions in a single reply.
2. If the brief is complete, confirm and proceed — don't ask unnecessary questions.
3. When a request implies print (flyer, brochure, poster, handout), include a medium
   challenge in your reply: describe the digital equivalent with specific benefits
   (reach, cost, speed), then offer both options.
4. Address the practice manager by first name. Address the practice by name.
5. Always sign off as "Studio Agent" — never as a person's name.
6. For text revisions: confirm what you're changing and regenerate.
7. For layout/structural changes: escalate to the human designer queue.

BRIEF FIELDS TO GATHER:
- collateral_types: what formats are needed (email_header, tv_slide, flyer_4x6, etc.)
- offer: what is the offer or message
- cta: call to action text
- audience: existing_patients | new_patients | both | referring_doctors
- deadline: when materials are needed
- doctor_featured: specific doctor or practice-branded

PRACTICE CONTEXT:
{{practiceContext}}

CONVERSATION HISTORY:
{{conversationHistory}}

Respond with a JSON object:
{
  "replyText": "...",
  "action": "ask_questions | medium_challenge | generate | escalate | complete",
  "briefUpdate": { ... },
  "escalationReason": null
}
`;

export async function runAgentTurn(
  requestId: string,
): Promise<AgentDecision> {
  const request = await requestRepo.findById(requestId);
  const practice = await practiceRepo.findById(request.practiceId);
  const conversations = await conversationRepo.findByRequestId(requestId);

  const prompt = SYSTEM_PROMPT
    .replace('{{practiceContext}}', buildPracticeContext(practice))
    .replace('{{conversationHistory}}', buildConversationHistory(conversations));

  const response = await anthropic.messages.create({
    model: 'claude-sonnet-4-6',
    max_tokens: 1024,
    messages: [{ role: 'user', content: prompt }],
  });

  const decision = JSON.parse(response.content[0].text) as AgentDecision;

  // Persist the agent's reply
  await conversationRepo.append({
    requestId,
    role: 'agent',
    content: decision.replyText,
    channel: request.channel,
  });

  // Update brief if new fields extracted
  if (decision.briefUpdate) {
    await requestRepo.updateBrief(requestId, decision.briefUpdate);
  }

  return decision;
}
```

### Brief Completeness Check

```typescript
export function isBriefComplete(brief: Partial<Brief>): boolean {
  const required = ['collateral_types', 'offer', 'audience'];
  return required.every(field => brief[field] && brief[field].length > 0);
}
```

---

## Module: Canva Integration Service

```typescript
export async function generateAssets(
  request: Request,
  practice: Practice,
): Promise<GeneratedAsset[]> {
  const assets: GeneratedAsset[] = [];

  for (const collateralType of request.brief.collateral_types) {
    const template = await templateRepo.findByType(collateralType);
    if (!template) throw new EscalationError(`No template for ${collateralType}`);

    // Call Canva Connect API to create a design from template
    const design = await canvaClient.createDesign({
      assetId: template.canvaTemplateId,
      title: `${practice.name} — ${collateralType} — ${new Date().toISOString()}`,
    });

    // Populate variable fields
    const variables = buildVariables(template, request.brief, practice);
    await canvaClient.updateDesignContent(design.id, variables);

    // Export preview (JPG 800px)
    const previewJob = await canvaClient.createExportJob({
      designId: design.id,
      format: 'jpg',
      exportQuality: 'regular',
    });
    const preview = await pollExportJob(previewJob.job.id);

    // Upload preview to Azure Blob Storage for Teams Adaptive Card URL
    const previewUrl = await blobStorage.upload(
      preview.urls[0],
      `previews/${request.id}/${collateralType}.jpg`
    );

    assets.push({
      collateralType,
      canvaDesignId: design.id,
      previewUrl,
      previewBase64: await urlToBase64(preview.urls[0]), // for email inline
    });
  }

  return assets;
}

export async function exportPrintReady(
  canvaDesignId: string,
  collateralType: string
): Promise<string> {
  // Export PDF/X-1a for print (verify in spike SP-1)
  // Fallback: export as PNG at 300 DPI if PDF/X-1a not supported
  const exportJob = await canvaClient.createExportJob({
    designId: canvaDesignId,
    format: 'pdf',
    exportQuality: 'pro',
  });
  const result = await pollExportJob(exportJob.job.id);
  return blobStorage.upload(result.urls[0], `print-ready/${canvaDesignId}.pdf`);
}
```

---

## Module: Deployment Service

### Weave Patient Email Scheduling

```typescript
export async function schedulePatientEmail(
  practice: Practice,
  assetUrl: string,
  sendDate: Date,
): Promise<WeaveScheduleResult> {
  if (!practice.weaveAccountId) {
    throw new IntegrationUnavailableError('Weave not configured for this practice');
  }

  // Weave API: upload asset and schedule send
  // NOTE: Exact Weave API shape to be confirmed in spike SP-3
  const result = await weaveClient.scheduleMessage({
    accountId: practice.weaveAccountId,
    message: {
      type: 'email',
      subject: 'Special offer from your dental practice',
      asset_url: assetUrl,
      audience: 'active_patients',
      send_at: sendDate.toISOString(),
    },
  });

  return result;
}
```

### Gelato Print Order

```typescript
export async function submitPrintOrder(
  practice: Practice,
  printFileUrl: string,
  spec: PrintSpec,
  quantity: number,
): Promise<GelatoOrderResult> {
  const order = await gelatoClient.orders.create({
    orderReferenceId: `sga-${Date.now()}`,
    customerReferenceId: practice.id,
    currency: 'USD',
    items: [{
      itemReferenceId: `item-1`,
      productUid: spec.gelatoProductUid, // pre-configured per template type
      files: [{ type: 'default', url: printFileUrl }],
      quantity,
    }],
    shipmentMethodUid: 'express',
    shippingAddress: {
      firstName: practice.primaryContactName.split(' ')[0],
      lastName: practice.primaryContactName.split(' ')[1] ?? '',
      addressLine1: practice.addressLine1,
      city: practice.addressCity,
      state: practice.addressState,
      postCode: practice.addressZip,
      country: 'US',
      email: practice.primaryContactEmail,
      phone: practice.phone,
    },
  });

  return order;
}
```

---

## Module: Analytics Event Pipeline

```typescript
// Called at each key state transition
export async function trackEvent(
  event: StudioEvent,
  properties: Record<string, unknown>
): Promise<void> {
  await posthog.capture({
    distinctId: properties.practiceId as string,
    event,
    properties: {
      ...properties,
      timestamp: new Date().toISOString(),
      environment: process.env.NODE_ENV,
    },
  });
}

// Key events
export const StudioEvents = {
  REQUEST_CREATED: 'studio_request_created',
  MEDIUM_CHALLENGE_SENT: 'studio_medium_challenge_sent',
  MEDIUM_CHALLENGE_ACCEPTED: 'studio_medium_challenge_accepted',
  GENERATION_STARTED: 'studio_generation_started',
  GENERATION_COMPLETED: 'studio_generation_completed',
  PRACTICE_APPROVED: 'studio_practice_approved',
  CD_REVIEW_STARTED: 'studio_cd_review_started',
  CD_REVIEW_COMPLETED: 'studio_cd_review_completed',
  ASSET_DEPLOYED: 'studio_asset_deployed',
  PRINT_ORDER_PLACED: 'studio_print_order_placed',
  REQUEST_ESCALATED: 'studio_request_escalated',
} as const;
```

---

## Error Handling

| Error | Handling |
|-------|----------|
| Practice not found for email domain | Agent replies asking for practice name; creates unlinked request |
| Canva API rate limit / timeout | Retry up to 3× with exponential backoff (2s, 4s, 8s). If still failing after 3×, escalate to human queue with message to practice: "I'm having trouble generating your assets right now — our designer will take over." |
| Claude API error | Retry once. If still failing, reply to practice: "I'm experiencing a temporary issue. Please try again in a few minutes, or email us at studio@sgadental.com and a team member will respond." |
| Weave API unavailable | Fall back to manual: deliver email asset as attachment with instructions |
| Gelato API error | Notify practice: "There was an issue placing your print order. Reply 'retry' to try again, or download the PDF to order locally." |
| Brief incomplete after 3 turns | Escalate to human designer queue with full conversation log |

---

## Environment Variables

```bash
# Claude API
ANTHROPIC_API_KEY=...
CLAUDE_MODEL=claude-sonnet-4-6

# Microsoft Graph / Teams
M365_TENANT_ID=...
M365_CLIENT_ID=...
M365_CLIENT_SECRET=...
STUDIO_EMAIL_ADDRESS=studio@sgadental.com
BOT_APP_ID=...
BOT_APP_PASSWORD=...

# Canva
CANVA_API_KEY=...
CANVA_BASE_URL=https://api.canva.com/rest/v1

# Gelato
GELATO_API_KEY=...
GELATO_API_URL=https://order.gelatoapis.com

# Weave (per practice — stored in DB, not env)
# WEAVE_API credentials stored encrypted in practice profile

# Database
DATABASE_URL=postgresql://...

# Redis (BullMQ)
REDIS_URL=redis://...

# Azure Storage (previews)
AZURE_STORAGE_CONNECTION_STRING=...
AZURE_STORAGE_CONTAINER=studio-previews

# PostHog
POSTHOG_API_KEY=...
POSTHOG_HOST=https://posthog.sga-internal.com

# Environment
NODE_ENV=production
```

---

## Phase 1 Constraints

- Email and Teams only (no Slack in Phase 1)
- Weave integration optional — graceful fallback to manual delivery if unavailable
- TV slide: delivered as file attachment, not auto-deployed
- 8 templates maximum — template expansion is a studio admin Canva task
- 15 Gen4 pilot practices only — practice DB seeded manually before launch
- CD review: Marcus's Teams account configured as primary, one backup required before launch
