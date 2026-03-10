# Inspiration Research

**Sprint:** Studio Automation at Scale
**Agent:** Solution Designer
**Date:** 2026-03-10
**Target:** The creative request intake and routing moment

---

## Research Approach

The target is a smart intake layer that understands intent, challenges the medium, checks assets, and routes to the right workflow. Research focused on: (1) how high-volume franchise/multi-location brands handle decentralized creative requests, (2) how AI-driven intake and routing systems work at scale, (3) how design automation has been implemented in Canva-native and adjacent environments.

---

## Inspiration Entries

### 1. Canva for Enterprise — Brand-Locked Template System
**Source:** Canva Enterprise, franchise/multi-brand clients (Marriott Hotels, News Corp, Redbull)
**What they do:** Enterprise Canva accounts allow a Brand Admin to lock brand colors, fonts, and logos at the workspace level. Individual users (franchisees, regional managers) can only edit approved fields in pre-built templates — name, date, phone number — while the design itself stays locked. The "Brand Kit" is the single source of truth. Outputs are print-ready or export-ready.
**The insight:** This is the design guardrail system SGA already partly uses — but informally, with no enterprise governance. The infrastructure exists. The process doesn't.
**Relevance to sprint:** High. Any solution should use Canva Enterprise's brand kit architecture as the backbone. The question is what sits in front of it.

---

### 2. Opus Clip / HeyGen — Conversational AI for Creative Production
**Source:** Opus Clip, HeyGen, ElevenLabs content generation workflows
**What they do:** Operators submit raw inputs (a prompt, a video file, a brief) through a chat or upload interface. The AI interprets intent, generates output, and presents options for review. No design training required. The human's job shifts from creation to selection.
**The insight:** The AI-as-creative-collaborator model is mature in video and content. The same pattern applies to collateral — intake the intent, generate options, select and approve.
**Relevance to sprint:** Medium-high. The conversational intake + multi-option generation pattern is transferable to print/digital collateral.

---

### 3. McDonald's Global Restaurant Experience Portal
**Source:** McDonald's franchisee operations playbook and digital asset management
**What they do:** Every McDonald's franchisee globally accesses marketing materials through a centralized portal. They cannot deviate from the brand. Materials are market/region-specific, with localization fields (address, hours, promotion). Print orders route through nominated vendors automatically — the franchisee clicks "order," the file goes to the printer, the materials arrive. The corporate team audits, not manages, each order.
**The insight:** The nominated vendor model (one approved source, no routing around it) works at McDonald's scale. Practices feel like they have freedom; they don't. The guardrails are invisible.
**Relevance to sprint:** High. This is the target state for SGA — branded portal, nominated vendor, practice manager clicks, materials arrive.

---

### 4. Chili Piper / Typeform — Smart Intake Routing
**Source:** Chili Piper (sales routing), Typeform (conditional logic forms)
**What they do:** Intake forms that route to different workflows based on answers. Chili Piper qualifies and routes inbound sales leads to the right rep in real-time. Typeform's conditional branching means different answers surface different questions — the form adapts to what you say.
**The insight:** The "intake" doesn't have to be a static form. A conditional, conversational intake — "What's this for? Who's the audience? When do you need it?" — routes to different outcomes (automated Canva template, human designer queue, digital channel suggestion) without the practice manager knowing the difference.
**Relevance to sprint:** High. The intake routing logic is a direct application.

---

### 5. Shutterfly Business / MOO Print API
**Source:** Shutterfly for Business, MOO Print API, Gelato print-on-demand API
**What they do:** APIs that accept print-ready files and trigger production + fulfillment without human involvement. Gelato has 130+ print partners globally; MOO handles premium business card and collateral production. Both offer white-label ordering portals. When a file is approved, the order triggers automatically — correct specs, correct size, ships directly to the location.
**The insight:** Print vendor automation is a solved problem at the API level. The SGA "nominated vendor" can be connected directly to the approval workflow — approved = ordered, no human touch.
**Relevance to sprint:** Medium-high. This is the vendor consolidation solution, connected to the intake output.

---

