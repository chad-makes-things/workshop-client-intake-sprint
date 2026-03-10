# SGA Dental Partners -- Schema Markup Specifications

**Prepared:** March 2026
**SEO strategist:** Content Strategy Agent
**Format:** JSON-LD (recommended by Google for all structured data)
**Validation:** All examples below conform to Schema.org vocabulary and Google's structured data guidelines

---

## 1. Organization Schema (sgadental.com -- Site-Wide)

Deploy on every page in the `<head>` section. This establishes the core entity graph for SGA Dental Partners.

```json
{
  "@context": "https://schema.org",
  "@type": "Organization",
  "@id": "https://sgadental.com/#organization",
  "name": "SGA Dental Partners",
  "alternateName": ["SGA Dental", "Gen4 Dental Partners", "MODIS Dental Partners"],
  "url": "https://sgadental.com",
  "logo": {
    "@type": "ImageObject",
    "url": "https://sgadental.com/images/sga-dental-partners-logo.png",
    "width": 600,
    "height": 200
  },
  "description": "SGA Dental Partners is a dental partnership platform backed by the Thurston Group, consolidating Gen4 Dental Partners and MODIS Dental Partners. Three verticals: community dentistry, collaborative general and specialty practice, and advanced implantology.",
  "foundingDate": "2020",
  "parentOrganization": {
    "@type": "Organization",
    "name": "Thurston Group",
    "url": "https://thurstongroup.com",
    "description": "Private equity firm with 38-year track record and $4B+ returned to partners across 8+ dental and healthcare platforms."
  },
  "address": {
    "@type": "PostalAddress",
    "streetAddress": "43 Ford Way",
    "addressLocality": "Richmond Hill",
    "addressRegion": "GA",
    "postalCode": "31324",
    "addressCountry": "US"
  },
  "contactPoint": [
    {
      "@type": "ContactPoint",
      "contactType": "partnership inquiries",
      "url": "https://sgadental.com/partners"
    },
    {
      "@type": "ContactPoint",
      "contactType": "career inquiries",
      "url": "https://sgadental.com/careers"
    }
  ],
  "sameAs": [
    "https://www.linkedin.com/company/sga-dental-partners",
    "https://www.linkedin.com/company/gen4-dental-partners",
    "https://www.linkedin.com/company/modis-dental-partners",
    "https://www.facebook.com/sgadental",
    "https://www.youtube.com/@InnovativeDental"
  ],
  "numberOfEmployees": {
    "@type": "QuantitativeValue",
    "minValue": 500,
    "maxValue": 2000
  },
  "areaServed": {
    "@type": "Country",
    "name": "United States"
  },
  "knowsAbout": [
    "dental service organizations",
    "dental practice partnerships",
    "dental implantology",
    "dental practice management",
    "continuing dental education"
  ]
}
```

**Implementation notes:**
- Update the logo URL, sameAs profiles, and employee count with actual values before deployment.
- The `@id` anchor allows other schema on the site to reference back to this entity.
- Include Gen4 and MODIS as `alternateName` to maintain entity association during brand consolidation.

---

## 2. LocalBusiness Schema (Practice Location Pages)

Deploy on each practice location page at `/practices/[name]`. Customize all fields per location.

