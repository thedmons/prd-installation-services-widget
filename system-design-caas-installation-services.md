# System Design: CAAS Installation Services Widget
### E-Commerce Web Platform — Content as a Service Architecture

---

| | |
|---|---|
| **Product** | E-Commerce Web Platform — Installation Services |
| **Document Type** | System Design |
| **Status** | Discovery Complete · Deprioritized (platform migration) |
| **Date** | 2020–2021 |
| **Surface** | Desktop web · Mobile web |
| **Author** | Digital Product |
| **Related Docs** | [PRD: Installation Services Lead Capture Widget](./prd-installation-services-widget.md) · [API Spec: CAAS Lead Capture](./api-spec-caas-installation-services.md) |

---

## Table of Contents

1. [Overview](#1-overview)
2. [Architecture Overview](#2-architecture-overview)
3. [Component Architecture](#3-component-architecture)
4. [CAAS Configuration Service](#4-caas-configuration-service)
5. [Widget State Machine](#5-widget-state-machine)
6. [Data Flow](#6-data-flow)
7. [CRM Integration — Microsoft Dynamics](#7-crm-integration--microsoft-dynamics)
8. [CAAS Config Dashboard](#8-caas-config-dashboard)
9. [Analytics Architecture](#9-analytics-architecture)
10. [Performance & Reliability](#10-performance--reliability)
11. [Future State Evolution](#11-future-state-evolution)

---

## 1. Overview

The CAAS (Content as a Service) Installation Services Widget is a configurable, API-driven lead capture system designed to replace 40+ individually maintained installation service forms with a single adaptive experience. This document describes the system architecture — component structure, configuration service, data flows, CRM integration, and admin tooling.

The architecture is built around a central principle: **configuration over code**. Service-specific behavior (form type, display type, step descriptions, CTA copy) is stored in the CAAS configuration service and retrieved at runtime — meaning new services can be added and existing services updated without code deploys.

---

## 2. Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                        Browser / Client                      │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              CAAS Widget (React)                     │   │
│  │                                                     │   │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────────────┐  │   │
│  │  │ Zipped   │  │ Stepper  │  │  Form / Scheduler │  │   │
│  │  │ State    │  │ UI       │  │  Component        │  │   │
│  │  └──────────┘  └──────────┘  └──────────────────┘  │   │
│  └───────────────────────┬─────────────────────────────┘   │
└──────────────────────────┼──────────────────────────────────┘
                           │
          ┌────────────────┼────────────────┐
          │                │                │
          ▼                ▼                ▼
  ┌───────────────┐ ┌─────────────┐ ┌──────────────────┐
  │ CAAS Config   │ │ Scheduler   │ │ Microsoft        │
  │ Service (API) │ │ API         │ │ Dynamics CRM API │
  └───────────────┘ └─────────────┘ └──────────────────┘
          │
          ▼
  ┌───────────────┐
  │ CAAS Config   │
  │ Dashboard     │
  │ (Admin UI)    │
  └───────────────┘
```

---

## 3. Component Architecture

### 3.1 Widget Component Tree

```
<CAASWidget>
  ├── <ZippedOutState>          // Collapsed CTA visible on page load
  ├── <ZippedInState>           // Expanded widget container
  │     ├── <ProgressStepper>   // Step indicator; updates on navigation
  │     ├── <CategorySelector>  // Step 1 — Hub Page standalone only
  │     ├── <SubcategorySelector> // Step 2
  │     ├── <ServiceDescription>  // Step 3 — renders CAAS config content
  │     ├── <AreasOfInterest>   // Optional — rendered when AOI config present
  │     ├── <LeadForm>          // Step 4 — adapts fields by service type
  │     │     ├── <StandardFields>    // Name, phone, email, address (all types)
  │     │     ├── <ScopeField>        // Third-Party Contractor only
  │     │     ├── <ProcessExplanation> // Inline 3-step guide
  │     │     └── <ConsentDisclosure> // TCPA compliance
  │     ├── <Scheduler>         // Step 5 — Measurement & Simple Install
  │     │     └── <PhoneConsultationFallback>
  │     ├── <Confirmation>      // Step 6
  │     │     └── <ProductCarousel>
  │     └── <UnavailabilityState> // Terminal — no local service
```

### 3.2 Display Type Rendering

The widget renders different primary flows based on the `displayType` returned by the CAAS config API:

| Display Type | Primary CTA | Secondary CTA | Scheduler |
|---|---|---|---|
| `form` | Standard lead form submission | Phone consultation link | Per service type |
| `detail_fee` | Contact method selector (form or schedule) | Fee/pricing detail block | On schedule selection |
| `scheduling_lead` | Schedule appointment directly | Phone consultation link | Always |

### 3.3 Page Placement Modes

| Mode | Widget Initial State | Category Pre-set | Subcategory Pre-set |
|---|---|---|---|
| `standalone` | Category selector | No | No |
| `department` | Subcategory selector | Yes | No |
| `service` | Service description | Yes | Yes |

---

## 4. CAAS Configuration Service

### 4.1 Purpose

The CAAS configuration service is the central store for all service-level widget behavior. On page load, the widget requests configuration for the current placement context. The API response drives:

- Which steps to render
- What display type to use
- Step description copy (consultation, scheduling, quoting, installation)
- Whether Areas of Interest are available
- Microsoft Dynamics campaign ID for lead attribution

### 4.2 Configuration Request

```
GET /api/caas/config
  ?category={categorySlug}
  &subcategory={subcategorySlug}   // optional — omit for department-level
  &placement={hub|department|service}
  &zip={zipCode}                   // used for local availability check
```

### 4.3 Configuration Response Schema

```json
{
  "serviceId": "string",
  "serviceName": "string",
  "serviceType": "measurement_required | third_party_contractor | simple_install",
  "displayType": "form | detail_fee | scheduling_lead",
  "localServiceAvailable": true,
  "areasOfInterest": [
    {
      "id": "string",
      "label": "string",
      "imageUrl": "string"
    }
  ],
  "stepDescriptions": {
    "consultation": "string (max 350 chars, optional)",
    "scheduling": "string (max 350 chars, optional)",
    "quoting": "string (max 350 chars, required)",
    "installation": "string (max 350 chars, required)"
  },
  "dynamicsCampaignId": "string (optional)",
  "landingUrl": "string"
}
```

### 4.4 Override Logic

The CAAS service supports category and department-level overrides — a configuration set at the department level can be overridden by a more specific service-level configuration. Override resolution order:

1. Service-level configuration (most specific)
2. Department-level configuration
3. Category-level configuration (least specific)

---

## 5. Widget State Machine

```
INITIAL
  └─► CONFIG_LOADING (CAAS API call on load)
        ├─► CONFIG_ERROR (API failure → error state with retry)
        └─► ZIPPED_OUT (config loaded; collapsed widget displayed)
              └─► ZIPPED_IN (user expands widget)
                    └─► CATEGORY_SELECTION (standalone mode only)
                          └─► SUBCATEGORY_SELECTION
                                └─► SERVICE_TYPE_LOADING (CAAS API call)
                                      ├─► SERVICE_UNAVAILABLE (localServiceAvailable: false)
                                      └─► SERVICE_DESCRIPTION
                                            └─► AOI_SELECTION (if areasOfInterest present)
                                                  └─► FORM_ENTRY (adapts by serviceType)
                                                        ├─► VALIDATION_ERROR (loop back)
                                                        └─► SCHEDULER (measurement / simple_install)
                                                              └─► SUBMISSION_LOADING
                                                                    ├─► SUBMISSION_ERROR (retry queue)
                                                                    └─► CONFIRMATION
```

---

## 6. Data Flow

### 6.1 Page Load Flow

```
1. Page loads → Widget JS loaded asynchronously (non-blocking)
2. Widget mounts → GET /api/caas/config (with placement context + zip)
3. Config response received:
   - localServiceAvailable: false → widget suppressed or unavailability state shown
   - localServiceAvailable: true → widget renders in zipped-out state
4. User expands widget → zipped-in state; step navigation begins
```

### 6.2 Lead Submission Flow

```
1. User completes form fields + scheduler (if applicable)
2. Client-side validation passes
3. POST /api/caas/lead (lead payload assembled)
4. Server-side:
   a. Validate payload
   b. POST to Microsoft Dynamics CRM API
   c. On success → return confirmation response
   d. On failure → queue for retry; return success to user (silent retry)
5. Widget renders confirmation state
6. Analytics submission event fired with service type + display type + AOI
```

---

## 7. CRM Integration — Microsoft Dynamics

### 7.1 Lead Object Mapping

| Widget Field | Dynamics Field | Notes |
|---|---|---|
| First Name | `firstname` | |
| Last Name | `lastname` | |
| Phone Number | `telephone1` | |
| Email Address | `emailaddress1` | |
| Street Address | `address1_line1` | |
| City | `address1_city` | |
| State | `address1_stateorprovince` | |
| ZIP Code | `address1_postalcode` | |
| Project Scope | `description` | Third-Party Contractor only |
| Service Type | `new_servicetype` | Custom field |
| Display Type | `new_displaytype` | Custom field |
| Areas of Interest | `new_areasofinterest` | Custom field; comma-separated |
| Source URL | `new_sourceurl` | Custom field |
| Placement Context | `new_placementcontext` | hub / department / service |
| Campaign ID | `campaignid` | From CAAS config |
| Appointment Date/Time | `new_appointmentdatetime` | Measurement / Simple Install |

### 7.2 Retry Logic

Failed Dynamics submissions are queued server-side with exponential backoff:
- Attempt 1: Immediate
- Attempt 2: 30 seconds
- Attempt 3: 5 minutes
- Attempt 4: 30 minutes
- Dead letter queue after 4 failures; alert triggered

### 7.3 Legacy System Note

Prior to CAAS, installation leads were split between Salesforce and email depending on form configuration. Historical leads in Salesforce are not migrated — CAAS establishes a clean Dynamics dataset from launch date forward.

---

## 8. CAAS Config Dashboard

The Config Dashboard is an internal admin tool for managing service configurations without engineering involvement.

### 8.1 Capabilities

- **Search:** Full-text search across service configurations by name or category
- **Add:** Create new service configuration via multi-tab form
- **Edit:** Update existing configurations (service descriptions, display type, scheduling details)
- **Export:** Download configuration list (CSV)

### 8.2 Config Form Structure

The service configuration form uses a tabbed interface with the following sections:

**Tab 1 — Basic Properties**
- Service Name (required)
- Landing URL (required — used for banner image)
- Service Type selector
- Display Type selector

**Tab 2 — Step Descriptions**
- Consultation Description (350 char, optional)
- Scheduling Description (350 char, optional)
- Quoting Details (350 char, required)
- Installation Description (350 char, required)

**Tab 3 — Advanced**
- Dynamics Campaign ID (optional)
- Areas of Interest configuration (add/remove/reorder)

### 8.3 Access Control

Config Dashboard access is owned by the Installation Services business unit — self-service configuration management is a core design goal. Engineering access is not required for day-to-day configuration updates. Access is provisioned to designated Installation Services staff via SSO.

---

## 9. Analytics Architecture

### 9.1 Event Schema

All widget events are fired to the platform's existing analytics layer (Adobe Analytics) as custom events:

| Event | Trigger | Dimensions |
|---|---|---|
| `caas_widget_load` | Config loaded successfully | serviceType, displayType, placementContext |
| `caas_widget_expand` | User expands zipped-out widget | serviceType, placementContext |
| `caas_step_advance` | User advances to next step | stepName, stepNumber, serviceType |
| `caas_step_back` | User navigates back | stepName, stepNumber |
| `caas_aoi_select` | Areas of Interest selection made | aoiSelection, serviceType |
| `caas_form_start` | User begins form entry | serviceType, displayType |
| `caas_form_abandon` | User leaves widget mid-form | lastStepCompleted, serviceType |
| `caas_scheduler_open` | Scheduler step rendered | serviceType |
| `caas_phone_consultation` | Phone consultation CTA clicked | serviceType |
| `caas_lead_submit` | Successful lead submission | serviceType, displayType, placementContext |
| `caas_service_unavailable` | Local service unavailability state shown | category, zip |

### 9.2 Funnel Reporting

Step-level events enable funnel analysis per service type, display type, and page placement — providing visibility into where customers drop off that did not exist with the legacy form system.

---

## 10. Performance & Reliability

**Async loading:** Widget JavaScript bundle is code-split and loaded asynchronously. Does not block page render or impact LCP/CLS on host pages.

**CAAS API SLA:** Configuration API must respond within 2 seconds. Widget displays loading state during resolution. On timeout or error, widget renders a graceful fallback (direct customer service phone number).

**Zipped-out first render:** Widget renders in collapsed state immediately after config load — no waiting for full widget initialization before something is visible to the user.

**Dynamics submission reliability:** Lead submissions are queued server-side with retry logic. Users see confirmation immediately on first submission attempt — retry failures are handled silently.

**Caching:** CAAS configuration responses are cached at the CDN layer with a short TTL (5 minutes) to reduce API load while keeping configuration updates near-real-time.

---

## 11. Future State Evolution

| Capability | Architectural Implication |
|---|---|
| Real-time installer availability | Scheduler API integration with live availability data source; current implementation uses representative windows |
| Authenticated experience | Widget reads account session token; pre-fills known fields; associates lead with customer account in Dynamics |
| Quote estimation | New CAAS config field for price range; rendered in Detail Fee display type before form |
| Post-submission status tracking | Dynamics lead status exposed via read API; customer-facing status page keyed on lead ID returned at confirmation |
| Native app | Widget logic extracted to shared service layer; app surfaces consume same CAAS config API |
