# API Spec: CAAS Installation Services Widget
### E-Commerce Web Platform — Lead Capture & Configuration APIs

---

| | |
|---|---|
| **Product** | E-Commerce Web Platform — Installation Services |
| **Document Type** | API Specification |
| **Status** | Discovery Complete · Deprioritized (platform migration) |
| **Date** | 2020–2021 |
| **Author** | Digital Product |
| **Related Docs** | [PRD: Installation Services Lead Capture Widget](./prd-installation-services-widget.md) · [System Design: CAAS Architecture](./system-design-caas-installation-services.md) |

---

## Table of Contents

1. [Overview](#1-overview)
2. [Base URL & Versioning](#2-base-url--versioning)
3. [Authentication](#3-authentication)
4. [API 1 — GET /config](#4-api-1--get-config)
5. [API 2 — POST /lead](#5-api-2--post-lead)
6. [API 3 — GET /services](#6-api-3--get-services)
7. [API 4 — PUT /services/{serviceId}](#7-api-4--put-servicesserviceid)
8. [API 5 — POST /services](#8-api-5--post-services)
9. [Error Handling](#9-error-handling)
10. [Microsoft Dynamics CRM Integration](#10-microsoft-dynamics-crm-integration)

---

## 1. Overview

This document defines the API contracts for the CAAS (Content as a Service) Installation Services Widget. Four APIs support the widget's runtime behavior and one supports internal configuration management:

| API | Method | Path | Purpose |
|---|---|---|---|
| Config | GET | `/api/caas/config` | Retrieve service configuration for widget runtime |
| Lead | POST | `/api/caas/lead` | Submit lead to Microsoft Dynamics |
| Service List | GET | `/api/caas/services` | List all service configurations (Config Dashboard) |
| Service Update | PUT | `/api/caas/services/{serviceId}` | Update service configuration (Config Dashboard) |
| Service Create | POST | `/api/caas/services` | Create new service configuration (Config Dashboard) |

---

## 2. Base URL & Versioning

```
Production:   https://api.example-retailer.com/caas/v1
Staging:      https://api-staging.example-retailer.com/caas/v1
```

API versioning is path-based (`/v1`). Breaking changes increment the version. Non-breaking additions (new optional fields) do not require version increment.

---

## 3. Authentication

**Widget APIs (Config, Lead):** No authentication required. APIs are public-facing and called from the browser. Rate limiting applied per IP.

**Config Dashboard APIs (Service List, Update, Create):** Require internal SSO authentication via bearer token.

```
Authorization: Bearer {sso_token}
```

---

## 4. API 1 — GET /config

Retrieves the widget configuration for a given service placement context. Called on page load to determine service type, display type, step descriptions, and local service availability.

### Request

```
GET /api/caas/v1/config
```

**Query Parameters**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `category` | string | Yes | Category slug (e.g., `flooring`, `appliances`) |
| `subcategory` | string | No | Subcategory slug; omit for department-level placement |
| `placement` | enum | Yes | `hub` · `department` · `service` |
| `zip` | string | Yes | Customer zip code — used for local service availability check |

**Example Request**

```
GET /api/caas/v1/config
  ?category=flooring
  &subcategory=hardwood
  &placement=service
  &zip=28105
```

### Response — 200 OK

```json
{
  "serviceId": "flooring-hardwood",
  "serviceName": "Hardwood Flooring Installation",
  "serviceType": "measurement_required",
  "displayType": "form",
  "localServiceAvailable": true,
  "areasOfInterest": [
    {
      "id": "vinyl-plank",
      "label": "Vinyl Plank",
      "imageUrl": "https://cdn.example-retailer.com/caas/aoi/vinyl-plank.jpg"
    },
    {
      "id": "hardwood",
      "label": "Hardwood",
      "imageUrl": "https://cdn.example-retailer.com/caas/aoi/hardwood.jpg"
    },
    {
      "id": "carpet",
      "label": "Carpet",
      "imageUrl": "https://cdn.example-retailer.com/caas/aoi/carpet.jpg"
    },
    {
      "id": "tile",
      "label": "Tile",
      "imageUrl": "https://cdn.example-retailer.com/caas/aoi/tile.jpg"
    }
  ],
  "stepDescriptions": {
    "consultation": "A professional installer will visit your home to take precise measurements.",
    "scheduling": null,
    "quoting": "Receive a detailed quote based on your measurement and material selections.",
    "installation": "Our independent installer will complete your project and ensure you're 100% satisfied."
  },
  "dynamicsCampaignId": "CAAS-FLOORING-2021-Q1",
  "landingUrl": "/flooring/hardwood-flooring/"
}
```

**Response — 200 OK (Local Service Unavailable)**

```json
{
  "serviceId": "flooring-hardwood",
  "serviceName": "Hardwood Flooring Installation",
  "serviceType": "measurement_required",
  "displayType": "form",
  "localServiceAvailable": false,
  "areasOfInterest": [],
  "stepDescriptions": { ... },
  "dynamicsCampaignId": null,
  "landingUrl": "/flooring/hardwood-flooring/"
}
```

Widget renders unavailability state when `localServiceAvailable: false`. Does not proceed to lead capture.

### Response Fields

| Field | Type | Description |
|---|---|---|
| `serviceId` | string | Unique service identifier |
| `serviceName` | string | Display name for the service |
| `serviceType` | enum | `measurement_required` · `third_party_contractor` · `simple_install` |
| `displayType` | enum | `form` · `detail_fee` · `scheduling_lead` |
| `localServiceAvailable` | boolean | Whether service is available in customer's zip code |
| `areasOfInterest` | array | Optional subcategory options; empty array if not applicable |
| `stepDescriptions` | object | CAAS-configured step copy; null fields use default copy |
| `dynamicsCampaignId` | string \| null | Dynamics campaign attribution ID |
| `landingUrl` | string | URL used for banner image reference |

### Error Responses

| Status | Code | Description |
|---|---|---|
| 400 | `MISSING_PARAMETER` | Required query parameter absent |
| 400 | `INVALID_PLACEMENT` | Placement value not in allowed enum |
| 404 | `SERVICE_NOT_FOUND` | No configuration found for category/subcategory combination |
| 503 | `CONFIG_SERVICE_UNAVAILABLE` | CAAS config service temporarily unavailable |

---

## 5. API 2 — POST /lead

Submits a completed lead from the widget to Microsoft Dynamics CRM. Called on user form submission after client-side validation passes.

### Request

```
POST /api/caas/v1/lead
Content-Type: application/json
```

**Request Body**

```json
{
  "serviceId": "flooring-hardwood",
  "serviceType": "measurement_required",
  "displayType": "form",
  "placementContext": "service",
  "sourceUrl": "https://www.example-retailer.com/l/install/hardwood-flooring.html",
  "dynamicsCampaignId": "CAAS-FLOORING-2021-Q1",
  "contact": {
    "firstName": "Jane",
    "lastName": "Smith",
    "phone": "7045550100",
    "email": "jane.smith@example.com",
    "address": {
      "street": "123 Main St",
      "city": "Charlotte",
      "state": "NC",
      "zip": "28105"
    }
  },
  "projectScope": null,
  "areasOfInterest": ["hardwood"],
  "appointment": {
    "preferredDate": "2021-06-15",
    "preferredTimeSlot": "morning"
  },
  "consentGiven": true,
  "consentTimestamp": "2021-06-01T14:23:11Z"
}
```

**Request Fields**

| Field | Type | Required | Description |
|---|---|---|---|
| `serviceId` | string | Yes | From config response |
| `serviceType` | enum | Yes | From config response |
| `displayType` | enum | Yes | From config response |
| `placementContext` | enum | Yes | `hub` · `department` · `service` |
| `sourceUrl` | string | Yes | Full URL of page where widget was submitted |
| `dynamicsCampaignId` | string \| null | No | From config response |
| `contact.firstName` | string | Yes | |
| `contact.lastName` | string | Yes | |
| `contact.phone` | string | Yes | 10-digit, digits only |
| `contact.email` | string | Yes | Valid email format |
| `contact.address.street` | string | Yes | Required for all service types (geographic routing) |
| `contact.address.city` | string | Yes | |
| `contact.address.state` | string | Yes | 2-letter state code |
| `contact.address.zip` | string | Yes | 5-digit zip |
| `projectScope` | string \| null | Conditional | Required when `serviceType: third_party_contractor` |
| `areasOfInterest` | array | No | AOI IDs selected; empty array if not applicable |
| `appointment.preferredDate` | string | Conditional | ISO date; required when `serviceType: measurement_required` or `simple_install` |
| `appointment.preferredTimeSlot` | enum \| null | No | `morning` · `afternoon` · `evening` |
| `consentGiven` | boolean | Yes | Must be `true`; reject if false |
| `consentTimestamp` | string | Yes | ISO 8601 UTC timestamp of consent |

### Response — 201 Created

```json
{
  "leadId": "CAAS-2021-00041823",
  "status": "received",
  "message": "We'll be in touch soon.",
  "dynamicsLeadId": "a8b3c4d5-1234-5678-abcd-ef0123456789"
}
```

### Response — 202 Accepted (Dynamics submission queued)

Returned when Dynamics API is temporarily unavailable and submission has been queued for retry:

```json
{
  "leadId": "CAAS-2021-00041824",
  "status": "queued",
  "message": "We'll be in touch soon.",
  "dynamicsLeadId": null
}
```

**Note:** Widget renders confirmation state for both `201` and `202` responses. Retry failures are handled server-side and not surfaced to the user.

### Error Responses

| Status | Code | Description |
|---|---|---|
| 400 | `MISSING_REQUIRED_FIELD` | Required field absent |
| 400 | `INVALID_EMAIL` | Email format invalid |
| 400 | `INVALID_PHONE` | Phone format invalid |
| 400 | `CONSENT_NOT_GIVEN` | `consentGiven` is false |
| 400 | `SCOPE_REQUIRED` | `projectScope` missing for third-party contractor |
| 429 | `RATE_LIMIT_EXCEEDED` | Too many submissions from IP |
| 503 | `LEAD_SERVICE_UNAVAILABLE` | Lead service temporarily unavailable |

---

## 6. API 3 — GET /services

Returns all service configurations. Used by the CAAS Config Dashboard. Requires SSO authentication.

### Request

```
GET /api/caas/v1/services
Authorization: Bearer {sso_token}
```

**Query Parameters**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `search` | string | No | Full-text search across service name and category |
| `serviceType` | enum | No | Filter by `measurement_required` · `third_party_contractor` · `simple_install` |
| `category` | string | No | Filter by category slug |
| `limit` | integer | No | Page size (default: 25, max: 100) |
| `offset` | integer | No | Pagination offset (default: 0) |

### Response — 200 OK

```json
{
  "total": 42,
  "limit": 25,
  "offset": 0,
  "services": [
    {
      "serviceId": "flooring-hardwood",
      "serviceName": "Hardwood Flooring Installation",
      "category": "flooring",
      "subcategory": "hardwood",
      "serviceType": "measurement_required",
      "displayType": "form",
      "updatedAt": "2021-05-14T09:32:00Z",
      "updatedBy": "author@example-retailer.com"
    }
  ]
}
```

---

## 7. API 4 — PUT /services/{serviceId}

Updates an existing service configuration. Used by the CAAS Config Dashboard. Requires SSO authentication.

### Request

```
PUT /api/caas/v1/services/{serviceId}
Authorization: Bearer {sso_token}
Content-Type: application/json
```

**Request Body**

```json
{
  "serviceName": "Hardwood Flooring Installation",
  "serviceType": "measurement_required",
  "displayType": "form",
  "landingUrl": "/flooring/hardwood-flooring/",
  "stepDescriptions": {
    "consultation": "A professional installer will visit your home to take precise measurements.",
    "scheduling": null,
    "quoting": "Receive a detailed quote based on your measurement and material selections.",
    "installation": "Our independent installer will complete your project and ensure you're 100% satisfied."
  },
  "areasOfInterest": [
    { "id": "vinyl-plank", "label": "Vinyl Plank", "imageUrl": "https://cdn.example-retailer.com/caas/aoi/vinyl-plank.jpg" },
    { "id": "hardwood", "label": "Hardwood", "imageUrl": "https://cdn.example-retailer.com/caas/aoi/hardwood.jpg" }
  ],
  "dynamicsCampaignId": "CAAS-FLOORING-2021-Q2"
}
```

### Response — 200 OK

```json
{
  "serviceId": "flooring-hardwood",
  "updatedAt": "2021-06-01T10:15:00Z",
  "updatedBy": "author@example-retailer.com"
}
```

---

## 8. API 5 — POST /services

Creates a new service configuration. Used by the CAAS Config Dashboard. Requires SSO authentication.

### Request

```
POST /api/caas/v1/services
Authorization: Bearer {sso_token}
Content-Type: application/json
```

Request body follows the same schema as PUT /services/{serviceId}, with the addition of:

| Field | Type | Required | Description |
|---|---|---|---|
| `category` | string | Yes | Category slug |
| `subcategory` | string | Yes | Subcategory slug — must be unique |

### Response — 201 Created

```json
{
  "serviceId": "appliances-dishwasher",
  "createdAt": "2021-06-01T11:00:00Z",
  "createdBy": "author@example-retailer.com"
}
```

### Error Responses

| Status | Code | Description |
|---|---|---|
| 409 | `SERVICE_ALREADY_EXISTS` | Configuration for this category/subcategory combination already exists |

---

## 9. Error Handling

### Standard Error Response Format

All errors return a consistent JSON structure:

```json
{
  "error": {
    "code": "MISSING_REQUIRED_FIELD",
    "message": "The field 'contact.email' is required.",
    "field": "contact.email"
  }
}
```

### Rate Limiting

Widget-facing APIs (Config, Lead) are rate limited per IP:

| API | Rate Limit |
|---|---|
| GET /config | 60 requests/minute |
| POST /lead | 10 requests/minute |

Rate limit responses return HTTP 429 with `Retry-After` header.

---

## 10. Microsoft Dynamics CRM Integration

### 10.1 Lead Submission to Dynamics

The CAAS Lead API proxies submissions to Microsoft Dynamics via the Dynamics 365 Web API. CAAS handles authentication, field mapping, and retry logic — the widget never calls Dynamics directly.

**Dynamics endpoint:**
```
POST https://[org].crm.dynamics.com/api/data/v9.2/leads
```

### 10.2 Field Mapping

| CAAS Field | Dynamics Field | Type |
|---|---|---|
| `contact.firstName` | `firstname` | string |
| `contact.lastName` | `lastname` | string |
| `contact.phone` | `telephone1` | string |
| `contact.email` | `emailaddress1` | string |
| `contact.address.street` | `address1_line1` | string |
| `contact.address.city` | `address1_city` | string |
| `contact.address.state` | `address1_stateorprovince` | string |
| `contact.address.zip` | `address1_postalcode` | string |
| `projectScope` | `description` | string |
| `serviceType` | `new_servicetype` | custom optionset |
| `displayType` | `new_displaytype` | custom optionset |
| `areasOfInterest` | `new_areasofinterest` | custom string (comma-separated) |
| `sourceUrl` | `new_sourceurl` | custom string |
| `placementContext` | `new_placementcontext` | custom optionset |
| `dynamicsCampaignId` | `campaignid` | lookup |
| `appointment.preferredDate` | `new_appointmentdate` | custom datetime |
| `appointment.preferredTimeSlot` | `new_appointmenttimeslot` | custom optionset |
| `consentGiven` | `new_tcpaconsentgiven` | custom boolean |
| `consentTimestamp` | `new_tcpaconsenttimestamp` | custom datetime |

### 10.3 Dynamics Authentication

CAAS authenticates to Dynamics via OAuth 2.0 client credentials flow (server-to-server). Credentials are stored in environment secrets and never exposed to the client.

```
POST https://login.microsoftonline.com/{tenantId}/oauth2/v2.0/token
grant_type=client_credentials
client_id={clientId}
client_secret={clientSecret}
scope=https://[org].crm.dynamics.com/.default
```

Token is cached server-side and refreshed before expiry.