```json
{
  "@context": "https://schema.org",
  "@type": "Dentist",
  "@id": "https://sgadental.com/practices/dental-partners-sw-georgia-valdosta-ga/#localbusiness",
  "name": "Dental Partners of SW Georgia",
  "image": "https://sgadental.com/images/practices/dental-partners-sw-georgia-office.jpg",
  "url": "https://sgadental.com/practices/dental-partners-sw-georgia-valdosta-ga",
  "telephone": "+1-229-555-0100",
  "priceRange": "$$",
  "address": {
    "@type": "PostalAddress",
    "streetAddress": "123 Main Street",
    "addressLocality": "Valdosta",
    "addressRegion": "GA",
    "postalCode": "31601",
    "addressCountry": "US"
  },
  "geo": {
    "@type": "GeoCoordinates",
    "latitude": 30.8327,
    "longitude": -83.2785
  },
  "openingHoursSpecification": [
    {
      "@type": "OpeningHoursSpecification",
      "dayOfWeek": ["Monday", "Tuesday", "Wednesday", "Thursday"],
      "opens": "08:00",
      "closes": "17:00"
    },
    {
      "@type": "OpeningHoursSpecification",
      "dayOfWeek": "Friday",
      "opens": "08:00",
      "closes": "14:00"
    }
  ],
  "department": [
    {
      "@type": "Dentist",
      "name": "General Dentistry",
      "medicalSpecialty": "GeneralDentistry"
    },
    {
      "@type": "Dentist",
      "name": "Cosmetic Dentistry",
      "medicalSpecialty": "CosmeticDentistry"
    }
  ],
  "parentOrganization": {
    "@type": "Organization",
    "@id": "https://sgadental.com/#organization"
  },
  "hasMap": "https://maps.google.com/?cid=GOOGLE_PLACE_ID_HERE",
  "aggregateRating": {
    "@type": "AggregateRating",
    "ratingValue": "4.8",
    "reviewCount": "245",
    "bestRating": "5"
  },
  "review": [
    {
      "@type": "Review",
      "author": {
        "@type": "Person",
        "name": "Sarah M."
      },
      "datePublished": "2026-01-15",
      "reviewRating": {
        "@type": "Rating",
        "ratingValue": "5",
        "bestRating": "5"
      },
      "reviewBody": "The team at Dental Partners has taken care of my family for over 10 years. They always take time to explain treatment options and never rush us through appointments."
    }
  ],
  "medicalSpecialty": [
    "GeneralDentistry",
    "CosmeticDentistry",
    "PediatricDentistry"
  ],
  "availableService": [
    {
      "@type": "MedicalProcedure",
      "name": "Dental Cleaning and Exam"
    },
    {
      "@type": "MedicalProcedure",
      "name": "Dental Implants"
    },
    {
      "@type": "MedicalProcedure",
      "name": "Cosmetic Dentistry"
    },
    {
      "@type": "MedicalProcedure",
      "name": "Pediatric Dentistry"
    },
    {
      "@type": "MedicalProcedure",
      "name": "Emergency Dental Care"
    }
  ]
}
```

**Implementation notes:**
- Replace all placeholder values (address, phone, coordinates, ratings, reviews) with actual practice data.
- The `aggregateRating` must reflect actual verified review data. Pull from Google Business Profile.
- The `review` array should include 2-3 representative reviews. Only use reviews with explicit patient consent.
- `medicalSpecialty` and `availableService` should be customized per practice (e.g., MODIS locations include implantology; pediatric practices include Pediatric Dentistry).
- Include `hasMap` with the Google Maps CID for each location.

---

## 3. FAQPage Schema (Landing Pages with FAQ Sections)

Deploy on any page containing an FAQ section: /partners, /careers, /implants, /why-sga, /specialists, and all blog articles.

### Example: Practice Owners FAQ (/partners)

