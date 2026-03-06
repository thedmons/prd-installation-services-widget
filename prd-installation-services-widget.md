# PRD: Installation Services Lead Capture Widget
### E-Commerce Web Platform — Consolidated Installation Services Experience

---

| | |
|---|---|
| **Product** | E-Commerce Web Platform — Installation Services |
| **Document Type** | Product Requirements Document (PRD) |
| **Status** | Discovery Complete · Deprioritized (platform migration) |
| **Date** | 2020–2021 |
| **Surface** | Desktop web · Mobile web |
| **Author** | Digital Product |
| **Stakeholders** | Installation Services Business Unit · UX · Engineering · Analytics · Microsoft Dynamics CRM Team |
| **Related Docs** | [System Design: CAAS Architecture](./system-design-caas-installation-services.md) · [API Spec: CAAS Lead Capture](./api-spec-caas-installation-services.md) |

---

## Table of Contents

1. [Overview](#1-overview)
2. [Background & Problem Statement](#2-background--problem-statement)
3. [Goals & Success Metrics](#3-goals--success-metrics)
4. [Scope](#4-scope)
5. [User Personas](#5-user-personas)
6. [Service Types & Display Types](#6-service-types--display-types)
7. [Widget Flow & Functional Requirements](#7-widget-flow--functional-requirements)
8. [Content Model & Configuration](#8-content-model--configuration)
9. [CRM Integration](#9-crm-integration)
10. [Non-Functional Requirements](#10-non-functional-requirements)
11. [Dependencies & Risks](#11-dependencies--risks)
12. [Future Considerations](#12-future-considerations)
13. [Discovery Status](#13-discovery-status)

---

## 1. Overview

This document defines the product requirements for the **Content as a Service (CAAS) Installation Services Lead Capture Widget** — a configurable, stepped-form experience designed to replace 40+ individually maintained installation service forms across 7 service categories with a single, guided adaptive flow that routes customers to the correct experience based on their selected service type.

The widget was designed to operate in two modes: as a standalone end-to-end experience, and as a configurable component scoped to a specific service or category and embedded on category or department landing pages — bringing lead capture closer to where customers were already browsing and most likely to convert.

All leads are consolidated to a single **Microsoft Dynamics CRM** endpoint, replacing an inconsistent legacy routing system that split leads across Salesforce, email notifications, and unconfirmed form submissions depending on which form the customer encountered.

This initiative advanced through full discovery — product vision artifacts, UX-owned prototypes, and user testing all completed — and was in active development elaboration before being deprioritized in favor of the platform migration. It represents a validated product vision with a clear implementation path.

---

## 2. Background & Problem Statement

### 2.1 Current State

Installation services were represented across 7 overarching service categories and approximately 40 individual subcategories, each with their own landing page and lead capture form. These forms had been authored independently in the CMS over time, resulting in:

- **No consistency in form design or fields** — form structure, required fields, and UX varied widely across subcategories with no shared standard
- **Inconsistent lead routing** — some forms submitted leads to Salesforce; others generated email notifications; some provided no submission confirmation and left customers at a dead end
- **No analytics instrumentation** — form submission was the only tracked signal; there was no visibility into where customers dropped off, which fields caused friction, or how completion rates varied across service types
- **High maintenance overhead** — 40+ individually authored forms required significant ongoing effort to maintain, update, and keep consistent with business changes
- **Misconfigurations causing lead failures** — inconsistent routing logic meant some leads were never received by the internal team, resulting in follow-up failures and missed business
- **Split CRM destination** — legacy leads were split between Salesforce and email depending on form configuration, with no unified view for the Installation Services business unit

### 2.2 The Opportunity

A single configurable widget — driven by a CAAS configuration service and guided by a stepped flow that determined service type via API and adapted its data collection accordingly — could replace the fragmented form landscape with a consistent, measurable, and maintainable experience.

Rather than maintaining 40+ individually authored forms, the CAAS widget would be configured once per service type, driven by a centralized configuration dashboard, and placed in higher-converting contexts across the platform.

The consolidation also created an opportunity to establish **Microsoft Dynamics** as the single lead CRM endpoint — replacing the legacy Salesforce and email routing with a unified view of inbound leads for the Installation Services business unit.

### 2.3 Strategic Context

This initiative was scoped during the broader consolidation of the retailer's DIY and Pro web platforms onto a single unified platform. Installation services represented a high-value, high-complexity product category with direct revenue implications. The fragmented lead capture experience was both a customer friction point and an operational liability. The initiative was deprioritized — not cancelled — when the platform migration was accelerated following the Black Friday outage, with the intent to revisit post-migration.

---

## 3. Goals & Success Metrics

### 3.1 Goals

- Replace 40+ individually maintained forms with a single CAAS-configured widget
- Guide customers to the correct data collection and display experience based on service type — eliminating dead ends and misconfigured routing
- Consolidate all installation service leads to Microsoft Dynamics as the single CRM endpoint
- Establish step-level analytics visibility on customer flow, drop-off, and form completion across service types
- Reduce CMS authoring and maintenance overhead for the Installation Services category
- Increase lead capture by placing the widget in higher-converting contexts across Hub, Department, and Service pages

### 3.2 Success Metrics

| Metric | Baseline | Target | Measurement |
|---|---|---|---|
| Lead routing accuracy | Unknown — split across Salesforce, email, unconfirmed | 100% of submissions reach Microsoft Dynamics | CRM lead ingestion monitoring |
| Form completion rate | Unknown — no drop-off tracking | Establish baseline; improve vs. legacy | Analytics funnel tracking per step |
| Step-level drop-off rate | Unknown | Establish baseline by service type | Analytics event tracking per step |
| Submission confirmation rate | Inconsistent — many forms had none | 100% of submissions show confirmation | QA validation |
| CMS forms requiring individual maintenance | 40+ | Reduced to CAAS configurations per service type | Content team reporting |

---

## 4. Scope

### 4.1 In Scope

- **CAAS widget:** Stepped, adaptive lead capture experience driven by centralized configuration service
- **Three display types:** Form · Detail Fee · Scheduling Lead (see Section 6)
- **Three service type variants:** Measurement Required · Third-Party Contractor · Simple Install (see Section 6)
- **Zipped-in / Zipped-out states:** Widget supports collapsed (zipped-out) and expanded (zipped-in) display states for embedded placements
- **Areas of Interest step:** Optional pre-form step when a service category contains multiple subcategory options — allows customer to self-select interest before proceeding to form
- **Dual configuration mode:** Standalone (full flow, start to finish) and embedded (scoped to a specific service or department, placed on existing pages)
- **Three page placement contexts:** Hub Page · Department Page · Service Page — each with appropriate widget entry state
- **Microsoft Dynamics integration:** All submissions routed to Dynamics as the single CRM endpoint regardless of service type
- **Scheduler integration:** Appointment scheduling for Measurement and Simple Install service types
- **Submission confirmation:** Consistent confirmation state for all service types; includes product carousel CTA ("Start shopping for your project")
- **Local service unavailability state:** Terminal state displayed when no local installer is available based on customer zip code
- **Three-step process explanation:** Inline process indicators (Submit → Quote → Installation Begins) displayed alongside form to set customer expectations
- **Analytics instrumentation:** Step-level event tracking for funnel visibility and drop-off analysis
- **CAAS Config Dashboard:** Internal admin tool for managing service configurations — search, add, edit, export
- **CMS authoring:** Widget placement and scope configurable by content authors without engineering support

### 4.2 Out of Scope — V1

- Real-time pricing or quote generation
- Online payment or deposit collection
- Contractor assignment or job management (handled by Installation Services business unit post-lead)
- Customer account integration or authenticated experience
- Native app surface
- Post-submission lead status tracking
- Full replacement of all 40+ landing pages in V1 — phased consolidation approach

---

## 5. User Personas

### 5.1 The Installation Customer

A homeowner or renter who has decided they want a product installed but is uncertain about the process, cost, or timeline. They may be arriving from a product page, category page, or direct search. They have high purchase intent but low tolerance for friction — a confusing or dead-end form experience will cause them to disengage or call a store instead.

- **Needs:** Clear guidance on what the service involves; confidence that their request was received; a simple way to get started without needing to know which specific service they need
- **Success:** Completes the widget flow, receives confirmation, is contacted by the Installation Services team within the expected timeframe

### 5.2 The Pro Customer

A contractor or small business owner requesting installation services at scale — potentially for multiple units or job sites. More likely to have specific scope requirements and to value a professional, process-driven experience.

- **Needs:** Ability to describe project scope; clear expectation setting on follow-up process
- **Success:** Submits a complete scope description; lead is received and routed correctly in Microsoft Dynamics

### 5.3 The Installation Services Coordinator

An internal team member responsible for managing inbound installation leads, coordinating with contractors, stores, and in-house installers, and ensuring customer follow-up. Currently operating with inconsistent lead data split across Salesforce and email.

- **Needs:** Consistent, complete lead data in a single Microsoft Dynamics view; no missed leads from misconfigured forms; accurate service type classification to route to the right fulfillment path
- **Success:** All inbound leads appear in Dynamics with complete fields and correct service type; zero dead-end or unrouted submissions

### 5.4 The CAAS Administrator

A member of the Installation Services business unit responsible for configuring and maintaining service entries in the CAAS Config Dashboard. Self-service configuration ownership sits within the business unit — not engineering or CMS authoring teams. They need to add, edit, and manage service configurations without requiring a code deploy or engineering ticket.

- **Needs:** Searchable dashboard of service configurations; ability to edit service descriptions, CTA copy, and scheduling details independently of engineering
- **Success:** Can update a service configuration and have the change reflected in the live widget without involving engineering

---

## 6. Service Types & Display Types

### 6.1 Service Types

Service type is determined after subcategory selection via an API call to CAAS that classifies the selected service. The widget adapts its data collection flow based on the returned service type.

| Service Type | Description | Required Fields | Scheduler |
|---|---|---|---|
| Measurement Required | In-person measurement visit required before work begins (e.g., flooring, countertops, custom window treatments) | Name · Phone · Email · Address · Appointment | ✅ |
| Third-Party Contractor | Fulfilled entirely by third-party contractors; scope details needed for routing and quoting (e.g., HVAC, electrical, plumbing) | Name · Phone · Email · Address · Project scope | ❌ |
| Simple Install | Standard appliance/fixture installs fulfilled by in-house or store-coordinated installers (e.g., dishwasher, washing machine) | Name · Phone · Email · Address · Appointment | ✅ |

**Note:** Address is required for all service types and is used for geographic lead routing to the appropriate store, contractor, or installer.

### 6.2 Display Types

Display type is set at the service configuration level in CAAS and determines the widget's presentation mode on the page.

| Display Type | Description | Use Case |
|---|---|---|
| Form | Standard lead capture form with all required fields | Default for most service types |
| Detail Fee | Presents service pricing/fee details alongside a contact method selector before routing to form or scheduling | Services where fee transparency aids conversion |
| Scheduling Lead | Scheduling-first flow; appointment booking is the primary CTA rather than form submission | Services where immediate scheduling is preferred |

### 6.3 Areas of Interest

When a service category contains multiple subcategory options (e.g., Flooring → Vinyl Plank, Hardwood, Carpet, Tile), the widget presents an **Areas of Interest** step before the main form. The customer selects their primary interest, which is captured as a lead attribute and used to pre-populate relevant product recommendations in the confirmation state.

---

## 7. Widget Flow & Functional Requirements

### 7.1 Stepped Flow

```
[Page Load]
  └─ Request CAAS config on load
        └─ Zipped-out state (collapsed widget visible on page)
              └─ User expands widget (Zipped-in)
                    └─ Step 1: Category Selection (Hub Page standalone only)
                          └─ Step 2: Department / Subcategory Selection
                                └─ [API call → Service type + Display type determination]
                                      └─ Step 3: Service Description
                                            └─ [Areas of Interest? → Yes: AOI selection step]
                                                  └─ Step 4: Information Collection (adapts by service type)
                                                        └─ Step 5: Scheduler (Measurement & Simple Install only)
                                                              └─ Step 6: Confirmation
                                                                    └─ [No local service? → Unavailability state]
```

### 7.2 Page Placement Contexts

| Page Context | Widget Entry State | Notes |
|---|---|---|
| Hub Page | Full flow — Category selection as first step | Standalone placement; all 7 categories available |
| Department Page | Pre-scoped to category — begins at subcategory selection | Embedded; category set by page context |
| Service Page | Pre-scoped to specific service — begins at service description | Embedded; subcategory set by page context |

### 7.3 Functional Requirements

| Step | Feature | Description | Priority |
|---|---|---|---|
| All | Zipped-out state | Collapsed widget visible on page; expands on user interaction | P0 |
| All | Zipped-in state | Expanded widget showing current step | P0 |
| All | Progress stepper | Visual step indicator showing current position in flow; updates on step advance and back navigation | P0 |
| All | Back navigation | User can return to previous step at any point | P0 |
| 1 | Category selection | Visual card or list selection across 7 installation categories | P0 |
| 2 | Subcategory selection | Filtered list of subcategories based on category selection | P0 |
| 3 | Service description | Full description of selected service — what it includes, process steps, typical timeline | P0 |
| 3 | Get Started CTA | Primary action triggers CAAS API call for service type and display type determination | P0 |
| 3 | Loading state | Visible loading indicator during API call | P0 |
| AOI | Areas of Interest | Visual subcategory selector (e.g., flooring type tiles with images); selection captured as lead attribute | P1 |
| 4 | Standard fields | Name, phone, email — required for all service types | P0 |
| 4 | Address fields | Street, city, state, zip — required for all service types; used for geographic lead routing | P0 |
| 4 | Scope description | Free-text field for project scope — required for Third-Party Contractor | P0 |
| 4 | Field validation | Inline validation on all required fields before step advance | P0 |
| 4 | Process explanation | Three-step process (Submit → Quote → Installation Begins) displayed alongside form | P1 |
| 4 | TCPA consent | Required opt-in disclosure copy per legal requirements; automated contact disclosure | P0 |
| 4 | Security messaging | SSL/encrypted transmission indicator | P1 |
| 5 | Scheduler | Date/time selection for Measurement and Simple Install appointments | P0 |
| 5 | Phone consultation alternative | "Free Phone Consultation" fallback CTA for users not ready to schedule | P1 |
| 6 | Confirmation state | "We'll be in touch soon" confirmation for all service types; includes next steps and expected contact timeframe | P0 |
| 6 | Product carousel | Post-submission product recommendations based on service/AOI selection ("Start shopping for your project") | P1 |
| 6 | Microsoft Dynamics submission | All leads submitted to Dynamics on confirmation; service type, display type, AOI, and source page captured as lead attributes | P0 |
| Terminal | Local service unavailability | Displayed when no local installer available based on zip code; provides alternative contact (retailer customer service phone number) | P0 |

---

## 8. Content Model & Configuration

Widget behavior is driven by the CAAS configuration service rather than static CMS authoring. Content authors manage service configurations through the **CAAS Config Dashboard** — a separate internal admin tool with search, add, edit, and export capabilities.

### 8.1 Service Configuration Fields

| Field | Description | Required |
|---|---|---|
| Service Name | Display name for the service | Yes |
| Landing URL | URL used for banner image reference | Yes |
| Service Type | Measurement Required · Third-Party Contractor · Simple Install | Yes |
| Display Type | Form · Detail Fee · Scheduling Lead | Yes |
| Consultation Description | 350-character description of the consultation step | Optional |
| Scheduling Description | 350-character description of the scheduling step | Optional |
| Quoting Details | 350-character description of the quote process | Required |
| Installation Description | 350-character description of the installation process | Required |
| Dynamics Campaign ID | Optional campaign attribution passed with lead submission | Optional |

### 8.2 CMS Widget Placement Configuration

Content authors configure widget instances in the CMS for page-level placement:

| Field | Description | Required |
|---|---|---|
| Widget mode | Standalone · Department scoped · Service scoped | Yes |
| Pre-selected category | For department-scoped and service-scoped modes | Conditional |
| Pre-selected subcategory | For service-scoped mode | Conditional |

---

## 9. CRM Integration

### 9.1 Migration from Legacy Routing

The CAAS widget consolidates all installation service leads to **Microsoft Dynamics**, replacing the inconsistent legacy routing:

| Legacy State | CAAS State |
|---|---|
| Some forms → Salesforce | All leads → Microsoft Dynamics |
| Some forms → Email notification | Eliminated |
| Some forms → No confirmation / dead end | All leads confirmed; Dynamics submission required for confirmation display |

### 9.2 Lead Payload

All leads submitted to Microsoft Dynamics include:

- Standard contact fields (name, phone, email, address)
- Service type classification
- Display type
- Areas of Interest selection (if applicable)
- Source page URL and placement context (Hub / Department / Service)
- Dynamics Campaign ID (if configured)
- Submission timestamp

### 9.3 Reliability Requirements

- Failed submissions must be queued for retry; user must not see an error state for transient failures
- Duplicate lead handling to be defined with Dynamics CRM team
- Submission monitoring and alerting required for failure rate thresholds

---

## 10. Non-Functional Requirements

**Performance:** CAAS widget JavaScript must load asynchronously. API call for service type determination must resolve within 2 seconds; loading state displayed during resolution. Widget must not impact Core Web Vitals on host pages.

**Accessibility:** Full keyboard navigation across all steps. All form inputs have appropriate labels. Error states are announced to screen readers. Scheduler is keyboard-accessible. WCAG 2.1 AA compliance required.

**Analytics:** Step-level events tracked for every advance, back navigation, and abandonment. Service type, display type, category, subcategory, and AOI selection captured as event dimensions. Scheduler interaction, confirmation view, and unavailability state tracked as conversion/terminal events.

**CMS authoring:** Widget placement must be configurable by content authors without engineering support. Service configurations managed via CAAS Config Dashboard independently of widget code deployments.

**TCPA compliance:** Automated contact consent must be explicitly captured and displayed on all form variants. Consent language reviewed by legal before launch.

---

## 11. Dependencies & Risks

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| CAAS API reliability | Medium | High | Fallback to manual service type selection if API call fails; error state with retry |
| Microsoft Dynamics field mapping across service types | Medium | High | Define Dynamics lead schema with CRM team before development begins; validate mapping in staging |
| Scheduler integration complexity | Medium | Medium | Define availability data source and integration contract early; scope as P1 to protect launch timeline if needed |
| Author adoption — CAAS dashboard maintenance | Medium | Medium | Provide authoring guidelines; establish content governance with Installation Services team |
| Scope creep from 40+ subcategory variations | High | Medium | Lock service type classification to three types via CAAS; route edge cases to Third-Party Contractor flow as default |
| Landing page consolidation coordination | High | Medium | Phase consolidation post-widget launch; establish redirect plan with SEO before any pages are retired |
| Legacy Salesforce data migration | Low | Medium | Determine with CRM team whether historical Salesforce leads need migration to Dynamics or can remain in legacy system |

---

## 12. Future Considerations

| Capability | Description |
|---|---|
| Real-time installer availability | Replace representative availability windows with live installer/contractor availability data in scheduler |
| Quote estimation | Surface estimated price range based on service type and scope before lead submission |
| Post-submission status tracking | Allow customers to check lead status, appointment confirmation, and job progress after submission |
| Authenticated experience | Pre-fill customer information for logged-in account holders; associate leads with customer account |
| Pro-specific flow | Dedicated CAAS widget configuration for Pro customers with multi-unit or multi-site scope requirements |
| Full landing page consolidation | Retire remaining individual subcategory landing pages; redirect traffic to department pages with embedded widget |

---

## 13. Discovery Status

This initiative reached an advanced state of discovery before being deprioritized in favor of the platform migration.

| Milestone | Status |
|---|---|
| Stakeholder discovery & problem definition | ✅ Complete |
| Competitive & comparative analysis | ✅ Complete |
| Product vision artifacts (flow diagrams, wireframes, page context) | ✅ Complete |
| UX prototypes (full fidelity — owned by UX team) | ✅ Complete |
| User testing | ✅ Complete |
| CAAS Config Dashboard wireframes | ✅ Complete |
| Development elaboration | 🟡 In Progress at deprioritization |
| Development sprint start | 🔴 Not started — initiative deprioritized |

The decision to postpone was made in the context of the broader platform migration being elevated to a business-critical priority following the Black Friday outage. The widget was scoped as a post-migration initiative with a validated design and clear implementation path ready for pickup.