### 6. Brandfolder / Bynder — Digital Asset Management at Scale
**Source:** Brandfolder (used by Red Bull, Slack, Shopify), Bynder (franchise and multi-brand DAM)
**What they do:** Centralized brand asset libraries with per-brand or per-location brand kits, version control, usage rights tagging, and API access for design tools. Brandfolder's AI-powered search understands "find the logo for Practice X" and surfaces the right asset. Brand-approved assets only; outdated versions are automatically archived.
**The insight:** A proper DAM prevents the "I don't have the logo in the right format" problem. The DAM feeds the Canva template system. Without it, AI generation produces generic output.
**Relevance to sprint:** Medium. A DAM layer (whether Brandfolder, Bynder, or Canva's built-in brand kit) is a prerequisite for the intake system — not the solution itself.

---

### 7. HubSpot Marketing Hub — Proactive Content Calendar + Automation
**Source:** HubSpot, ActiveCampaign, Klaviyo automated campaign workflows
**What they do:** Marketing automation platforms that push content on a schedule rather than waiting for requests. A pre-built campaign calendar — linked to seasonal promotions, recurring patient touchpoints, and local events — triggers asset creation and distribution automatically. The practice doesn't request; they receive.
**The insight:** Proactive push eliminates reactive intake entirely for known, recurring content needs. The review request card, the quarterly promotion, the holiday hours update — these can all be scheduled, generated, and deployed automatically.
**Relevance to sprint:** High for one of the solution approaches. Proactive content calendaring as an alternative to reactive intake is the boldest reframe of the problem.

---

### 8. Figma's Dev Mode + Design Token System
**Source:** Figma Dev Mode, design token frameworks (Theo, Style Dictionary)
**What they do:** Design systems where variables (color, font, spacing) are stored centrally and propagate to all templates automatically. When a brand color changes, every template updates instantly. Figma's component library means a change in one master component updates every instance.
**The insight:** The "brand kit as code" principle applies to Canva templates. A change to a practice's brand color should auto-propagate across all their templates — not require manual updates.
**Relevance to sprint:** Medium. Architectural principle for how the DAM and template system should be structured.

---

### 9. Franchise Brand Portals (Subway, Sport Clips)
**Source:** Franchise brand management platforms (Marq formerly Lucidpress, Papirfly, Lytho)
**What they do:** Marq/Lucidpress builds brand-controlled template portals specifically for franchises. Franchisees log in, select a template, customize approved fields, and download or order print. Papirfly adds campaign management and digital asset storage. Sport Clips uses a version of this so stylists can create local promotional signage without touching the master design.
**The insight:** There is a mature category of "franchise brand portals" purpose-built for exactly SGA's problem. These sit above Canva and add structured intake, approval workflows, and vendor connections.
**Relevance to sprint:** High. Marq/Lucidpress, Papirfly, or Lytho might be the platform layer — with or without Canva underneath.

---

### 10. Intercom / Drift — Conversational AI Bots for Service Routing
**Source:** Intercom Fin AI, Drift Conversational Marketing
**What they do:** AI chatbots that handle intake, qualify the request, answer FAQs, and route to the right human or workflow when needed. They understand natural language, not form fields. "We need marketing materials for our liner special" is understood without requiring the practice manager to know what category, format, or template they need.
**The insight:** The chatbot frontend is mature, available, and widely adopted. The question isn't whether to use a chatbot — it's what the chatbot knows and routes to.
**Relevance to sprint:** High. Chatbot as the intake frontend is the most natural UX for a practice manager who currently texts the designer.

---

## Synthesis: What the Best Solutions Have in Common

1. **Guardrails are invisible.** The best franchise brand systems make the rules invisible — you can only do what's brand-approved, but it feels like freedom.
2. **The intake adapts to the answer.** Conditional routing based on what the practice says, not a static form.
3. **Assets are the prerequisite.** Every system fails without a clean, current, per-practice asset library.
4. **Approval is mobile and fast.** One-tap approve on a phone, not "open Canva and review."
5. **Vendor is connected, not separate.** Print ordering happens as part of approval, not after.
6. **Proactive beats reactive.** The best systems push content on a schedule, not wait for requests.