```json
{
  "@context": "https://schema.org",
  "@type": "FAQPage",
  "mainEntity": [
    {
      "@type": "Question",
      "name": "Do I keep my practice name after partnering with SGA?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "Yes. Every SGA partner practice retains its local brand name, team, and community identity. Dental Partners of SW Georgia is still Dental Partners of SW Georgia. Periodontal Associates of Memphis is still Periodontal Associates of Memphis. Your patients will not notice a difference except that operations run more smoothly."
      }
    },
    {
      "@type": "Question",
      "name": "How does the equity partnership model work?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "SGA uses a joint-venture equity partnership model. Partner dentists receive a liquidity event (cash at close) while retaining an ownership stake in the combined SGA platform. You hold equity shares for as long as you remain an active partner, creating long-term wealth through the platform's compounding value. This is different from an outright sale where your financial upside ends at closing."
      }
    },
    {
      "@type": "Question",
      "name": "Will I lose clinical autonomy if I partner with SGA?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "No. SGA partners retain full clinical independence. You choose your materials, your lab, your treatment protocols, and your schedule. As Gen4 CCO Dr. Mitch Ellingson explains: 'I want to protect my profession from becoming like medicine. In medicine, as it consolidated it became less about the doctors and patients. We want it to be more about the doctors and the patients.'"
      }
    },
    {
      "@type": "Question",
      "name": "What happens to my staff after the partnership?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "Your staff stays. SGA maintains an 85% staff retention rate across its network. Staff receive upgraded benefits including 401(k) through Voya, healthcare through Cigna, life insurance through Mutual of Omaha, PTO, and CE stipends. The SGA back-office team handles payroll, HR, compliance, and billing so your team can focus on patient care."
      }
    },
    {
      "@type": "Question",
      "name": "Who is the Thurston Group?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "The Thurston Group is SGA's financial sponsor, founded in 1988 by Patrick J. Haynes III. Over 38 years, Thurston has returned over $4 billion to partners across 8+ dental and healthcare platforms including Smile Doctors, US Endodontics Partners, US Oral Surgery Management, and ARC Health. This is the most extensive dental-specific PE track record in the industry."
      }
    }
  ]
}
```

### Example: Implant Patient FAQ (/implants)

```json
{
  "@context": "https://schema.org",
  "@type": "FAQPage",
  "mainEntity": [
    {
      "@type": "Question",
      "name": "How much do full arch dental implants cost?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "Full arch dental implants typically range from $20,000 to $50,000 per arch, depending on the procedure type, materials (zirconia vs. acrylic), and complexity of your case. Financing options are available starting at approximately $250 per month. A free consultation with one of our implant specialists will provide a personalized treatment plan and transparent cost estimate."
      }
    },
    {
      "@type": "Question",
      "name": "Can I get dental implants in one day?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "Yes. Same-day dental implant procedures allow your implants to be placed and temporary teeth fitted in a single visit. Not every patient is a candidate for same-day implants. Factors including bone density, overall health, and the complexity of your case determine whether a same-day or staged approach is best. Your consultation includes a CT scan to determine your candidacy."
      }
    },
    {
      "@type": "Question",
      "name": "Are dental implants painful?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "Most patients report that the dental implant procedure is less painful than they expected. You are under sedation during the procedure and feel no pain. Post-procedure discomfort is typically managed with over-the-counter pain medication and resolves within a few days. Many patients describe the recovery as comparable to a tooth extraction."
      }
    },
    {
      "@type": "Question",
      "name": "What is the success rate of dental implants?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "Dental implants have a success rate of 95-98% over 10 years when placed by experienced implant specialists. Success depends on factors including bone quality, oral hygiene, overall health, and the skill of your implant team. SGA's implant specialists include Dr. Michael Pikos, a world-renowned pioneer in implantology, and Dr. Grant Olson, who has performed thousands of implant procedures."
      }
    }
  ]
}
```

---

## 4. Article Schema (Blog Posts)

Deploy on every blog article page. Customize per article.

