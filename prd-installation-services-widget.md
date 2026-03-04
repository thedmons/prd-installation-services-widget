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
| **Stakeholders** | Installation Services Business Unit · UX · Engineering · Analytics · Salesforce CRM Team |

---

## Table of Contents

1. [Overview](#1-overview)
2. [Background & Problem Statement](#2-background--problem-statement)
3. [Goals & Success Metrics](#3-goals--success-metrics)
4. [Scope](#4-scope)
5. [User Personas](#5-user-personas)
6. [Service Types](#6-service-types)
7. [Widget Flow & Functional Requirements](#7-widget-flow--functional-requirements)
8. [Content Model & Configuration](#8-content-model--configuration)
9. [Non-Functional Requirements](#9-non-functional-requirements)
10. [Dependencies & Risks](#10-dependencies--risks)
11. [Future Considerations](#11-future-considerations)
12. [Discovery Status](#12-discovery-status)

---

## 1. Overview

This document defines the product requirements for an Installation Services Lead Capture Widget — a configurable, stepped-form experience designed to consolidate a fragmented set of 40+ individual installation service forms across 7 service categories into a single, guided flow that routes customers to the correct data collection experience based on their selected service type.

The widget was designed to operate in two modes: as a standalone end-to-end experience, and as a configurable component that could be scoped to a specific service or category and embedded on category landing pages — bringing the lead capture experience closer to where customers were already browsing and most likely to convert.

This initiative reached full UX prototype and user testing completion and was in active elaboration with development before being deprioritized in favor of the platform migration. It represents a validated product vision with a clear implementation path.

---

## 2. Background & Problem Statement

### 2.1 Current State

Installation services were represented across 7 overarching service categories and approximately 40 individual subcategories, each with their own landing page and lead capture form. These forms had been authored independently in the CMS over time, resulting in:

- **No consistency in form design or fields** — form structure, required fields, and UX varied widely across subcategories with no shared standard
- **Inconsistent lead routing** — some forms submitted leads directly to Salesforce; others generated email notifications; some provided no submission confirmation and left customers at a dead end
- **No analytics instrumentation** — form submission was the only tracked signal; there was no visibility into where customers dropped off, which fields caused friction, or how completion rates varied across service types
- **High maintenance overhead** — 40+ individually authored forms required significant ongoing effort to maintain, update, and keep consistent with business changes
- **Misconfigurations causing lead failures** — inconsistent routing logic meant some leads were never received by the internal team, resulting in follow-up failures and missed business

### 2.2 The Opportunity

A single, configurable widget — guided by a stepped flow that determined service type via API and adapted its data collection accordingly — could replace the fragmented form landscape with a consistent, measurable, and maintainable experience. Rather than maintaining 40+ forms, the widget would be configured once per service type and placed in higher-converting contexts across the platform.

The consolidation also created an opportunity to establish Salesforce as the single lead endpoint — eliminating the routing inconsistencies that were causing follow-up failures and giving the Installation Services business unit a unified view of inbound leads.

### 2.3 Strategic Context

This initiative was scoped during the broader consolidation of the retailer's DIY and Pro web platforms onto a single unified platform. Installation services represented a high-value, high-complexity product category with direct revenue implications. The fragmented lead capture experience was both a customer friction point and an operational liability. The initiative was deprioritized — not cancelled — when the platform migration was accelerated following the Black Friday outage, with the intent to revisit post-migration.

---

## 3. Goals & Success Metrics

### 3.1 Goals

- Replace 40+ individually maintained installation service forms with a single configurable widget
- Guide customers to the correct data collection experience based on service type — eliminating dead ends and misconfigured routing
- Consolidate all installation service leads to a single Salesforce endpoint
- Establish baseline analytics on customer flow, drop-off, and form completion across service types
- Reduce CMS authoring and maintenance overhead for the Installation Services category
- Increase lead capture by placing the widget in higher-converting contexts on category and landing pages

### 3.2 Success Metrics

| Metric | Baseline | Target | Measurement |
|---|---|---|---|
| Lead routing accuracy | Unknown — no consistent endpoint | 100% of submissions reach Salesforce | Salesforce lead ingestion monitoring |
| Form completion rate | Unknown — no drop-off tracking | Establish baseline; improve vs. legacy | Analytics funnel tracking per step |
| Step-level drop-off rate | Unknown | Establish baseline by service type | Analytics event tracking per step |
| Submission confirmation rate | Inconsistent — many forms had none | 100% of submissions show confirmation | QA validation |
| CMS forms requiring individual maintenance | 40+ | Reduced to widget configurations per service type | Content team reporting |

---

## 4. Scope

### 4.1 In Scope

- **Stepped widget flow:** Category → Subcategory → Service description → Service type determination (API) → Data collection → Scheduler (where applicable) → Confirmation
- **Three service type variants:** Measurement required · Third-party contractor · Simple install (see Section 6)
- **Dual configuration mode:** Standalone (full flow, start to finish) and embedded (scoped to a specific service or category, placed on existing landing pages)
- **Salesforce integration:** All submissions routed to Salesforce as the single lead endpoint regardless of service type
- **Scheduler integration:** Appointment scheduling for Measurement and Simple Install service types
- **Submission confirmation:** Consistent confirmation state for all service types
- **Analytics instrumentation:** Step-level event tracking for funnel visibility and drop-off analysis
- **CMS authoring:** Widget configurable by content authors — service scope, placement mode, and field requirements set at configuration level

### 4.2 Out of Scope

- Real-time pricing or quote generation
- Online payment or deposit collection
- Contractor assignment or job management (handled by Installation Services business unit post-lead)
- Customer account integration or authenticated experience
- Native app surface
- Post-submission status tracking (lead progress, appointment confirmation)
- Full replacement of all 40+ landing pages in V1 — phased consolidation approach

---

## 5. User Personas

### 5.1 The Installation Customer

A homeowner or renter who has decided they want a product installed but is uncertain about the process, cost, or timeline. They may be browsing installation services from a product page, a category page, or a direct search. They have high purchase intent but low tolerance for friction — a confusing or dead-end form experience will cause them to disengage or call a store instead.

- **Needs:** Clear guidance on what the service involves; confidence that their request was received; a simple way to get started without needing to know which specific service they need
- **Success:** Completes the widget flow, receives confirmation, and is contacted by the Installation Services team within the expected timeframe

### 5.2 The Pro Customer

A contractor or small business owner requesting installation services at scale — potentially for multiple units or job sites. More likely to have specific scope requirements and to value a professional, process-driven experience over a consumer-friendly one.

- **Needs:** Ability to describe project scope; clear expectation setting on follow-up process
- **Success:** Submits a complete scope description; lead is received and routed correctly in Salesforce

### 5.3 The Installation Services Coordinator

An internal team member responsible for managing inbound installation leads, coordinating with contractors, stores, and in-house installers, and ensuring customer follow-up. Currently operating with inconsistent lead data across multiple routing mechanisms.

- **Needs:** Consistent, complete lead data in a single Salesforce view; no missed leads from misconfigured forms; accurate service type classification to route to the right fulfillment path
- **Success:** All inbound leads appear in Salesforce with complete fields and correct service type; zero dead-end or unrouted submissions

---

## 6. Service Types

Service type is determined after subcategory selection via an API call that classifies the selected service. The widget adapts its data collection flow based on the returned service type. Three types are defined:

### 6.1 Measurement Required

Services that require an in-person measurement visit before work can begin (e.g., flooring, countertops, custom window treatments). The measurement appointment is the first step in the fulfillment process.

**Required fields:** Name · Phone number · Email · Service address · Preferred appointment date/time (via scheduler)

**Flow:** Standard fields → Address → Scheduler → Confirmation

### 6.2 Third-Party Contractor

Services fulfilled entirely by third-party contractors where scope and project details are needed to route and quote the job accurately (e.g., HVAC, electrical, plumbing).

**Required fields:** Name · Phone number · Email · Service address · Project scope / description

**Flow:** Standard fields → Address → Scope description → Confirmation

### 6.3 Simple Install

Standard appliance and fixture installations that do not require measurement and are fulfilled by in-house or store-coordinated installers (e.g., dishwasher, washing machine, light fixtures).

**Required fields:** Name · Phone number · Email · Service address · Preferred appointment date/time (via scheduler)

**Flow:** Standard fields → Address → Scheduler → Confirmation

---

## 7. Widget Flow & Functional Requirements

### 7.1 Stepped Flow

```
Step 1: Category Selection
  └─ Step 2: Subcategory Selection
        └─ Step 3: Service Description & "Get Started" CTA
              └─ [API call → Service type determination]
                    └─ Step 4: Information Collection (adapts by service type)
                          └─ Step 5: Scheduler (Measurement & Simple Install only)
                                └─ Step 6: Confirmation
```

### 7.2 Functional Requirements by Step

| Step | Feature | Description | Priority |
|---|---|---|---|
| 1 | Category selection | Visual card or list selection of 7 overarching installation categories | P0 |
| 1 | Back navigation | User can return to previous step at any point | P0 |
| 2 | Subcategory selection | Filtered list of subcategories based on category selection | P0 |
| 2 | Search/filter | Optional filter to find subcategory by keyword | P1 |
| 3 | Service description | Full description of selected service — what it includes, what to expect, typical timeline | P0 |
| 3 | Get Started CTA | Primary action triggers API call for service type determination | P0 |
| 3 | Loading state | Visible loading indicator during API call | P0 |
| 4 | Standard fields | Name, phone number, email — required for all service types | P0 |
| 4 | Address fields | Street, city, state, zip — required for all service types; used for lead routing | P0 |
| 4 | Scope description | Free-text field for project scope — required for Third-Party Contractor | P0 |
| 4 | Field validation | Inline validation on all required fields before step advance | P0 |
| 5 | Scheduler | Date/time selection for Measurement and Simple Install appointments | P0 |
| 5 | Availability integration | Scheduler reflects real or representative availability windows | P1 |
| 6 | Confirmation state | Consistent confirmation message for all service types — submission received, next steps, expected contact timeframe | P0 |
| 6 | Salesforce submission | All leads submitted to Salesforce on confirmation; service type captured as lead attribute | P0 |
| All | Progress indicator | Step indicator showing current position in flow | P1 |
| All | Mobile optimization | Full-width, touch-friendly inputs; scheduler optimized for mobile viewport | P0 |

### 7.3 Widget Configuration Modes

| Mode | Description | Use Case |
|---|---|---|
| Standalone | Full flow from category selection through confirmation | Dedicated installation services landing page; homepage placement |
| Embedded — category scoped | Widget pre-set to a specific category; begins at subcategory selection | Category landing pages (e.g., Flooring, Appliances) |
| Embedded — service scoped | Widget pre-set to a specific subcategory; begins at service description | Individual service landing pages |

---

## 8. Content Model & Configuration

Widget instances are configured by content authors in the CMS. Configuration fields:

| Field | Description | Required |
|---|---|---|
| Widget mode | Standalone · Category scoped · Service scoped | Yes |
| Pre-selected category | For category-scoped and service-scoped modes | Conditional |
| Pre-selected subcategory | For service-scoped mode | Conditional |
| Service descriptions | Rich text per subcategory — authored and maintained in CMS | Yes |
| Confirmation message | Configurable per service type; includes expected follow-up timeframe | Yes |
| Salesforce campaign ID | Optional campaign attribution field passed with lead submission | No |

---

## 9. Non-Functional Requirements

**Performance:** Widget JavaScript must load asynchronously. API call for service type determination must resolve within 2 seconds; loading state displayed during resolution. Widget must not impact Core Web Vitals on host pages.

**Accessibility:** Full keyboard navigation across all steps. All form inputs have appropriate labels. Error states are announced to screen readers. Scheduler is keyboard-accessible. WCAG 2.1 AA compliance required.

**Salesforce integration:** All form submissions must reach Salesforce reliably. Failed submissions must be queued for retry. Service type, campaign ID, and step-completion metadata must be passed as lead attributes. Duplicate lead handling to be defined with Salesforce CRM team.

**Analytics:** Step-level events tracked for every advance, back navigation, and abandonment. Service type, category, and subcategory captured as event dimensions. Scheduler interaction and confirmation view tracked as conversion events.

**CMS authoring:** Widget configuration must be manageable by content authors without engineering support. Service descriptions and confirmation copy must be updatable independently of widget code.

---

## 10. Dependencies & Risks

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Service type API reliability | Medium | High | Fallback to manual service type selection if API call fails; error state with retry |
| Scheduler integration complexity | Medium | Medium | Define availability data source and integration contract early; scope scheduler as P1 if needed to protect launch timeline |
| Salesforce field mapping across service types | Medium | High | Define Salesforce lead schema with CRM team before development begins; validate mapping in staging |
| Author adoption — service description maintenance | Medium | Medium | Provide authoring guidelines and templates; establish content governance with Installation Services team |
| Scope creep from 40+ subcategory variations | High | Medium | Lock service type classification to three types; route edge cases to Third-Party Contractor flow as default |
| Landing page consolidation coordination | High | Medium | Phase consolidation post-widget launch; establish redirect plan with SEO before any pages are retired |

---

## 11. Future Considerations

| Capability | Description |
|---|---|
| Real-time availability | Replace representative availability windows with live installer/contractor availability data |
| Quote estimation | Surface estimated price range based on service type and scope before lead submission |
| Post-submission status tracking | Allow customers to check lead status, appointment confirmation, and job progress after submission |
| Authenticated experience | Pre-fill customer information for logged-in account holders; associate leads with customer account |
| Pro-specific flow | Dedicated widget configuration for Pro customers with multi-unit or multi-site scope requirements |
| Full landing page consolidation | Retire remaining individual subcategory landing pages; redirect traffic to category pages with embedded widget |

---

## 12. Discovery Status

This initiative reached an advanced state of discovery before being deprioritized in favor of the platform migration.

| Milestone | Status |
|---|---|
| Stakeholder discovery & problem definition | ✅ Complete |
| Competitive & comparative analysis | ✅ Complete |
| UX prototypes (full flow) | ✅ Complete |
| User testing | ✅ Complete |
| Development elaboration | 🟡 In Progress at deprioritization |
| Development sprint start | 🔴 Not started — initiative deprioritized |

The decision to postpone was made in the context of the broader platform migration being elevated to a business-critical priority following the Black Friday outage. The widget was scoped as a post-migration initiative with a validated design and clear implementation path ready for pickup.