```json
{
  "@context": "https://schema.org",
  "@type": "Article",
  "headline": "How to Evaluate a DSO Partnership: A Practice Owner's Checklist",
  "description": "A step-by-step guide for dental practice owners evaluating DSO partnership offers. Covers deal structure, clinical autonomy protections, equity models, and due diligence questions to ask.",
  "image": "https://sgadental.com/images/blog/evaluate-dso-partnership-hero.jpg",
  "author": {
    "@type": "Person",
    "name": "Lamonte Jensen",
    "jobTitle": "CEO, Gen4 Dental Partners",
    "url": "https://sgadental.com/about/lamonte-jensen",
    "sameAs": "https://www.linkedin.com/in/lamontejensen"
  },
  "publisher": {
    "@type": "Organization",
    "@id": "https://sgadental.com/#organization"
  },
  "datePublished": "2026-03-15",
  "dateModified": "2026-03-15",
  "mainEntityOfPage": {
    "@type": "WebPage",
    "@id": "https://sgadental.com/blog/evaluate-dso-partnership"
  },
  "wordCount": 2200,
  "articleSection": "Practice Transition",
  "keywords": ["evaluate DSO partnership", "DSO due diligence", "dental practice partnership checklist"],
  "about": [
    {
      "@type": "Thing",
      "name": "Dental Service Organizations"
    },
    {
      "@type": "Thing",
      "name": "Dental Practice Transitions"
    }
  ]
}
```

**Template variables per article:**
- `headline`: Article title (H1)
- `description`: Meta description
- `image`: Hero image URL
- `author`: Named author with credentials (critical for E-E-A-T)
- `datePublished` / `dateModified`: Keep updated when content is refreshed
- `wordCount`: Actual word count
- `articleSection`: One of: "Practice Transition", "Dental Careers", "Dental Implants", "Industry Insights"
- `keywords`: Target keywords for the article

**Author pool for article schema:**
| Author | Credentials | Content Type |
|--------|------------|-------------|
| Lamonte Jensen | CEO, Gen4 Dental Partners; ex-McKinsey | Business, partnership, industry |
| Dr. Mitch Ellingson | CCO, Gen4 Dental Partners; practicing dentist | Clinical autonomy, practice culture |
| Dr. Grant Olson | Founder, Innovative Dental; CEREC Mentor; 138K YouTube subscribers | Implants, clinical technology |
| Dr. Michael Pikos | Founding Partner, MODIS/Pikos Institute; world-renowned implantologist | Implantology, specialist education |
| Pete Wong | CEO, MODIS Dental Partners | Specialist partnerships, MODIS vertical |

---

## 5. HowTo Schema (Process/Guide Pages)

Deploy on articles that describe step-by-step processes. High value for featured snippet capture.

### Example: "How to Sell Your Dental Practice" (Cluster 1 Pillar)

```json
{
  "@context": "https://schema.org",
  "@type": "HowTo",
  "name": "How to Sell Your Dental Practice to a DSO",
  "description": "A step-by-step guide to selling or partnering your dental practice with a dental service organization, from initial valuation through closing.",
  "totalTime": "P6M",
  "estimatedCost": {
    "@type": "MonetaryAmount",
    "currency": "USD",
    "value": "0",
    "description": "No upfront cost to explore partnership. Legal and advisory fees at closing vary."
  },
  "step": [
    {
      "@type": "HowToStep",
      "position": 1,
      "name": "Assess Your Readiness",
      "text": "Evaluate your practice's financial health, operational dependence on you as the owner, and your personal timeline. Consider whether you want a full exit, a partial liquidity event with ongoing involvement, or a management-only transition.",
      "url": "https://sgadental.com/blog/sell-dental-practice-guide#assess-readiness"
    },
    {
      "@type": "HowToStep",
      "position": 2,
      "name": "Understand Your Options",
      "text": "Compare the three primary paths: private sale to another dentist, broker-assisted sale, or DSO partnership. Each has different timelines, valuations, and post-sale implications for your practice identity and team.",
      "url": "https://sgadental.com/blog/sell-dental-practice-guide#understand-options"
    },
    {
      "@type": "HowToStep",
      "position": 3,
      "name": "Get a Practice Valuation",
      "text": "Obtain a professional valuation based on your practice's EBITDA, revenue trends, patient base, equipment condition, and market position. Valuations for DSO partnerships typically use EBITDA multiples ranging from 6-12x depending on practice size and performance.",
      "url": "https://sgadental.com/blog/sell-dental-practice-guide#get-valuation"
    },
    {
      "@type": "HowToStep",
      "position": 4,
      "name": "Evaluate Potential Partners",
      "text": "Research DSOs that match your values, geographic market, and practice model. Key evaluation criteria include clinical autonomy protections, equity model structure, PE sponsor track record, education offerings, and cultural fit.",
      "url": "https://sgadental.com/blog/sell-dental-practice-guide#evaluate-partners"
    },
    {
      "@type": "HowToStep",
      "position": 5,
      "name": "Enter Discovery Conversations",
      "text": "Meet with 2-3 DSOs in confidential discovery conversations. Ask for partner references, visit existing partner practices, and review the equity model in detail with your financial advisor.",
      "url": "https://sgadental.com/blog/sell-dental-practice-guide#discovery"
    },
    {
      "@type": "HowToStep",
      "position": 6,
      "name": "Complete Due Diligence and Close",
      "text": "Once you select a partner, enter due diligence. This includes financial audit, legal review, staff transition planning, and finalizing the equity partnership terms. The process from LOI to close typically takes 60-120 days.",
      "url": "https://sgadental.com/blog/sell-dental-practice-guide#close"
    }
  ]
}
```

### Example: "What to Expect at Your Dental Implant Consultation" (Cluster 3)

```json
{
  "@context": "https://schema.org",
  "@type": "HowTo",
  "name": "What to Expect at Your Dental Implant Consultation",
  "description": "Step-by-step walkthrough of a dental implant consultation, from imaging to treatment planning to cost discussion.",
  "totalTime": "PT2H",
  "step": [
    {
      "@type": "HowToStep",
      "position": 1,
      "name": "Comprehensive Imaging",
      "text": "Your consultation begins with a 3D CT scan of your jaw. This takes about 15 minutes and is completely painless. The scan shows your bone density, nerve locations, and sinus anatomy to determine your implant candidacy."
    },
    {
      "@type": "HowToStep",
      "position": 2,
      "name": "Clinical Examination",
      "text": "Your implant specialist examines your mouth, reviews your dental and medical history, and discusses your goals. This is your opportunity to ask questions about the procedure, recovery, and outcomes."
    },
    {
      "@type": "HowToStep",
      "position": 3,
      "name": "Treatment Plan Presentation",
      "text": "Based on your imaging and exam, your specialist presents a customized treatment plan including the recommended procedure type, number of implants, material options, and expected timeline from start to final teeth."
    },
    {
      "@type": "HowToStep",
      "position": 4,
      "name": "Cost and Financing Discussion",
      "text": "You receive a transparent cost breakdown for your specific treatment plan. The team reviews insurance coverage, financing options, and payment plans. There is no obligation at your consultation."
    }
  ]
}
```

---

## 6. VideoObject Schema (Pages with Embedded YouTube Content)

Deploy on any page embedding Dr. Olson's YouTube videos or other SGA video content.

```json
{
  "@context": "https://schema.org",
  "@type": "VideoObject",
  "name": "Full Arch Dental Implant Transformation - Patient Story",
  "description": "Watch a real patient's dental implant transformation from consultation to final reveal. Dr. Grant Olson of Innovative Dental walks through the full arch implant procedure.",
  "thumbnailUrl": "https://sgadental.com/images/video-thumbnails/full-arch-transformation.jpg",
  "uploadDate": "2025-11-15",
  "duration": "PT12M30S",
  "contentUrl": "https://www.youtube.com/watch?v=VIDEOID",
  "embedUrl": "https://www.youtube.com/embed/VIDEOID",
  "interactionStatistic": {
    "@type": "InteractionCounter",
    "interactionType": "https://schema.org/WatchAction",
    "userInteractionCount": 450000
  },
  "author": {
    "@type": "Person",
    "name": "Dr. Grant Olson",
    "jobTitle": "Implant Dentist and Founder, Innovative Dental",
    "url": "https://sgadental.com/about/dr-grant-olson"
  },
  "publisher": {
    "@type": "Organization",
    "@id": "https://sgadental.com/#organization"
  }
}
```

**Implementation notes:**
- Add VideoObject schema to every page with an embedded YouTube video.
- Update `interactionStatistic.userInteractionCount` periodically with actual view counts.
- The `thumbnailUrl` should be a custom image hosted on sgadental.com, not the YouTube default.
- For pages with multiple videos, include an array of VideoObject entries.

---

## 7. JobPosting Schema (Career Listings)

Deploy on each individual job listing page. Critical for Google for Jobs inclusion.

```json
{
  "@context": "https://schema.org",
  "@type": "JobPosting",
  "title": "Associate Dentist - Valdosta, GA",
  "description": "SGA Dental Partners is seeking an associate dentist for our Valdosta, Georgia practice. This position includes clinical mentorship, access to the Pikos Institute for implant training, a clear pathway to partnership with equity participation, and comprehensive benefits.",
  "datePosted": "2026-03-01",
  "validThrough": "2026-06-01",
  "employmentType": "FULL_TIME",
  "hiringOrganization": {
    "@type": "Organization",
    "@id": "https://sgadental.com/#organization"
  },
  "jobLocation": {
    "@type": "Place",
    "address": {
      "@type": "PostalAddress",
      "streetAddress": "123 Main Street",
      "addressLocality": "Valdosta",
      "addressRegion": "GA",
      "postalCode": "31601",
      "addressCountry": "US"
    }
  },
  "baseSalary": {
    "@type": "MonetaryAmount",
    "currency": "USD",
    "value": {
      "@type": "QuantitativeValue",
      "minValue": 150000,
      "maxValue": 250000,
      "unitText": "YEAR"
    }
  },
  "jobBenefits": "401(k) through Voya, Cigna healthcare, Mutual of Omaha life insurance, PTO, CE stipends, Pikos Institute access, clinical mentorship program, equity participation from day one as Class A shareholder",
  "qualifications": "DDS or DMD from an accredited dental school. Active dental license in the state of Georgia or eligibility to obtain. GPR or AEGD residency preferred.",
  "responsibilities": "Provide comprehensive dental care including restorative, preventive, and cosmetic procedures. Participate in clinical mentorship program. Contribute to practice growth and patient experience.",
  "industry": "Healthcare / Dentistry",
  "occupationalCategory": "29-1021.00",
  "directApply": true,
  "applicationContact": {
    "@type": "ContactPoint",
    "url": "https://sgadental.com/careers/apply/associate-dentist-valdosta"
  }
}
```

**Implementation notes:**
- Salary ranges must reflect actual compensation. Google penalizes misleading salary data in job schema.
- `validThrough` must be kept current. Expired job postings hurt search visibility.
- `occupationalCategory` uses the O*NET code for Dentists (29-1021.00).
- Include `directApply: true` if applications can be submitted directly on the SGA website.

---

## 8. BreadcrumbList Schema (All Pages)

Deploy on every page to enhance navigation in search results.

```json
{
  "@context": "https://schema.org",
  "@type": "BreadcrumbList",
  "itemListElement": [
    {
      "@type": "ListItem",
      "position": 1,
      "name": "Home",
      "item": "https://sgadental.com"
    },
    {
      "@type": "ListItem",
      "position": 2,
      "name": "Blog",
      "item": "https://sgadental.com/blog"
    },
    {
      "@type": "ListItem",
      "position": 3,
      "name": "How to Evaluate a DSO Partnership",
      "item": "https://sgadental.com/blog/evaluate-dso-partnership"
    }
  ]
}
```

**Path patterns:**
- Landing pages: Home > [Page Name]
- Blog articles: Home > Blog > [Article Title]
- Location pages: Home > Practices > [Practice Name]
- Career listings: Home > Careers > [Job Title]

---

## 9. WebSite Schema with SearchAction (Homepage)

Deploy on the homepage only. Enables sitelinks search box in Google results.

```json
{
  "@context": "https://schema.org",
  "@type": "WebSite",
  "name": "SGA Dental Partners",
  "url": "https://sgadental.com",
  "publisher": {
    "@type": "Organization",
    "@id": "https://sgadental.com/#organization"
  },
  "potentialAction": {
    "@type": "SearchAction",
    "target": {
      "@type": "EntryPoint",
      "urlTemplate": "https://sgadental.com/search?q={search_term_string}"
    },
    "query-input": "required name=search_term_string"
  }
}
```

---

## 10. Person Schema (Leadership Profiles)

Deploy on the About page and author byline pages.

```json
{
  "@context": "https://schema.org",
  "@type": "Person",
  "name": "Dr. Grant Olson",
  "jobTitle": "Founder, Innovative Dental",
  "description": "Dr. Grant Olson is one of approximately 300 CEREC Mentors worldwide and a leader in guided implant technology. His YouTube channel has over 138,000 subscribers and 23 million views, making him one of the most-watched dental educators online.",
  "url": "https://sgadental.com/about/dr-grant-olson",
  "image": "https://sgadental.com/images/team/dr-grant-olson.jpg",
  "sameAs": [
    "https://www.youtube.com/@InnovativeDental",
    "https://www.instagram.com/drgrantolson",
    "https://www.linkedin.com/in/grantolson"
  ],
  "worksFor": {
    "@type": "Organization",
    "@id": "https://sgadental.com/#organization"
  },
  "alumniOf": {
    "@type": "EducationalOrganization",
    "name": "University of Missouri-Kansas City School of Dentistry"
  },
  "knowsAbout": [
    "dental implants",
    "CEREC technology",
    "guided implant surgery",
    "full arch dental implants",
    "cosmetic dentistry"
  ]
}
```

**Create Person schema for each key figure:**
- Dr. Grant Olson (Innovative Dental / MODIS)
- Dr. Michael Pikos (Pikos Institute / MODIS)
- Lamonte Jensen (CEO, Gen4)
- Dr. Mitch Ellingson (CCO, Gen4)
- Pete Wong (CEO, MODIS)

---

## 11. Schema Deployment Checklist

| Page Type | Schema Types Required | Priority |
|-----------|----------------------|----------|
| Homepage | Organization, WebSite, BreadcrumbList | Critical |
| /partners | FAQPage, BreadcrumbList | Critical |
| /careers | BreadcrumbList | High |
| /careers/[job] | JobPosting, BreadcrumbList | Critical (Google for Jobs) |
| /implants | FAQPage, VideoObject, BreadcrumbList | High |
| /why-sga | FAQPage, BreadcrumbList | High |
| /specialists | FAQPage, BreadcrumbList | High |
| /about | Person (multiple), BreadcrumbList | High |
| /practices/[name] | Dentist (LocalBusiness), BreadcrumbList | Critical (Local SEO) |
| /blog | CollectionPage, BreadcrumbList | Medium |
| /blog/[article] | Article, FAQPage, BreadcrumbList | High |
| /blog/[article-with-video] | Article, FAQPage, VideoObject, BreadcrumbList | High |
| /blog/[how-to-article] | Article, FAQPage, HowTo, BreadcrumbList | High |

---

## 12. Validation and Testing

**Before deployment:**
1. Validate every schema block using Google's Rich Results Test (https://search.google.com/test/rich-results)
2. Test with Schema.org Validator (https://validator.schema.org)
3. After deployment, monitor Search Console > Enhancements for errors and warnings
4. Check that FAQ, HowTo, and JobPosting rich results appear within 1-2 weeks of deployment

**Common pitfalls to avoid:**
- Do not include FAQPage schema on pages where the FAQ content is not visible to users (cloaking)
- Do not inflate AggregateRating numbers beyond verified review data
- Do not include salary ranges in JobPosting that do not reflect actual compensation
- Keep `dateModified` in Article schema current when content is updated
- Do not use Review schema for editorial quotes or testimonials that are not genuine product/service reviews
