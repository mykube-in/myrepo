# Technical Specification Document - EAII-472 - IBA Loyalty App API - Azure APIM to Salesforce Integration

## Table of Contents

- 1 Document Overview
  1. 1.1 Background
  2. 1.2 Document Purpose
  3. 1.3 Horizon Impact Assessment
  4. 1.4 Known Issues
  5. 1.5 Glossary
- 2 Scope
  1. 2.1 In Scope
  2. 2.2 Out of Scope
  3. 2.3 Limitations and Constraints
- 3 Compliance
  1. 3.1 Architecture Principles
  2. 3.2 Standards, Guidelines and Patterns
- 4 Requirements Overview
  1. 4.1 Business Process Requirement and Integration
     1. 4.1.1 Process Diagram
     2. 4.1.2 Functional Specification Document
  2. 4.2 Technical Standards and Requirements
  3. 4.3 Required Components
  4. 4.4 Environmental Requirement
- 5 Integration Overview and Implementation
  1. 5.1 EAII-472 - IBA Loyalty App API
     1. 5.1.1 As-is Process(es)
     2. 5.1.2 Proposed Process(es)
        1. 5.1.2.1 Description of Process(es)
        2. 5.1.2.2 Process Flow Diagram
        3. 5.1.2.3 Sequence Diagram
        4. 5.1.2.4 API Endpoints Summary
        5. 5.1.2.5 Interface Requirements / Mapping Details
        6. 5.1.2.6 Authentication and Authorization
        7. 5.1.2.7 Non-functional Requirements
     3. 5.1.3 Coordination
        1. 5.1.3.1 Transaction Boundaries
        2. 5.1.3.2 State Management
     4. 5.1.4 Design Considerations
     5. 5.1.5 Service High Level Design
        1. 5.1.5.1 Component List
        2. 5.1.5.2 Services
        3. 5.1.5.3 APIM Policy Framework
        4. 5.1.5.4 Logging and Monitoring
        5. 5.1.5.5 Service Operations Detailed Mapping
     6. 5.1.6 SMC Onboarding
        1. 5.1.6.1 Integration Onboarding
        2. 5.1.6.2 SMC Load Estimations
     7. 5.1.7 Monitoring
     8. 5.1.8 Reporting
     9. 5.1.9 Integration and Testing Requirements
        1. 5.1.9.1 Development and Integration Strategy
        2. 5.1.9.2 Unit Testing
- 6 Architecture Overview
  1. 6.1 Physical Architecture Diagram
  2. 6.2 Network Connectivity
  3. 6.3 Security
     1. 6.3.1 Data Handling and Encryption
     2. 6.3.2 Authentication Architecture
     3. 6.3.3 IDOR Prevention
  4. 6.4 Infrastructure Support
  5. 6.5 Infrastructure Requirements
- 7 Communication Overview
  1. 7.1 Communication Components
- 8 Data Architecture
  1. 8.1 Data Model
  2. 8.2 Salesforce Data Objects
  3. 8.3 Data Retention
     1. 8.3.1 Data Retention Procedures
     2. 8.3.2 Data Backup
     3. 8.3.3 Data Destruction Procedures
- 9 Deployment and Monitoring Considerations
  1. 9.1 Transition Period
  2. 9.2 Deployment Preference
  3. 9.3 Deployment Environment Migration
- 10 Disaster Recovery
- 11 Support Documentation
- 12 Limitations and Constraints
- 13 Assumptions
- 14 Reference Documents and Architecture
- 15 TSD Approval Checklist

---

|  |  |
| --- | --- |
| **TSD Author** | Integration Team - Metcash |
| **JIRA Ticket** | EAII-472 |
| **API Name** | IBA-LOYALTY-APP |
| **API Version** | v1 |
| **Document Version** | 1.0 |
| **Last Updated** | March 10, 2026 |

|  |  |
| --- | --- |
| **Document Approver** | **Approver Role** |
|  | Integration Platform Representative |
|  | Integration Platform Operations |
|  | Solution Architect |
|  | Test Manager |

---

# 1 Document Overview

## 1.1 Background

Metcash operates three retail loyalty programs under the Independent Brands Australia (IBA) banner:
- **The Bottle-O (TBO)** - Liquor retail loyalty program
- **Cellarbrations** - Premium wine and spirits loyalty program  
- **Porters** - Liquor retail loyalty program

These programs serve customers through mobile applications that require real-time access to loyalty data including:
- Member account information and profile management
- Points balance, transaction history, and redemption tracking
- Store locator and home store preferences
- Product catalogs, categories, and promotional campaigns
- Notification preferences and cross-program enrollment

Currently, all loyalty data resides in **Salesforce Experience Cloud** (Spring '26, API v66.0). Mobile applications require a secure, scalable, and standardized API gateway to access this data while maintaining:
- Strong authentication and authorization controls
- Protection against IDOR (Insecure Direct Object Reference) vulnerabilities
- Per-user and per-IP rate limiting
- Centralized logging and monitoring
- Multi-brand support with brand-specific configurations

This integration establishes **Azure API Management (APIM)** as the API facade between mobile applications and Salesforce, providing enterprise-grade security, observability, and governance.

## 1.2 Document Purpose

This Technical Specification Document (TSD) defines the design, implementation, and operational requirements for the **IBA Loyalty App API** (iba-loyalty/v1), which provides:

**23 REST API Endpoints:**
- **20 Authenticated Endpoints**: Require Salesforce JWT token from Headless Identity login
  - 7 Account operations (profile management, notifications, deactivation, password change)
  - 5 Loyalty operations (points history, bonus points, redemption, cross-program enrollment)
  - 4 Store operations (home store, nearby stores, related stores)
  - 4 Product operations (categories, banners, preferences, promotions)

- **3 Unauthenticated Endpoints**: IP-based rate limiting
  - Store listings (all stores, nearby stores)
  - Email availability check (pre-registration validation)

**Key Technical Features:**
- OAuth 2.1 Authorization Code + PKCE flow for mobile app authentication
- JWT signature validation via Salesforce JWKS endpoints
- Server-side SOQL query construction with semi-join patterns (IDOR prevention)
- Integration User credential management with token caching and auto-refresh
- Per-brand loyalty program mapping (TBO/Cellarbrations/Porters)
- SMC (Solution Monitoring & Control) integration for enterprise observability
- Application Insights diagnostics with PII-safe logging
- WAF integration with field-level exclusions for password security

## 1.3 Horizon Impact Assessment

*List any applications mentioned and their risk colour code for Solution Architect CAB submission.*

|  |  |
| --- | --- |
| **System** | **Colour** |
| Azure API Management (APIM) | Green |
| Salesforce Experience Cloud (Spring '26, API v66.0) | Amber |
| Azure Key Vault | Green |
| Azure Application Insights | Green |
| SMC (Solution Monitoring & Control) | Green |
| Mobile Apps (TBO, Cellarbrations, Porters - iOS/Android) | Amber |

**Salesforce Experience Cloud (Amber):** Integration relies on:
- Salesforce External Client Apps configuration (OAuth 2.0 + PKCE)
- Salesforce Headless Identity API for self-registration and login
- Salesforce JWKS endpoint availability for JWT signature validation
- Salesforce REST API v66.0 SOQL query interface
- Custom Salesforce field schema stability (e.g., `Loyalty_Program_Member_TBO__pc`)

Changes to Salesforce schema, API versioning, or OAuth configuration may impact this integration.

**Mobile Apps (Amber):** Mobile applications must correctly:
- Implement OAuth 2.1 Authorization Code + PKCE flow with Salesforce
- Obtain and cache JWT tokens from Salesforce Headless Identity
- Include JWT in `Authorization: Bearer {jwt}` header for all authenticated endpoints
- Handle 401 Unauthorized responses (token expiration/revocation)
- Respect rate limiting (50 calls/60s per user, or per-IP for unauthenticated)

## 1.4 Known Issues

*Any known issues with the legacy integration that are to be addressed or not.*

|  |  |  |  |
| --- | --- | --- | --- |
| **Issue #** | **Description** | **Status** | **Mitigation** |
| N/A | This is a new integration - no legacy system to migrate from | N/A | N/A |
| EAII-472-001 | Salesforce field naming discrepancies between old mobile app codebase and API specification | Open | CRM team validation required for 8 SOQL queries (see Technical Mapping v2.1 change log) |
| EAII-472-002 | Per-brand product preference field names (TBO/Porters) inferred from naming convention | Open | CRM confirmation needed for `{Category}_{BrandCode}__pc` field pattern |

## 1.5 Glossary

|  |  |
| --- | --- |
| **Item** | **Description** |
| **IBA** | Independent Brands Australia - Metcash retail brand group |
| **TBO** | The Bottle-O - Liquor retail loyalty program |
| **Cellarbrations** | Premium wine and spirits loyalty program (abbreviated as CBN in Salesforce fields) |
| **Porters** | Liquor retail loyalty program |
| **APIM** | Azure API Management - Microsoft's API gateway platform |
| **JWT** | JSON Web Token - Compact, URL-safe means of representing claims between two parties |
| **JWKS** | JSON Web Key Set - Set of public keys used to verify JWT signatures |
| **PKCE** | Proof Key for Code Exchange - OAuth 2.1 security extension for public clients |
| **IDOR** | Insecure Direct Object Reference - Vulnerability where users can access unauthorized resources |
| **SOQL** | Salesforce Object Query Language - SQL-like query language for Salesforce |
| **Semi-Join** | SOQL pattern using subquery in WHERE clause (e.g., `WHERE Id IN (SELECT ... )`) |
| **External Client Apps** | Salesforce OAuth 2.0 configuration (replaces deprecated "Connected Apps" in Spring '26) |
| **Headless Identity** | Salesforce API for self-registration, login, and password management without webview |
| **Integration User** | Salesforce service account used by APIM for backend API calls |
| **SMC** | Solution Monitoring & Control - Metcash enterprise monitoring platform |
| **WAF** | Web Application Firewall - OWASP ModSecurity Core Rule Set protection |
| **GST** | Goods and Services Tax - Australian tax (10%) |
| **MSO** | Member Service Organization - Loyalty program organizational unit |
| **PII** | Personally Identifiable Information - Sensitive user data requiring protection |

---

# 2 Scope

*The scope of the integration defines its objectives and boundaries to ensure stakeholder alignment.*

## 2.1 In Scope

**Functional Scope:**

- **23 REST API Endpoints** (20 authenticated + 3 unauthenticated) covering:
  - Account management (retrieve, update, deactivate, notifications, password change)
  - Loyalty points (transaction history, bonus points, points-to-cash redemption)
  - Cross-program enrollment (idempotent enrollment API)
  - Store operations (all stores, nearby stores, home store, related stores)
  - Product catalog (categories, banners, preferences, promotions)
  - Pre-registration validation (email availability check)

- **Authentication & Authorization:**
  - JWT signature validation via Salesforce JWKS endpoint
  - Per-brand client ID validation (TBO, Cellarbrations, Porters)
  - Integration User credential management with token caching (15-minute TTL)
  - Per-user rate limiting (50 calls/60 seconds)
  - Per-IP rate limiting for unauthenticated endpoints (100 calls/60 seconds)

- **Security Features:**
  - Server-side SOQL query construction (no client-provided IDs)
  - Semi-join patterns for Account ID resolution (IDOR prevention)
  - WAF protection with per-field exclusions for password special characters
  - PII-safe logging (password fields excluded from Application Insights)
  - HTTPS-only communication with certificate validation

- **Data Transformation:**
  - Salesforce response mapping to mobile app schema
  - Per-brand field mapping (e.g., `CBN_Home_Store__pc` vs `TBO_Home_Store__pc`)
  - Date format conversions (Salesforce ISO 8601 to mobile app formats)
  - Error response standardization with correlation IDs

- **Monitoring & Observability:**
  - SMC integration for error logging and alerting
  - Application Insights diagnostics (request/response telemetry)
  - Custom correlation ID tracking (`x-correlation-id` header)
  - Performance metrics (latency, throughput, error rates)

**Technical Scope:**

- Azure API Management configuration (policies, operations, backends, named values)
- Salesforce REST API v66.0 integration (SOQL queries, OAuth token management)
- Azure Key Vault integration for secrets management
- OpenAPI 3.0.1 specification for API contract
- SMC policy fragment for standardized error handling
- Application Insights integration for technical diagnostics

**Data Scope:**

- Salesforce objects: `Account`, `User`, `Contact`, `LoyaltyProgramMember`, `TransactionJournal`, `LoyaltyTierGroup`, `Product2`, `PricebookEntry`, `ProductCategory`, `ProductCategoryProduct`
- Custom Salesforce fields (25+ custom fields across objects)
- Read-only queries for most endpoints (PATCH operations for update/deactivate)
- No data persistence in APIM (stateless gateway pattern)

## 2.2 Out of Scope

**The following are explicitly excluded from this integration:**

- **Identity Operations:** Self-registration, login, password reset, token refresh (handled directly by mobile apps via Salesforce Headless Identity)
- **Mobile App Development:** iOS/Android development, OAuth/PKCE implementation, JWT caching, UI design
- **Salesforce Configuration:** OAuth setup, user provisioning, schema management, Apex workflows (DevOps/Admin responsibility)
- **Business Logic:** Points calculation, promotional eligibility, product catalog management (Salesforce responsibility)
- **Data Migration:** Historical data migration, bulk import/export, data cleansing

## 2.3 Limitations and Constraints

**Authentication:**
- JWT tokens issued by Salesforce only (APIM does not issue tokens)
- Token expiration handled by Salesforce (typically 2-hour TTL)
- No support for API key authentication (JWT required for authenticated endpoints)

**Rate Limiting:**
- Authenticated: 50 calls/60 seconds per user (based on JWT `sub` claim)
- Unauthenticated: 100 calls/60 seconds per source IP address
- Burst allowance: 10 additional requests before rate limit enforcement

**Data Volume:**
- Transaction history: Maximum 100 records per request (pagination via `limit` parameter)
- Product IDs in batch queries: Maximum 20 Product2 IDs per request
- Store search radius: Maximum 100km for nearby stores query

**Salesforce API Limits:**
- Salesforce API daily limits apply (per-org allocation shared across all integrations)
- SOQL query complexity limits (e.g., no nested semi-joins beyond 1 level)
- Salesforce REST API v66.0 functionality constraints

**Supported Brands:**
- Three loyalty programs only: TBO, Cellarbrations, Porters
- No support for additional brands without configuration changes

**Execution Time:**
- Target API response time: < 500ms (95th percentile)
- Maximum timeout: 30 seconds before APIM returns 504 Gateway Timeout

**Availability:**
- Dependent on Salesforce availability (typically 99.9% SLA)
- Planned maintenance windows aligned with Salesforce maintenance schedule

---

# 3 Compliance

## 3.1 Architecture Principles

The following Metcash principles apply to this integration:

- **Security:** Enforce role-based access and governance for secure and compliant integrations.
  - JWT signature validation via Salesforce JWKS
  - Server-side authorization (semi-join SOQL patterns)
  - Secrets stored in Azure Key Vault
  - HTTPS-only communication
  
- **Standardization:** Apply consistent naming, formats, and design patterns across all integrations.
  - Kebab-case for APIM context variables (`user-id`, `loyalty-program-id`)
  - RESTful API design principles
  - OpenAPI 3.0.1 specification
  - Standard HTTP status codes (200, 400, 401, 403, 404, 429, 500, 504)

- **Resilience:** Ensure integrations can handle failures through retries, failover, and recovery.
  - Token auto-refresh for Integration User credentials
  - Circuit breaker pattern for Salesforce backend failures
  - Graceful degradation (e.g., unauthenticated stores endpoints remain available)

- **Observability:** Enable real-time monitoring and logging for visibility and issue detection.
  - SMC integration for error logging and alerting
  - Application Insights diagnostics
  - Correlation ID tracking (`x-correlation-id`)
  - Performance metrics and SLA monitoring

For information related to all architecture principles, refer to:
**[Integration Design Principles - v1.0](https://metcash.atlassian.net/wiki/spaces/MAP/pages/1656913947)**

## 3.2 Standards, Guidelines and Patterns

The following standards, guidelines, and patterns apply:

**[Guidelines](https://metcash.atlassian.net/wiki/spaces/MAP/pages/1668874252)**

**[AIS (Azure Integration Services)](https://metcash.atlassian.net/wiki/spaces/MAP/pages/1638498308)**

**Specific Standards:**
- **API Design:** RESTful principles, JSON payloads, HTTPS-only
- **Authentication:** OAuth 2.1 Authorization Code + PKCE, JWT tokens
- **Error Handling:** RFC 7807 Problem Details for HTTP APIs
- **Logging:** PII-safe logging (exclude passwords, tokens from logs)
- **Rate Limiting:** Token bucket algorithm (per-user and per-IP)
- **Naming Conventions:** Kebab-case URLs, camelCase JSON fields

---

# 4 Requirements Overview

## 4.1 Business Process Requirement and Integration

The IBA Loyalty App API must provide secure, real-time access to loyalty program data for three mobile applications (TBO, Cellarbrations, Porters). The system shall:

1. **Authenticate User Requests:** Validate JWT tokens issued by Salesforce Headless Identity login against Salesforce JWKS public keys.

2. **Authorize Data Access:** Derive user identity from JWT claims and construct SOQL queries server-side to prevent unauthorized data access (IDOR mitigation).

3. **Map Per-Brand Configurations:** Route requests to appropriate Salesforce loyalty program IDs based on JWT client_id claim (TBO/Cellarbrations/Porters).

4. **Retrieve Loyalty Data:** Execute SOQL queries via Integration User credentials to fetch account details, points history, store information, and product catalogs.

5. **Transform Response Data:** Map Salesforce field names and data formats to mobile app schema requirements.

6. **Log and Monitor:** Record request/response telemetry to SMC for error alerting and Application Insights for technical diagnostics.

7. **Enforce Rate Limits:** Throttle excessive API usage (50 calls/60s per user, 100 calls/60s per IP for unauthenticated).

### 4.1.1 Process Diagram

```
┌──────────────────┐
│  Mobile App      │
│  (TBO/CBN/       │
│   Porters)       │
└────────┬─────────┘
         │ 1. Direct OAuth Login
         ├──────────────────────────────┐
         │                              │
         ▼                              ▼
┌──────────────────────┐      ┌─────────────────────┐
│  Salesforce          │      │  Salesforce         │
│  Headless Identity   │      │  OAuth 2.0          │
│  /auth/headless/*    │      │  /oauth2/*          │
└──────────┬───────────┘      └──────────┬──────────┘
           │                             │
           │ 2. JWT Token                │
           └──────────────┬──────────────┘
                          │
                          ▼
                 ┌─────────────────┐
                 │  Mobile App     │
                 │  (Cached JWT)   │
                 └────────┬────────┘
                          │
                          │ 3. API Request with JWT
                          │    Authorization: Bearer {jwt}
                          ▼
                 ┌─────────────────────────────┐
                 │  Azure API Management       │
                 │  (iba-loyalty/v1)           │
                 ├─────────────────────────────┤
                 │  • JWT Validation (JWKS)    │
                 │  • Brand ID Mapping         │
                 │  • Rate Limiting            │
                 │  • Integration Token Cache  │
                 └────────┬────────────────────┘
                          │
                          │ 4. SOQL Query (Integration User Token)
                          ▼
                 ┌─────────────────────────────┐
                 │  Salesforce REST API        │
                 │  /services/data/v66.0/*     │
                 ├─────────────────────────────┤
                 │  • Execute SOQL Query       │
                 │  • Return Records           │
                 └────────┬────────────────────┘
                          │
                          │ 5. Response Data
                          ▼
                 ┌─────────────────────────────┐
                 │  APIM (Outbound Policies)   │
                 ├─────────────────────────────┤
                 │  • Transform Response       │
                 │  • Error Handling (SMC)     │
                 │  • Add Correlation ID       │
                 └────────┬────────────────────┘
                          │
                          │ 6. JSON Response
                          ▼
                 ┌─────────────────┐
                 │  Mobile App     │
                 │  (Display Data) │
                 └─────────────────┘
```

### 4.1.2 Functional Specification Document

|  |  |
| --- | --- |
| **Title** | **Link to the FSD** |
| **IBA Loyalty App API - Azure APIM to Salesforce Integration** | EAII-472 - APIM to Salesforce API Technical Mapping (v2.4) |

## 4.2 Technical Standards and Requirements

**Application of industry standards:**

- **OpenAPI 3.0.1** for REST API contract specification
- **JSON** for request/response payloads
- **REST** over HTTPS for communication protocol
- **OAuth 2.1 Authorization Code + PKCE** for mobile app authentication
- **JWT (RFC 7519)** for token-based authentication
- **JWKS (RFC 7517)** for JWT signature verification

**Security Best Practices:**

- **OAuth 2.1** for API authentication and authorization
- **Azure Key Vault** for secrets management
- **HTTPS with TLS 1.2+** for encrypted communication
- **WAF (OWASP ModSecurity CRS)** for request validation
- **PII-safe logging** (exclude passwords, tokens from diagnostic logs)

## 4.3 Required Components

The following components are required for this integration:

- **Salesforce Experience Cloud** (Spring '26, API v66.0)
- **Azure API Management** (Premium or Standard tier)
- **Azure Key Vault** (for Integration User credentials)
- **Azure Application Insights** (for diagnostics and telemetry)
- **SMC (Solution Monitoring & Control)** (for error logging and alerting)
- **Azure Virtual Network** (for secure APIM backend communication)
- **Mobile Applications** (iOS/Android for TBO, Cellarbrations, Porters)

## 4.4 Environmental Requirement

The following environments will be used for this interface:

| Environment | APIM Instance | Salesforce Instance | Purpose |
|-------------|---------------|---------------------|---------|
| **DEV** | t3-met-core-dev-ae1-apim | metcash--dev.sandbox.my.salesforce.com | Development and unit testing |
| **SIT** | t3-met-core-sit-ae1-apim | metcash--sit.sandbox.my.salesforce.com | System integration testing |
| **UAT** | t3-met-core-uat-ae1-apim | metcash--uat.sandbox.my.salesforce.com | User acceptance testing |
| **PREPROD** | t3-met-core-preprod-ae1-apim | metcash--preprod.sandbox.my.salesforce.com | Pre-production validation |
| **PROD** | t3-met-core-prod-ae1-apim | metcash.my.salesforce.com | Production |

The above aligns with the **[AIS Environments Strategy](https://metcash.atlassian.net/wiki/spaces/MAP/pages/1637548092)**.

---

# 5 Integration Overview and Implementation

## 5.1 EAII-472 - IBA Loyalty App API

### 5.1.1 As-is Process(es)

#### 5.1.1.1 Description of Process(es)

**Legacy Mobile App Implementation:**

Prior to this integration, the legacy mobile applications (TBO, Cellarbrations, Porters) communicated **directly** with Salesforce Experience Cloud without an API gateway layer. This approach had several limitations:

- **No Centralized Security:** Each mobile app managed OAuth tokens independently without enterprise-level JWT validation or IDOR protection.
- **No Rate Limiting:** No throttling mechanism to prevent excessive API usage or protect against DDoS attacks.
- **Limited Observability:** No centralized logging or monitoring; troubleshooting required Salesforce debug logs.
- **No API Versioning:** Direct Salesforce API coupling made versioning and backward compatibility challenging.
- **Brand-Specific Logic in Apps:** Per-brand configurations (loyalty program IDs, field mappings) embedded in mobile app codebases.

### 5.1.2 Proposed Process(es)

#### 5.1.2.1 Description of Process(es)

**Overview:**  
The IBA Loyalty App API introduces **Azure API Management (APIM)** as an API facade between mobile applications and Salesforce. APIM provides enterprise-grade security, observability, rate limiting, and abstraction of backend complexity.

**Key Architectural Changes:**

1. **JWT Validation at Gateway:** APIM validates JWT signatures using Salesforce JWKS endpoints before forwarding requests.

2. **Server-Side Authorization:** APIM constructs SOQL queries server-side using authenticated user IDs from JWT claims, eliminating IDOR vulnerabilities.

3. **Integration User Pattern:** APIM uses a dedicated Salesforce Integration User account for backend API calls, with token caching (15-minute TTL) and auto-refresh.

4. **Per-Brand Mapping:** APIM maps JWT `client_id` claims to loyalty program IDs (TBO/Cellarbrations/Porters) server-side.

5. **Rate Limiting:** Token bucket algorithm enforces per-user (50 calls/60s) and per-IP (100 calls/60s) limits.

6. **SMC Integration:** Standardized error logging and alerting via SMC policy fragment.

7. **API Contract Abstraction:** Mobile apps consume a stable OpenAPI 3.0.1 contract independent of Salesforce schema changes.

#### 5.1.2.2 Process Flow Diagram

```
┌─────────────────────────────────────────────────────────────────────┐
│                        API REQUEST FLOW                             │
└─────────────────────────────────────────────────────────────────────┘

Step 1: Mobile App Authentication (Direct to Salesforce)
┌──────────────┐       OAuth 2.1 + PKCE      ┌──────────────────────┐
│  Mobile App  │ ───────────────────────────>│  Salesforce          │
│  (Client)    │ <───────────────────────────│  Headless Identity   │
└──────────────┘       JWT Token             └──────────────────────┘

Step 2: API Request with JWT
┌──────────────┐      GET /account           ┌──────────────────────┐
│  Mobile App  │      Authorization: Bearer  │  Azure APIM          │
│              │ ───────────────────────────>│  (Gateway)           │
└──────────────┘      {user_jwt}             └──────────────────────┘

Step 3: APIM Inbound Processing
┌──────────────────────────────────────────────────────────────────────┐
│  APIM Inbound Policy                                                 │
├──────────────────────────────────────────────────────────────────────┤
│  1. Extract JWT from Authorization header                            │
│  2. Validate JWT signature via Salesforce JWKS endpoint              │
│  3. Verify JWT claims (iss, aud, exp, sub, client_id)                │
│  4. Extract user-id from JWT sub claim                               │
│  5. Map client_id → loyalty-program-id (TBO/CBN/Porters)             │
│  6. Check rate limit (50 calls/60s per user)                         │
│  7. Retrieve cached Integration User token (or refresh if expired)   │
│  8. Construct SOQL query with semi-join pattern (IDOR prevention)    │
│  9. Rewrite request URI to Salesforce REST API endpoint              │
│ 10. Add Integration User token to Authorization header               │
└──────────────────────────────────────────────────────────────────────┘
                                │
                                ▼
Step 4: Salesforce Backend Call
┌──────────────────────┐      SOQL Query     ┌──────────────────────┐
│  APIM Backend        │ ──────────────────> │  Salesforce REST API │
│  Service             │ <────────────────── │  /services/data/v66.0│
└──────────────────────┘      JSON Response  └──────────────────────┘

Step 5: APIM Outbound Processing
┌──────────────────────────────────────────────────────────────────────┐
│  APIM Outbound Policy                                                │
├──────────────────────────────────────────────────────────────────────┤
│  1. Check Salesforce response status code                            │
│  2. Transform Salesforce response to API contract format             │
│  3. Handle errors (SMC logging for 4xx/5xx)                          │
│  4. Add correlation-id header (x-correlation-id)                     │
│  5. Add standard security headers (X-Frame-Options, etc.)            │
│  6. Return response to mobile app                                    │
└──────────────────────────────────────────────────────────────────────┘
                                │
                                ▼
Step 6: Response to Mobile App
┌──────────────┐      JSON Response          ┌──────────────────────┐
│  Mobile App  │ <───────────────────────────│  APIM Gateway        │
│              │      200 OK / 4xx / 5xx     │                      │
└──────────────┘                             └──────────────────────┘
```

#### 5.1.2.3 Sequence Diagram

```
Mobile App          APIM Gateway          Salesforce JWKS    Salesforce REST API      SMC
    │                   │                       │                    │                 │
    │ 1. GET /account   │                       │                    │                 │
    │ Authorization:    │                       │                    │                 │
    │ Bearer {jwt}      │                       │                    │                 │
    ├──────────────────>│                       │                    │                 │
    │                   │                       │                    │                 │
    │                   │ 2. Validate JWT       │                    │                 │
    │                   │ Signature             │                    │                 │
    │                   ├──────────────────────>│                    │                 │
    │                   │<─────────────────────┤                    │                 │
    │                   │ 3. JWKS Public Keys   │                    │                 │
    │                   │                       │                    │                 │
    │                   │ 4. Extract user-id    │                    │                 │
    │                   │    from JWT sub claim │                    │                 │
    │                   │                       │                    │                 │
    │                   │ 5. Build SOQL Query   │                    │                 │
    │                   │    (Semi-Join Pattern)│                    │                 │
    │                   │                       │                    │                 │
    │                   │ 6. GET /query?q={SOQL}                    │                 │
    │                   │    Authorization:                          │                 │
    │                   │    Bearer {int_token}                      │                 │
    │                   ├───────────────────────────────────────────>│                 │
    │                   │                       │                    │                 │
    │                   │                       │ 7. Execute SOQL    │                 │
    │                   │                       │    Query           │                 │
    │                   │                       │                    │                 │
    │                   │<───────────────────────────────────────────┤                 │
    │                   │ 8. {"records": [...]} │                    │                 │
    │                   │                       │                    │                 │
    │                   │ 9. Transform Response │                    │                 │
    │                   │                       │                    │                 │
    │<──────────────────┤                       │                    │                 │
    │ 10. 200 OK        │                       │                    │                 │
    │ {account_data}    │                       │                    │                 │
    │                   │                       │                    │                 │
    │                   │                       │                    │                 │
    │                   │       Error Scenario: Salesforce 500       │                 │
    │                   │<───────────────────────────────────────────┤                 │
    │                   │ 11. 500 Internal Error                     │                 │
    │                   │                       │                    │                 │
    │                   │ 12. Log Error to SMC  │                    │                 │
    │                   ├────────────────────────────────────────────────────────────>│
    │                   │                       │                    │                 │
    │<──────────────────┤                       │                    │                 │
    │ 13. 500 Internal  │                       │                    │                 │
    │ Server Error      │                       │                    │                 │
    │ x-correlation-id  │                       │                    │                 │
```

#### 5.1.2.4 API Endpoints Summary

**Total: 23 Endpoints (20 Authenticated + 3 Unauthenticated)**

| Category | Method | Endpoint | Auth Required | Description |
|----------|--------|----------|---------------|-------------|
| **Account Operations** | | | | |
| | GET | `/account` | ✓ | Retrieve authenticated user's account details |
| | PATCH | `/account` | ✓ | Update account fields (name, phone, postal code, home store) |
| | PATCH | `/account/deactivate` | ✓ | Deactivate user login (set `User.IsActive = false`) |
| | PATCH | `/account/change-password` | ✓ | Change password via Integration User |
| | GET | `/account/notifications` | ✓ | Retrieve push/email notification preferences |
| | PATCH | `/account/notifications` | ✓ | Update notification settings |
| | GET | `/account/validate` | ✓ | Validate membership details (server-side JWT derivation) |
| **Loyalty Operations** | | | | |
| | GET | `/loyalty/points` | ✓ | Points transaction history (from `TransactionJournal`) |
| | GET | `/loyalty/points/bonus` | ✓ | Bonus points transactions only |
| | GET | `/loyalty/points-to-cash` | ✓ | Points-to-cash redemption information |
| | GET | `/loyalty/program` | ✓ | Loyalty program details and tier information |
| | POST | `/loyalty/enrol` | ✓ | Cross-program enrollment (idempotent) |
| **Store Operations** | | | | |
| | GET | `/stores/all` | ✗ | All stores (unauthenticated, IP rate-limited) |
| | GET | `/stores/nearby` | ✗ | Nearby stores by geo location (unauthenticated, IP rate-limited) |
| | GET | `/stores/home` | ✓ | User's home store (derived from Account via semi-join) |
| | GET | `/stores/related` | ✓ | Related stores to user's home store (MSO hierarchy) |
| **Product Operations** | | | | |
| | GET | `/categories` | ✓ | All product categories |
| | GET | `/banners-by-category` | ✓ | Marketing banners filtered by category |
| | GET | `/products/preferences/selected` | ✓ | User's selected product preferences (per-brand fields) |
| | GET | `/products/promotions` | ✓ | Promotions with `PricebookEntry` details |
| **Registration** | | | | |
| | GET | `/registration/check-email` | ✗ | Check email availability (unauthenticated, email format validation) |
| **Search** | | | | |
| | GET | `/search/products` | ✓ | Product search by keyword |
| | GET | `/search/stores` | ✓ | Store search by name or location |

#### 5.1.2.5 Interface Requirements / Mapping Details

**Example: GET /account**

**Request:**
```http
GET https://api-dev.metcash.com/iba-loyalty/v1/account
Authorization: Bearer {user_jwt}
Accept: application/json
```

**APIM Processing:**
1. Validate JWT signature via Salesforce JWKS
2. Extract `user-id` from JWT `sub` claim (e.g., `005U900000P1ajeIAB`)
3. Retrieve cached Integration User token
4. Construct SOQL query:
   ```sql
   SELECT Id, FirstName, LastName, PersonEmail, PersonBirthdate,
          PersonMailingPostalCode, Phone, CBN_Home_Store__pc,
          Loyalty_Program_Member_TBO__pc, Loyalty_Program_Member_CBN__pc
   FROM Account
   WHERE PersonContactId IN (SELECT ContactId FROM User WHERE Id = '005U900000P1ajeIAB')
   ```
5. Rewrite URI to Salesforce: `/services/data/v66.0/query?q={encoded_soql}`
6. Forward request with Integration User token

**Salesforce Response:**
```json
{
  "records": [
    {
      "Id": "001U900000ABCDEFGH",
      "FirstName": "John",
      "LastName": "Smith",
      "PersonEmail": "john.smith@example.com",
      "PersonBirthdate": "1985-06-15",
      "PersonMailingPostalCode": "3000",
      "Phone": "0498765432",
      "CBN_Home_Store__pc": "a07U9000000XYZ123",
      "Loyalty_Program_Member_TBO__pc": "0lpU9000000TBO123",
      "Loyalty_Program_Member_CBN__pc": "0lpU9000000CBN456"
    }
  ]
}
```

**APIM Transformation:**
```json
{
  "Id": "001U900000ABCDEFGH",
  "FirstName": "John",
  "LastName": "Smith",
  "PersonEmail": "john.smith@example.com",
  "PersonBirthdate": "1985-06-15",
  "PersonMailingPostalCode": "3000",
  "Phone": "0498765432",
  "CBN_Home_Store__pc": "a07U9000000XYZ123",
  "Loyalty_Program_Member_TBO__pc": "0lpU9000000TBO123",
  "Loyalty_Program_Member_CBN__pc": "0lpU9000000CBN456"
}
```

**Response to Mobile App:**
```http
HTTP/1.1 200 OK
Content-Type: application/json
x-correlation-id: 123e4567-e89b-12d3-a456-426614174000

{account_data}
```

#### 5.1.2.6 Authentication and Authorization

**Authentication Flow:**

```
┌──────────────────────────────────────────────────────────────────────┐
│  Step 1: Mobile App Obtains JWT from Salesforce (Direct)            │
└──────────────────────────────────────────────────────────────────────┘

Mobile App → Salesforce Headless Identity API
  POST /services/auth/headless/init/registration (self-registration)
  POST /services/oauth2/authorize (OAuth 2.1 + PKCE)
  POST /services/oauth2/token (code exchange)
  
  Response: JWT Token
  {
    "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCIsImtpZCI6IjIwNiJ9...",
    "refresh_token": "...",
    "token_type": "Bearer",
    "expires_in": 7200
  }

┌──────────────────────────────────────────────────────────────────────┐
│  Step 2: Mobile App Sends JWT to APIM                               │
└──────────────────────────────────────────────────────────────────────┘

Mobile App → APIM
  Authorization: Bearer {user_jwt}

┌──────────────────────────────────────────────────────────────────────┐
│  Step 3: APIM Validates JWT                                          │
└──────────────────────────────────────────────────────────────────────┘

APIM → Salesforce JWKS Endpoint
  GET https://metcash--dev.sandbox.my.salesforce.com/.well-known/openid-configuration
  GET https://metcash--dev.sandbox.my.salesforce.com/id/keys
  
  Validates:
  - Signature (RS256 algorithm with public key from JWKS)
  - Issuer (iss claim matches Salesforce instance URL)
  - Audience (aud claim matches loyalty URL)
  - Expiration (exp claim > current time)
  - Client ID (client_id claim in allowed list: TBO/CBN/Porters)
  - Subject (sub claim exists)

┌──────────────────────────────────────────────────────────────────────┐
│  Step 4: APIM Uses Integration User Token for Backend Calls         │
└──────────────────────────────────────────────────────────────────────┘

APIM → Salesforce OAuth Token Endpoint
  POST /services/oauth2/token
  grant_type=refresh_token
  refresh_token={integration_user_refresh_token}
  client_id={integration_user_client_id}
  client_secret={integration_user_client_secret}
  
  Response: Integration User Token (cached for 15 minutes)
  
APIM → Salesforce REST API
  Authorization: Bearer {integration_user_token}
```

**JWT Claims Used:**

| JWT Claim | Purpose | Example Value |
|-----------|---------|---------------|
| `sub` | User ID (for SOQL queries) | `https://test.salesforce.com/id/00Dxx0000001gEREAY/005xx000001SwiZAAS` |
| `iss` | Issuer validation | `https://metcash.my.salesforce.com` |
| `aud` | Audience validation | `https://metcash.my.site.com/loyalty` |
| `exp` | Token expiration | `1704067200` |
| `client_id` | Brand identification | `3!1VG9...` (TBO), `3!1VG9...` (CBN), `3!1VG9...` (Porters) |

**Authorization Decision:**

- **Authenticated Endpoints (20):** Require valid JWT; user accesses only their own data (IDOR prevention via semi-join)
- **Unauthenticated Endpoints (3):** No JWT required; IP-based rate limiting (100 calls/60s)

#### 5.1.2.7 Non-functional Requirements

| NFR Category | Requirement | Target Value |
|--------------|-------------|--------------|
| **Performance** | API response time (95th percentile) | < 500ms |
| | API response time (99th percentile) | < 1000ms |
| | Maximum timeout | 30 seconds |
| | Throughput (per user) | 50 requests/minute |
| **Availability** | API uptime SLA | 99.9% (dependent on Salesforce) |
| | Planned maintenance window | Aligned with Salesforce maintenance |
| **Scalability** | Concurrent users | 10,000+ |
| | Requests per second (peak) | 1,000 RPS |
| | Token cache size | 10,000 cached Integration User tokens |
| **Security** | Encryption in transit | TLS 1.2+ (HTTPS only) |
| | Encryption at rest (Key Vault) | AES-256 |
| | Token expiration | JWT: 2 hours, Integration token: 15 minutes (cache TTL) |
| | Rate limiting (authenticated) | 50 calls/60 seconds per user |
| | Rate limiting (unauthenticated) | 100 calls/60 seconds per IP |
| **Monitoring** | Log retention (Application Insights) | 90 days |
| | SMC alert response time | < 5 minutes |
| | Correlation ID tracking | 100% of requests |
| **Data Integrity** | SOQL query validation | 100% (no client-provided IDs) |
| | Response format validation | OpenAPI 3.0.1 schema validation |

### 5.1.3 Coordination

#### 5.1.3.1 Transaction Boundaries

**Request-Level Transactions:**

Each API request is a **single logical transaction** with the following boundaries:

1. **Inbound Boundary:** Mobile app sends HTTP request to APIM
2. **Processing Boundary:** APIM validates JWT, constructs SOQL, calls Salesforce
3. **Backend Boundary:** Salesforce executes SOQL query (read or update operation)
4. **Outbound Boundary:** APIM transforms response and returns to mobile app

**Transaction Characteristics:**

- **Atomicity:** Salesforce DML operations are atomic (e.g., PATCH /account updates all fields or none)
- **Idempotency:** POST /loyalty/enrol is idempotent (repeated enrollment requests do not create duplicates)
- **Timeout:** 30-second timeout for entire request lifecycle
- **Retry Policy:** Mobile apps should implement exponential backoff for transient 5xx errors

**No Distributed Transactions:**

- APIM does not persist state (stateless gateway pattern)
- No two-phase commit or saga patterns required
- Salesforce is the single source of truth for loyalty data

#### 5.1.3.2 State Management

**Stateless Gateway Pattern:**

APIM does not store user session state or loyalty data. All state resides in:

1. **Mobile App:** JWT token (cached until expiration)
2. **Salesforce:** Loyalty data (Account, Member, Points, etc.)
3. **APIM Cache:** Integration User tokens only (15-minute TTL, auto-refresh)

**Token Management:**

- **User JWT:** Issued and managed by Salesforce (2-hour expiration); mobile app responsible for refresh
- **Integration User Token:** Cached in APIM with 15-minute TTL; APIM auto-refreshes on cache miss or expiration

**No Session Affinity:**

- API requests can be served by any APIM gateway node (horizontal scaling)
- No sticky sessions or session replication required

### 5.1.4 Design Considerations

**Key Design Decisions:**

1. **Integration User Pattern:**
   - **Decision:** Use dedicated Salesforce Integration User for backend calls instead of user impersonation.
   - **Rationale:** Simplifies Salesforce permission management; single service account vs. per-user credentials.
   - **Trade-off:** Salesforce audit logs show Integration User instead of end-user (mitigated by correlation ID tracking).

2. **Semi-Join SOQL Pattern:**
   - **Decision:** Derive Account ID server-side via `WHERE PersonContactId IN (SELECT ContactId FROM User WHERE Id = {user_id})`.
   - **Rationale:** Eliminates IDOR vulnerability; client cannot manipulate Account ID parameter.
   - **Trade-off:** Slightly more complex SOQL queries (two-step resolution for some endpoints).

3. **JWT Validation at Gateway:**
   - **Decision:** APIM validates JWT signatures using Salesforce JWKS endpoints.
   - **Rationale:** Centralized authentication enforcement; mobile apps cannot bypass validation.
   - **Trade-off:** Dependency on Salesforce JWKS endpoint availability (mitigated by caching public keys).

4. **Per-Brand Client ID Mapping:**
   - **Decision:** Map JWT `client_id` claim to loyalty program IDs server-side.
   - **Rationale:** Mobile apps don't need to know loyalty program IDs; APIM handles brand-specific logic.
   - **Trade-off:** APIM configuration update required for new brands.

5. **Unauthenticated Store Endpoints:**
   - **Decision:** Allow `/stores/all` and `/stores/nearby` without authentication.
   - **Rationale:** Store listings are public data; removing auth friction improves UX.
   - **Mitigation:** IP-based rate limiting (100 calls/60s) to prevent scraping.

6. **SMC Error Logging:**
   - **Decision:** Log Salesforce 4xx/5xx errors to SMC for alerting.
   - **Rationale:** Proactive error detection and resolution; reduces MTTR (Mean Time To Resolution).
   - **Trade-off:** Additional API call overhead (<50ms per error).

### 5.1.5 Service High Level Design

#### 5.1.5.1 Component List

| Component | Technology | Purpose |
|-----------|------------|---------|
| **Azure API Management** | Azure APIM Premium | API gateway, policy enforcement, JWT validation, rate limiting |
| **Salesforce Experience Cloud** | Salesforce Spring '26 (v66.0) | Loyalty data storage, REST API, OAuth authentication |
| **Azure Key Vault** | Azure Key Vault | Secrets management (Integration User credentials) |
| **Azure Application Insights** | Azure Monitor | Diagnostics, telemetry, performance metrics |
| **SMC (Solution Monitoring & Control)** | Custom APIM Policy | Error logging, alerting, operational monitoring |
| **Mobile Applications** | iOS/Android | End-user interface (TBO, Cellarbrations, Porters apps) |

#### 5.1.5.2 Services

**Service 1: Mobile App → APIM (API Request)**
- **Protocol:** HTTPS (TLS 1.2+)
- **Authentication:** Bearer JWT in Authorization header
- **Data Format:** JSON (request bodies for PATCH operations)
- **Rate Limiting:** 50 calls/60 seconds per user (authenticated), 100 calls/60 seconds per IP (unauthenticated)

**Service 2: APIM → Salesforce JWKS (JWT Validation)**
- **Protocol:** HTTPS
- **Endpoint:** `https://metcash.my.salesforce.com/.well-known/openid-configuration`
- **Caching:** Public keys cached in APIM for 24 hours
- **Purpose:** Retrieve public keys for JWT signature verification

**Service 3: APIM → Salesforce REST API (Data Queries)**
- **Protocol:** HTTPS
- **Authentication:** Bearer token (Integration User)
- **Base URL:** `https://metcash.my.salesforce.com/services/data/v66.0/`
- **Operations:** GET (SOQL queries), PATCH (DML updates), POST (enrollment)

**Service 4: APIM → SMC (Error Logging)**
- **Protocol:** HTTPS
- **Endpoint:** `https://t3-met-core-dev-ae1-apim.azure-api.net/monitoring/smc/v1/*`
- **Operations:** `startObservation`, `observationPoint`, `completeObservation`
- **Purpose:** Log errors, warnings, and completion events for operational monitoring

#### 5.1.5.3 APIM Policy Framework

**API-Level Policies (Gateway Layer):**

```xml
<policies>
  <inbound>
    <!-- 1. Conditional JWT Validation (skip for 3 unauthenticated endpoints) -->
    <choose>
      <when condition="@(context.Request.Url.Path.Contains("/stores/all") || 
                          context.Request.Url.Path.Contains("/stores/nearby") ||
                          context.Request.Url.Path.Contains("/registration/check-email"))">
        <!-- Unauthenticated endpoint: Skip JWT validation -->
        <rate-limit-by-key calls="100" renewal-period="60" 
                           counter-key="@(context.Request.IpAddress)" />
      </when>
      <otherwise>
        <!-- Authenticated endpoint: Validate JWT -->
        <validate-jwt header-name="Authorization" 
                      failed-validation-httpcode="401"
                      failed-validation-error-message="Invalid or expired JWT token">
          <openid-config url="{{salesforce-ibaloyalty-loyaltyurl}}/.well-known/openid-configuration" />
          <audiences>
            <audience>{{salesforce-ibaloyalty-loyaltyurl}}</audience>
          </audiences>
          <issuers>
            <issuer>{{salesforce-ibaloyalty-loyaltyurl}}</issuer>
          </issuers>
          <required-claims>
            <claim name="sub" />
            <claim name="client_id" match="any">
              <value>{{salesforce-ibaloyalty-porters-clientid}}</value>
              <value>{{salesforce-ibaloyalty-tbo-clientid}}</value>
              <value>{{salesforce-ibaloyalty-celebrations-clientid}}</value>
            </claim>
          </required-claims>
        </validate-jwt>
        
        <!-- Extract user-id from JWT sub claim -->
        <set-variable name="user-id" value="@{
          string token = context.Request.Headers.GetValueOrDefault("Authorization","").Split(' ')[1];
          var jwt = token.AsJwt();
          var sub = jwt?.Subject;
          if (!string.IsNullOrEmpty(sub) && sub.Contains("/")) {
            return sub.Split('/').Last();
          }
          return sub;
        }" />
        
        <!-- Map client_id to loyalty-program-id -->
        <set-variable name="loyalty-program-id" value="@{
          string token = context.Request.Headers.GetValueOrDefault("Authorization","").Split(' ')[1];
          var jwt = token.AsJwt();
          var clientId = jwt?.Claims.GetValueOrDefault("client_id", "");
          
          if (clientId == "{{salesforce-ibaloyalty-tbo-clientid}}") {
            return "{{salesforce-ibaloyalty-tbo-loyaltyprogramid}}";
          } else if (clientId == "{{salesforce-ibaloyalty-celebrations-clientid}}") {
            return "{{salesforce-ibaloyalty-celebrations-loyaltyprogramid}}";
          } else if (clientId == "{{salesforce-ibaloyalty-porters-clientid}}") {
            return "{{salesforce-ibaloyalty-porters-loyaltyprogramid}}";
          }
          return "";
        }" />
        
        <!-- Rate limiting (per-user) -->
        <rate-limit-by-key calls="50" renewal-period="60" 
                           counter-key="@((string)context.Variables["user-id"])" />
      </otherwise>
    </choose>
    
    <!-- 2. Retrieve Integration User token (cached or refresh) -->
    <cache-lookup-value key="sf-integration-token" variable-name="integration-token" />
    <choose>
      <when condition="@(!context.Variables.ContainsKey("integration-token"))">
        <!-- Token not in cache: Refresh -->
        <send-request mode="new" response-variable-name="tokenResponse" timeout="10" ignore-error="false">
          <set-url>{{salesforce-ibaloyalty-baseurl}}/services/oauth2/token</set-url>
          <set-method>POST</set-method>
          <set-header name="Content-Type" exists-action="override">
            <value>application/x-www-form-urlencoded</value>
          </set-header>
          <set-body>@{
            return "grant_type=refresh_token" +
                   "&refresh_token={{salesforce-ibaloyalty-integrationuser-refreshtoken}}" +
                   "&client_id={{salesforce-ibaloyalty-integrationuser-clientid}}" +
                   "&client_secret={{salesforce-ibaloyalty-integrationuser-clientsecret}}";
          }</set-body>
        </send-request>
        
        <!-- Extract access_token from response -->
        <set-variable name="integration-token" value="@{
          var response = ((IResponse)context.Variables["tokenResponse"]).Body.As<JObject>();
          return response["access_token"].ToString();
        }" />
        
        <!-- Cache token for 15 minutes -->
        <cache-store-value key="sf-integration-token" 
                           value="@((string)context.Variables["integration-token"])" 
                           duration="900" />
      </when>
    </choose>
    
    <!-- 3. Add correlation-id header -->
    <set-header name="x-correlation-id" exists-action="skip">
      <value>@(Guid.NewGuid().ToString())</value>
    </set-header>
  </inbound>
  
  <outbound>
    <!-- Transform response and add security headers -->
    <set-header name="X-Frame-Options" exists-action="override">
      <value>DENY</value>
    </set-header>
    <set-header name="X-Content-Type-Options" exists-action="override">
      <value>nosniff</value>
    </set-header>
    <set-header name="x-correlation-id" exists-action="override">
      <value>@(context.Request.Headers.GetValueOrDefault("x-correlation-id",""))</value>
    </set-header>
  </outbound>
  
  <on-error>
    <!-- SMC Error Logging -->
    <include-fragment fragment-id="salesforce-ibaloyalty-smc-logger_v1" />
  </on-error>
</policies>
```

**Operation-Level Policies (Endpoint-Specific):**

Example: GET /account

```xml
<policies>
  <inbound>
    <!-- Build SOQL query with semi-join pattern -->
    <set-variable name="soql-query" value="@($"SELECT Id, FirstName, LastName, PersonEmail, PersonBirthdate, PersonMailingPostalCode, Phone, CBN_Home_Store__pc, Loyalty_Program_Member_TBO__pc, Loyalty_Program_Member_CBN__pc FROM Account WHERE PersonContactId IN (SELECT ContactId FROM User WHERE Id = '{context.Variables["user-id"]}')")" />
    
    <!-- Rewrite request to Salesforce -->
    <set-backend-service base-url="{{salesforce-ibaloyalty-baseurl}}/services/data/v66.0" />
    <rewrite-uri template="@("/query?q=" + System.Net.WebUtility.UrlEncode((string)context.Variables["soql-query"]))" />
    <set-header name="Authorization" exists-action="override">
      <value>@("Bearer " + context.Variables["integration-token"])</value>
    </set-header>
  </inbound>
  
  <outbound>
    <!-- Transform Salesforce response -->
    <set-body>@{
      var sfResponse = context.Response.Body.As<JObject>();
      var records = sfResponse["records"] as JArray;
      if (records != null && records.Count > 0) {
        return records[0].ToString();
      }
      return "{}";
    }</set-body>
  </outbound>
</policies>
```

#### 5.1.5.4 Logging and Monitoring

**SMC Policy Fragment (salesforce-ibaloyalty-smc-logger_v1):**

```xml
<fragment>
  <choose>
    <when condition="@(context.Response.StatusCode >= 400)">
      <!-- Log error to SMC -->
      <send-request mode="new" response-variable-name="smcResponse" timeout="10" ignore-error="true">
        <set-url>https://t3-met-core-dev-ae1-apim.azure-api.net/monitoring/smc/v1/observationPoint</set-url>
        <set-method>POST</set-method>
        <set-header name="Content-Type" exists-action="override">
          <value>application/json</value>
        </set-header>
        <set-body>@{
          return new JObject(
            new JProperty("correlationId", context.Request.Headers.GetValueOrDefault("x-correlation-id","")),
            new JProperty("statusCode", context.Response.StatusCode),
            new JProperty("errorMessage", context.Response.Body.As<string>(preserveContent: true)),
            new JProperty("endpoint", context.Request.Url.Path),
            new JProperty("userId", context.Variables.ContainsKey("user-id") ? context.Variables["user-id"] : "unauthenticated")
          ).ToString();
        }</set-body>
      </send-request>
    </when>
  </choose>
</fragment>
```

**Application Insights Diagnostics:**

- **Request telemetry:** HTTP method, URL, status code, duration
- **Dependency telemetry:** Salesforce API calls, JWKS lookups, SMC requests
- **Exception telemetry:** Unhandled errors (policy execution failures)
- **Custom events:** JWT validation failures, rate limit exceeded, cache misses

**PII-Safe Logging:**

- Password fields excluded from diagnostic logs (override at operation level)
- JWT tokens not logged in plaintext (correlation ID used for tracing)
- Email addresses hashed in logs (one-way hash for privacy)

#### 5.1.5.5 Service Operations Detailed Mapping

**Account Operations (7 endpoints):**

1. **GET /account** - Retrieve account details
   - **SOQL:** `SELECT ... FROM Account WHERE PersonContactId IN (SELECT ContactId FROM User WHERE Id = {user_id})`
   - **Response:** Single Account object
   - **Rate Limit:** 50/60s per user

2. **PATCH /account** - Update account fields
   - **Step 1:** Resolve Account ID (same semi-join as GET /account)
   - **Step 2:** `PATCH /services/data/v66.0/sobjects/Account/{account_id}`
   - **Request Body:** `{"FirstName": "...", "LastName": "...", "Phone": "...", "PostalCode": "...", "CBN_Home_Store__pc": "..."}`
   - **Response:** 204 No Content (success) or 400 Bad Request (validation error)

3. **PATCH /account/deactivate** - Deactivate user login
   - **Step 1:** Resolve User ID from JWT sub claim
   - **Step 2:** `PATCH /services/data/v66.0/sobjects/User/{user_id}` with `{"IsActive": false}`
   - **Response:** 204 No Content

4. **PATCH /account/change-password** - Change password
   - **Step 1:** Validate new password (min 8 chars, complexity rules)
   - **Step 2:** `POST /services/data/v66.0/sobjects/User/{user_id}/password` with `{"newPassword": "..."}`
   - **Security:** WAF exclusion for password field (rules 942430-942432), diagnostic logging override
   - **Response:** 204 No Content

5. **GET /account/notifications** - Retrieve notification preferences
   - **SOQL:** `SELECT Email_Notifications__pc, Push_Notifications__pc FROM Account WHERE PersonContactId IN (SELECT ContactId FROM User WHERE Id = {user_id})`
   - **Response:** `{"emailNotifications": true, "pushNotifications": false}`

6. **PATCH /account/notifications** - Update notification preferences
   - **Step 1:** Resolve Account ID
   - **Step 2:** `PATCH /services/data/v66.0/sobjects/Account/{account_id}` with `{"Email_Notifications__pc": true, "Push_Notifications__pc": false}`
   - **Response:** 204 No Content

7. **GET /account/validate** - Validate membership
   - **SOQL:** `SELECT Loyalty_Program_Member_TBO__pc, Loyalty_Program_Member_CBN__pc FROM Account WHERE PersonContactId IN (SELECT ContactId FROM User WHERE Id = {user_id})`
   - **Response:** `{"valid": true, "membershipId": "0lpU9000000TBO123"}`

**Loyalty Operations (5 endpoints):**

1. **GET /loyalty/points** - Transaction history
   - **SOQL:** `SELECT Id, ActivityDate, Points_Mobile__c, Points_Info_Mobile__c FROM TransactionJournal WHERE MemberId IN (SELECT Id FROM LoyaltyProgramMember WHERE ContactId IN (SELECT ContactId FROM User WHERE Id = {user_id}) AND ProgramId = {loyalty_program_id}) AND Points_Mobile__c != null ORDER BY ActivityDate DESC LIMIT {limit}`
   - **Query Parameters:** `limit` (default: 50, max: 100)
   - **Response:** Array of transaction objects

2. **GET /loyalty/points/bonus** - Bonus points only
   - **SOQL:** Same as /loyalty/points with additional filter `AND TransactionType = 'BONUS'`
   - **Response:** Array of bonus transaction objects

3. **GET /loyalty/points-to-cash** - Redemption information
   - **SOQL:** `SELECT Id, LoyaltyPoints, VoucherAmount FROM LoyaltyMemberCurrency WHERE MemberId IN (SELECT Id FROM LoyaltyProgramMember WHERE ContactId IN (SELECT ContactId FROM User WHERE Id = {user_id}) AND ProgramId = {loyalty_program_id}) AND OSF_Is_Redeemable__Points__c = true`
   - **Response:** `{"pointsBalance": 5000, "cashValue": 50.00, "redemptionRate": 0.01}`

4. **GET /loyalty/program** - Program details
   - **SOQL:** `SELECT Id, Name, TierGroup.Id, TierGroup.Name, TierGroup.IsActive FROM LoyaltyTierGroup WHERE ProgramId = {loyalty_program_id} AND IsActive = true`
   - **Response:** Loyalty program object with tier information

5. **POST /loyalty/enrol** - Cross-program enrollment
   - **Idempotent:** Checks for existing enrollment before creating
   - **Step 1:** Check: `SELECT Id FROM LoyaltyProgramMember WHERE ContactId IN (SELECT ContactId FROM User WHERE Id = {user_id}) AND ProgramId = {target_loyalty_program_id}`
   - **Step 2 (if not exists):** `POST /services/data/v66.0/sobjects/LoyaltyProgramMember` with `{"ContactId": "...", "ProgramId": "...", "EnrollmentDate": "..."}`
   - **Response:** 201 Created (new enrollment) or 200 OK (already enrolled)

**Store Operations (4 endpoints):**

1. **GET /stores/all** - All stores (UNAUTHENTICATED)
   - **SOQL:** `SELECT Id, Name, ShippingAddress, Geo_Location__c, BSO__c, Is_Loyalty_Store__c FROM Account WHERE RecordType.Name = 'Loyalty Store' ORDER BY Name`
   - **Rate Limit:** 100/60s per IP
   - **Response:** Array of store objects

2. **GET /stores/nearby** - Nearby stores (UNAUTHENTICATED)
   - **SOQL:** `SELECT Id, Name, ShippingAddress, Geo_Location__c, DISTANCE(Geo_Location__c, GEOLOCATION({lat},{lng}), 'km') distance FROM Account WHERE RecordType.Name = 'Loyalty Store' AND DISTANCE(Geo_Location__c, GEOLOCATION({lat},{lng}), 'km') < {radius} ORDER BY distance LIMIT 50`
   - **Query Parameters:** `lat`, `lng`, `radius` (default: 10km, max: 100km)
   - **Rate Limit:** 100/60s per IP
   - **Response:** Array of store objects with distance

3. **GET /stores/home** - User's home store
   - **Step 1:** Resolve home store field from Account based on loyalty program: `SELECT CBN_Home_Store__pc FROM Account WHERE PersonContactId IN (SELECT ContactId FROM User WHERE Id = {user_id})`
   - **Step 2:** Fetch store details: `SELECT Id, Name, ShippingAddress, Geo_Location__c FROM Account WHERE Id = {home_store_id}`
   - **Response:** Single store object

4. **GET /stores/related** - Related stores (MSO hierarchy)
   - **SOQL:** `SELECT Id, Name, ShippingAddress FROM Account WHERE BSO__c = {home_store_bso} AND RecordType.Name = 'Loyalty Store'`
   - **Response:** Array of stores in same BSO (Member Service Organization)

**Product Operations (4 endpoints):**

1. **GET /categories** - All categories
   - **SOQL:** `SELECT Id, Product_Category__c.Id, Product_Category__c.Name FROM ProductCategory ORDER BY CreatedDate DESC`
   - **Response:** Array of category objects

2. **GET /banners-by-category** - Marketing banners
   - **SOQL:** `SELECT Id, Name, Image_URL__c, Main_Category__c FROM ProductCategoryProduct WHERE Main_Category__c = {category_id} ORDER BY CreatedDate DESC LIMIT 20`
   - **Query Parameters:** `categoryId` (required)
   - **Response:** Array of banner objects

3. **GET /products/preferences/selected** - User's product preferences
   - **Per-Brand Field Mapping:**
     - TBO: `{Category}_TBO__pc` (e.g., `Beer_TBO__pc`, `Wine_TBO__pc`)
     - Cellarbrations: `{Category}_CBN__pc`
     - Porters: `{Category}_POR__pc`
   - **SOQL:** `SELECT Beer_TBO__pc, Wine_TBO__pc, Spirits_TBO__pc FROM Account WHERE PersonContactId IN (SELECT ContactId FROM User WHERE Id = {user_id})`
   - **Response:** `{"preferences": ["Beer", "Wine", "Spirits"]}`

4. **GET /products/promotions** - Promotions with pricing
   - **SOQL:** `SELECT Id, Product2.Id, Product2.Name, Product2.Description, UnitPrice, Pricebook2.Id FROM PricebookEntry WHERE Pricebook2Id = {price_book_id} AND IsActive = true ORDER BY Product2.Name LIMIT 100`
   - **Query Parameters:** `price_book_id` (required), `product_ids` (optional, comma-separated, max 20)
   - **Response:** Array of promotion objects with product details

**Registration (1 endpoint):**

1. **GET /registration/check-email** - Email availability (UNAUTHENTICATED)
   - **SOQL:** `SELECT Id FROM User WHERE Username = '{email}' OR Email = '{email}'`
   - **Email Validation:** Regex pattern `^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$`
   - **Rate Limit:** 100/60s per IP
   - **Response:** `{"available": false}` (email exists) or `{"available": true}` (email free)
   - **Security Trade-off:** Email enumeration possible but mitigated by IP rate limiting

**Search (2 endpoints):**

1. **GET /search/products** - Product search
   - **SOQL:** `FIND {search_term} IN ALL FIELDS RETURNING Product2(Id, Name, Description) LIMIT 50`
   - **Query Parameters:** `q` (search term, min 3 characters)
   - **Response:** Array of product objects

2. **GET /search/stores** - Store search
   - **SOQL:** `FIND {search_term} IN NAME FIELDS RETURNING Account(Id, Name, ShippingAddress WHERE RecordType.Name = 'Loyalty Store') LIMIT 50`
   - **Query Parameters:** `q` (search term, min 3 characters)
   - **Response:** Array of store objects

### 5.1.6 SMC Onboarding

#### 5.1.6.1 Integration Onboarding

**SMC Registration Details:**

| Field | Value |
|-------|-------|
| **Integration Name** | IBA-LOYALTY-APP-API |
| **Integration ID** | EAII-472 |
| **Owner Team** | Integration Platform Team |
| **Business Owner** | Retail Operations - IBA Loyalty |
| **Technical Contact** | integration@metcash.com |
| **Criticality** | High (customer-facing mobile app dependency) |
| **SLA** | 99.9% availability |

**SMC Observation Points:**

1. **JWT Validation Failure** (401 Unauthorized)
   - **Trigger:** Invalid or expired JWT token
   - **Severity:** Warning (user action required: re-login)
   - **Alert:** No (expected user behavior)

2. **Rate Limit Exceeded** (429 Too Many Requests)
   - **Trigger:** User/IP exceeds rate limit (50/60s or 100/60s)
   - **Severity:** Warning
   - **Alert:** Yes (if sustained > 5 minutes, potential abuse)

3. **Salesforce 4xx Error** (400, 403, 404)
   - **Trigger:** Bad request, forbidden, or not found
   - **Severity:** Error
   - **Alert:** Yes (if error rate > 5% of total requests)

4. **Salesforce 5xx Error** (500, 502, 503, 504)
   - **Trigger:** Salesforce internal error or unavailability
   - **Severity:** Critical
   - **Alert:** Yes (immediate alert to on-call engineer)

5. **APIM Timeout** (504 Gateway Timeout)
   - **Trigger:** Salesforce response not received within 30 seconds
   - **Severity:** Critical
   - **Alert:** Yes (if frequency > 10 requests/minute)

#### 5.1.6.2 SMC Load Estimations

**Expected API Traffic:**

| Metric | Value | Calculation Basis |
|--------|-------|-------------------|
| **Total Registered Users** | 50,000 | Existing loyalty program members |
| **Active Users (Monthly)** | 25,000 | 50% engagement rate |
| **Active Users (Daily)** | 5,000 | 20% of monthly active users |
| **API Calls per User per Day** | 20 | Average app usage (view points, search stores, check promotions) |
| **Total API Calls per Day** | 100,000 | 5,000 users × 20 calls |
| **Peak API Calls per Second** | 50 RPS | Peak hour traffic (8-9 AM, 5-6 PM) |

**Error Rate Estimations:**

| Error Type | Expected Rate | SMC Logs per Day |
|------------|---------------|------------------|
| **JWT Validation Failure (401)** | 2% | 2,000 (users with expired tokens) |
| **Rate Limit Exceeded (429)** | 0.5% | 500 (power users) |
| **Salesforce 4xx Errors** | 1% | 1,000 (bad requests) |
| **Salesforce 5xx Errors** | 0.1% | 100 (expected Salesforce availability: 99.9%) |
| **APIM Timeouts (504)** | 0.05% | 50 (slow Salesforce queries) |

**SMC Storage Requirements:**

- **Log Entries per Day:** ~3,650 (errors only, not logging 200 OK responses)
- **Average Log Size:** 2 KB per entry
- **Daily Storage:** ~7 MB
- **Retention Period:** 90 days
- **Total Storage:** ~630 MB

### 5.1.7 Monitoring

**Application Insights Dashboards:**

1. **API Performance Dashboard:**
   - Request count (total, per endpoint)
   - Response time (p50, p95, p99)
   - Error rate (4xx, 5xx)
   - Throughput (requests per second)

2. **Salesforce Backend Dashboard:**
   - Salesforce dependency latency
   - Salesforce error rate
   - SOQL query complexity metrics
   - Integration User token cache hit rate

3. **Security Dashboard:**
   - JWT validation failures
   - Rate limit violations
   - Unauthorized access attempts (401, 403)
   - IP address patterns (anomaly detection)

**Alerts:**

| Alert Name | Condition | Threshold | Action |
|------------|-----------|-----------|--------|
| **High Error Rate** | 5xx errors | > 5% of total requests for 5 minutes | Page on-call engineer |
| **Salesforce Unavailable** | 503 errors | > 50% of requests for 2 minutes | Page on-call + escalate to Salesforce support |
| **Slow Response Time** | p95 latency | > 2 seconds for 10 minutes | Investigate Salesforce query performance |
| **Rate Limit Abuse** | 429 errors | > 100 from single IP in 5 minutes | Review IP for potential blocking |
| **JWT Validation Spike** | 401 errors | > 20% of total requests for 5 minutes | Check Salesforce JWKS endpoint availability |

### 5.1.8 Reporting

**Weekly Operational Report:**

- Total API calls (by endpoint, by brand)
- Average response time (by endpoint)
- Error rate breakdown (401, 429, 4xx, 5xx)
- Top 10 most active users (by call count)
- Salesforce API quota consumption

**Monthly Business Report:**

- Active users (MAU) by brand (TBO, Cellarbrations, Porters)
- API adoption rate (mobile app version distribution)
- Feature usage (most popular endpoints)
- Points redemption trends (from /loyalty/points API)
- Store search patterns (nearby stores, home store changes)

### 5.1.9 Integration and Testing Requirements

#### 5.1.9.1 Development and Integration Strategy

**Development Stages:**

1. **DEV Environment:**
   - APIM policy development and unit testing
   - Salesforce sandbox (metcash--dev) with test data
   - Integration User credentials configured in Key Vault
   - Test mobile app (iOS/Android simulators) pointing to DEV APIM

2. **SIT Environment:**
   - End-to-end integration testing with SIT Salesforce sandbox
   - Regression testing (all 23 endpoints)
   - Performance testing (load generation with JMeter)
   - Security testing (OWASP ZAP, Burp Suite)

3. **UAT Environment:**
   - User acceptance testing with business stakeholders
   - Mobile app (TestFlight/Firebase) pointing to UAT APIM
   - UAT Salesforce sandbox with production-like data
   - Business validation of loyalty program calculations

4. **PREPROD Environment:**
   - Production readiness validation
   - Smoke testing before PROD deployment
   - Monitoring and alerting validation
   - Rollback plan verification

5. **PROD Environment:**
   - Blue-green deployment (gradual traffic shift)
   - Post-deployment validation (synthetic transactions)
   - Monitoring dashboards active
   - On-call engineer standby

#### 5.1.9.2 Unit Testing

**APIM Policy Testing:**

1. **JWT Validation Tests:**
   - Valid JWT with correct signature → 200 OK
   - Expired JWT → 401 Unauthorized
   - Invalid signature → 401 Unauthorized
   - Missing JWT (authenticated endpoint) → 401 Unauthorized
   - Missing JWT (unauthenticated endpoint) → 200 OK

2. **Rate Limiting Tests:**
   - 50 requests in 60 seconds (authenticated) → All succeed
   - 51st request in 60 seconds (authenticated) → 429 Too Many Requests
   - Wait 60 seconds → Rate limit resets

3. **SOQL Query Construction Tests:**
   - User ID extracted from JWT → Correct semi-join query
   - Per-brand loyalty program ID mapping → Correct ProgramId in query
   - SQL injection attempt in JWT claim → Rejected (JWT validation failure)

4. **Integration User Token Tests:**
   - Token in cache → Use cached token (no refresh)
   - Token not in cache → Refresh and cache token
   - Token expired → Refresh and update cache

5. **Error Handling Tests:**
   - Salesforce 500 error → SMC logged, 500 returned to client
   - Salesforce timeout → 504 Gateway Timeout
   - Invalid SOQL query → APIM policy error, 500 returned

**Mobile App Integration Tests:**

1. **Authentication Flow:**
   - User logs in via Salesforce Headless Identity → JWT obtained
   - Mobile app calls /account with JWT → Account data returned

2. **Token Expiration:**
   - JWT expires after 2 hours → 401 Unauthorized
   - Mobile app refreshes token → New JWT obtained
   - Retry API call with new JWT → Success

3. **Network Resilience:**
   - API call fails (network error) → Mobile app retries with exponential backoff
   - Timeout (30 seconds) → Mobile app shows error, allows retry

---

# 6 Architecture Overview

## 6.1 Physical Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────────┐
│                          MOBILE APPLICATIONS                            │
├─────────────────────────────────────────────────────────────────────────┤
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐             │
│  │  The Bottle-O│    │Cellarbrations│    │   Porters    │             │
│  │  (iOS/Android)│   │ (iOS/Android)│    │(iOS/Android) │             │
│  └──────┬───────┘    └──────┬───────┘    └──────┬───────┘             │
│         │                    │                    │                     │
└─────────┼────────────────────┼────────────────────┼─────────────────────┘
          │                    │                    │
          │ HTTPS (OAuth 2.1 + PKCE)                │
          └─────┬──────────────┴────────────────────┘
                │
                ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                        SALESFORCE EXPERIENCE CLOUD                      │
│                         (Spring '26, API v66.0)                         │
├─────────────────────────────────────────────────────────────────────────┤
│  ┌───────────────────────────────────────────────────────────┐         │
│  │  Headless Identity API (Direct from Mobile Apps)          │         │
│  │  - /services/auth/headless/init/registration              │         │
│  │  - /services/oauth2/authorize (PKCE)                      │         │
│  │  - /services/oauth2/token                                 │         │
│  │  - /services/auth/headless/forgot_password                │         │
│  └───────────────────────────────────────────────────────────┘         │
│                                                                         │
│  ┌───────────────────────────────────────────────────────────┐         │
│  │  JWKS Endpoint (JWT Signature Verification)               │         │
│  │  - /.well-known/openid-configuration                      │         │
│  │  - /id/keys (public keys)                                 │         │
│  └───────────────────────────────────────────────────────────┘         │
└─────────────────────────────────────────────────────────────────────────┘
                │                             ▲
                │ JWT Token                   │ SOQL Queries
                ▼                             │ (Integration User)
┌─────────────────────────────────────────────────────────────────────────┐
│                      AZURE API MANAGEMENT (APIM)                        │
│                     t3-met-core-{env}-ae1-apim                          │
├─────────────────────────────────────────────────────────────────────────┤
│  ┌───────────────────────────────────────────────────────────┐         │
│  │  Gateway Layer (iba-loyalty/v1)                           │         │
│  │  - JWT Validation (JWKS)                                  │         │
│  │  - Rate Limiting (token bucket)                           │         │
│  │  - Per-Brand Mapping (client_id → loyalty_program_id)     │         │
│  │  - Integration User Token Cache (15-minute TTL)           │         │
│  └───────────────────────────────────────────────────────────┘         │
│                                                                         │
│  ┌───────────────────────────────────────────────────────────┐         │
│  │  Policy Framework                                         │         │
│  │  - API-Level Policies (inbound/outbound/on-error)        │         │
│  │  - Operation-Level Policies (23 endpoints)               │         │
│  │  - SMC Logger Fragment (error handling)                  │         │
│  └───────────────────────────────────────────────────────────┘         │
│                                                                         │
│  ┌───────────────────────────────────────────────────────────┐         │
│  │  Backend Services                                         │         │
│  │  - Salesforce REST API (v66.0)                            │         │
│  │  - SMC API (monitoring/smc/v1)                            │         │
│  └───────────────────────────────────────────────────────────┘         │
└─────────────────────────────────────────────────────────────────────────┘
       │                             │                     │
       │ Secrets                     │ Telemetry           │ Errors
       ▼                             ▼                     ▼
┌───────────────┐         ┌─────────────────────┐  ┌──────────────┐
│  Azure Key    │         │  Application        │  │     SMC      │
│  Vault        │         │  Insights           │  │  (Solution   │
│               │         │  (Diagnostics)      │  │  Monitoring) │
└───────────────┘         └─────────────────────┘  └──────────────┘

┌─────────────────────────────────────────────────────────────────────────┐
│                           SALESFORCE DATA LAYER                         │
├─────────────────────────────────────────────────────────────────────────┤
│  ┌────────────┐  ┌──────────────────┐  ┌────────────────────┐         │
│  │  Account   │  │ LoyaltyProgram   │  │ TransactionJournal │         │
│  │  (Person)  │  │ Member           │  │ (Points History)   │         │
│  └────────────┘  └──────────────────┘  └────────────────────┘         │
│                                                                         │
│  ┌────────────┐  ┌──────────────────┐  ┌────────────────────┐         │
│  │  User      │  │ LoyaltyTierGroup │  │ Product2 /         │         │
│  │  (Login)   │  │ (Tiers)          │  │ PricebookEntry     │         │
│  └────────────┘  └──────────────────┘  └────────────────────┘         │
└─────────────────────────────────────────────────────────────────────────┘
```

## 6.2 Network Connectivity

**Mobile App → Salesforce (Direct):**
- **Protocol:** HTTPS (TLS 1.2+)
- **Ports:** 443
- **DNS:** metcash.my.salesforce.com (PROD), metcash--{env}.sandbox.my.salesforce.com (non-PROD)
- **Purpose:** OAuth 2.1 authentication, JWT issuance

**Mobile App → APIM:**
- **Protocol:** HTTPS (TLS 1.2+)
- **Ports:** 443
- **DNS:** api-dev.metcash.com (DEV), api.metcash.com (PROD)
- **Purpose:** API requests with JWT

**APIM → Salesforce JWKS:**
- **Protocol:** HTTPS (TLS 1.2+)
- **Ports:** 443
- **Frequency:** On-demand (cached public keys for 24 hours)
- **Purpose:** JWT signature verification

**APIM → Salesforce REST API:**
- **Protocol:** HTTPS (TLS 1.2+)
- **Ports:** 443
- **Authentication:** Bearer token (Integration User)
- **Frequency:** Per API request (synchronous)
- **Purpose:** SOQL queries, DML operations

**APIM → Azure Key Vault:**
- **Protocol:** HTTPS (TLS 1.2+)
- **Ports:** 443
- **Authentication:** Managed Identity
- **Frequency:** On-demand (secrets cached for 24 hours)
- **Purpose:** Retrieve Integration User credentials

**APIM → Application Insights:**
- **Protocol:** HTTPS (TLS 1.2+)
- **Ports:** 443
- **Authentication:** Instrumentation Key
- **Frequency:** Real-time (telemetry batching)
- **Purpose:** Diagnostics, performance metrics

**APIM → SMC:**
- **Protocol:** HTTPS (TLS 1.2+)
- **Ports:** 443
- **Authentication:** APIM Subscription Key
- **Frequency:** On error (asynchronous)
- **Purpose:** Error logging, alerting

## 6.3 Security

### 6.3.1 Data Handling and Encryption

**Encryption in Transit:**
- **TLS 1.2+** enforced for all HTTPS communication
- **Certificate validation** enabled (no self-signed certificates)
- **Cipher suites:** ECDHE-RSA-AES256-GCM-SHA384, ECDHE-RSA-AES128-GCM-SHA256 (forward secrecy)

**Encryption at Rest:**
- **Azure Key Vault:** AES-256 encryption for secrets (Integration User credentials)
- **Salesforce:** AES-256 encryption for sensitive fields (PII, payment data)

**PII Protection:**
- **Diagnostic Logs:** Password fields excluded from Application Insights
- **JWT Tokens:** Not logged in plaintext (correlation ID used for tracing)
- **Email Addresses:** Hashed in logs (one-way SHA-256 hash)

### 6.3.2 Authentication Architecture

**Two-Token Pattern:**

1. **User JWT (Issued by Salesforce):**
   - **Purpose:** Authenticate mobile app user
   - **Issuer:** Salesforce Headless Identity
   - **Lifetime:** 2 hours (configurable in Salesforce External Client Apps)
   - **Claims:** `sub` (User ID), `iss`, `aud`, `exp`, `client_id`
   - **Validation:** APIM validates signature via Salesforce JWKS

2. **Integration User Token (Issued by Salesforce):**
   - **Purpose:** APIM backend calls to Salesforce REST API
   - **Issuer:** Salesforce OAuth Token Endpoint
   - **Lifetime:** 15 minutes (cached in APIM, auto-refresh on expiration)
   - **Grant Type:** `refresh_token` (long-lived refresh token stored in Key Vault)
   - **Scope:** Full Salesforce REST API access (limited by Integration User permissions)

### 6.3.3 IDOR Prevention

**Semi-Join SOQL Pattern:**

Traditional vulnerable approach (IDOR risk):
```http
GET /account?accountId=001U900000ABCDEFGH
Authorization: Bearer {user_jwt}
```
**Vulnerability:** User can manipulate `accountId` parameter to access other users' data.

**Secure approach (semi-join):**
```http
GET /account
Authorization: Bearer {user_jwt}
```
APIM constructs SOQL server-side:
```sql
SELECT ... FROM Account
WHERE PersonContactId IN (SELECT ContactId FROM User WHERE Id = '{userId_from_JWT}')
```
**Security:** Account ID derived from JWT User ID; no client-provided parameter.

**Benefits:**
- **No parameter tampering:** Client cannot manipulate which Account is accessed
- **Server-side authorization:** APIM enforces data access rules
- **Audit trail:** Correlation ID tracks which user accessed which data

## 6.4 Infrastructure Support

**Azure Resources:**

| Resource Type | Resource Name | Purpose |
|---------------|---------------|---------|
| **API Management** | t3-met-core-{env}-ae1-apim | API gateway, policy enforcement |
| **Key Vault** | t3-met-core-{env}-ae1-kv | Secrets management (Integration User credentials) |
| **Application Insights** | t3-met-core-{env}-ae1-ai | Diagnostics, telemetry, performance metrics |
| **Virtual Network** | t3-met-core-{env}-ae1-vnet | Secure backend communication |
| **Resource Group** | t3-met-core-{env}-ae1-rsg-apim | Resource grouping for APIM components |

**Salesforce Resources:**

| Resource Type | Resource Name | Purpose |
|---------------|---------------|---------|
| **External Client Apps** | IBA-Loyalty-TBO, IBA-Loyalty-CBN, IBA-Loyalty-Porters | OAuth 2.1 configurations for mobile apps |
| **Integration User** | integration.user@metcash.com | Service account for APIM backend calls |
| **Experience Cloud Site** | /loyalty | Community portal (not used by APIM, direct mobile app access only) |

## 6.5 Infrastructure Requirements

**APIM Tier:**
- **Minimum:** Standard tier (for Custom Domain, Virtual Network integration)
- **Recommended:** Premium tier (for multi-region deployment, additional scale units)

**Compute Resources:**
- **APIM Capacity:** 2 scale units (handles 5,000 concurrent connections)
- **Expected RPS:** 50 RPS (peak), 5 RPS (average)
- **Scaling:** Auto-scale enabled (scale out to 4 units at 80% CPU utilization)

**Storage Requirements:**
- **APIM Policy Cache:** 100 MB (cached JWT public keys, Integration User tokens)
- **Application Insights:** 10 GB/month (telemetry data, 90-day retention)
- **SMC Logs:** 630 MB (90-day retention, ~3,650 log entries/day)

**Network Requirements:**
- **Outbound Connections:** APIM → Salesforce (443), APIM → Key Vault (443), APIM → Application Insights (443), APIM → SMC (443)
- **Inbound Connections:** Mobile Apps → APIM (443)
- **Bandwidth:** 10 Mbps (average), 100 Mbps (peak)

---

# 7 Communication Overview

## 7.1 Communication Components

**Communication Protocols:**

| Source | Destination | Protocol | Port | Authentication | Purpose |
|--------|-------------|----------|------|----------------|---------|
| Mobile App | Salesforce Headless Identity | HTTPS | 443 | OAuth 2.1 + PKCE | User authentication, JWT issuance |
| Mobile App | APIM Gateway | HTTPS | 443 | Bearer JWT | API requests |
| APIM | Salesforce JWKS | HTTPS | 443 | None (public endpoint) | JWT signature verification |
| APIM | Salesforce REST API | HTTPS | 443 | Bearer token (Integration User) | SOQL queries, DML operations |
| APIM | Azure Key Vault | HTTPS | 443 | Managed Identity | Secrets retrieval |
| APIM | Application Insights | HTTPS | 443 | Instrumentation Key | Telemetry |
| APIM | SMC API | HTTPS | 443 | APIM Subscription Key | Error logging |

**Data Formats:**

- **Request/Response:** JSON (application/json)
- **Authentication:** JWT (Bearer token in Authorization header)
- **Error Responses:** RFC 7807 Problem Details (application/problem+json)

**Headers:**

| Header | Direction | Purpose |
|--------|-----------|---------|
| `Authorization` | Mobile App → APIM, APIM → Salesforce | Bearer token (JWT or Integration User token) |
| `Content-Type` | Both | `application/json` |
| `Accept` | Mobile App → APIM | `application/json` |
| `x-correlation-id` | APIM → Mobile App | Request tracing (UUID) |
| `X-Frame-Options` | APIM → Mobile App | Security header (`DENY`) |
| `X-Content-Type-Options` | APIM → Mobile App | Security header (`nosniff`) |

---

# 8 Data Architecture

## 8.1 Data Model

**Salesforce Data Relationships:**

```
User (Login Identity)
  │
  ├── ContactId ──────────────────┐
  │                                │
  ▼                                ▼
Contact (Standard)           Account (Person Account)
  │                                │
  │                                ├── Loyalty_Program_Member_TBO__pc
  │                                ├── Loyalty_Program_Member_CBN__pc
  │                                ├── CBN_Home_Store__pc (Lookup to Account)
  │                                ├── TBO_Home_Store__pc
  │                                └── Personal Info (FirstName, LastName, Email, etc.)
  │
  ▼
LoyaltyProgramMember
  │
  ├── ProgramId ──────────────────► LoyaltyProgram (TBO/CBN/Porters)
  │
  ├── MemberId ───────────────────► TransactionJournal (Points History)
  │
  └── MemberId ───────────────────► LoyaltyMemberCurrency (Points Balance)
```

## 8.2 Salesforce Data Objects

**Account (Person Account):**

| Field Name | API Name | Type | Purpose |
|------------|----------|------|---------|
| Account ID | `Id` | ID | Unique identifier |
| First Name | `FirstName` | Text | User's first name |
| Last Name | `LastName` | Text | User's last name |
| Email | `PersonEmail` | Email | User's email address |
| Birthdate | `PersonBirthdate` | Date | User's birthdate |
| Postal Code | `PersonMailingPostalCode` | Text | User's postal code |
| Phone | `Phone` | Phone | User's phone number |
| TBO Member ID | `Loyalty_Program_Member_TBO__pc` | Lookup | TBO loyalty program member |
| CBN Member ID | `Loyalty_Program_Member_CBN__pc` | Lookup | Cellarbrations loyalty program member |
| CBN Home Store | `CBN_Home_Store__pc` | Lookup | Cellarbrations home store |
| TBO Home Store | `TBO_Home_Store__pc` | Lookup | TBO home store |
| Email Notifications | `Email_Notifications__pc` | Checkbox | Email notification preference |
| Push Notifications | `Push_Notifications__pc` | Checkbox | Push notification preference |

**User (Salesforce Login):**

| Field Name | API Name | Type | Purpose |
|------------|----------|------|---------|
| User ID | `Id` | ID | Unique identifier (JWT sub claim) |
| Username | `Username` | Email | Login username |
| Email | `Email` | Email | User's email |
| Contact ID | `ContactId` | Lookup | Links to Person Account via Contact |
| Is Active | `IsActive` | Checkbox | Login enabled/disabled |

**LoyaltyProgramMember:**

| Field Name | API Name | Type | Purpose |
|------------|----------|------|---------|
| Member ID | `Id` | ID | Unique identifier |
| Contact ID | `ContactId` | Lookup | Links to Contact (Person Account) |
| Program ID | `ProgramId` | Lookup | Loyalty program (TBO/CBN/Porters) |
| Enrollment Date | `EnrollmentDate` | Date | Date user enrolled |
| Member Status | `MemberStatus` | Picklist | Active/Inactive |

**TransactionJournal (Points History):**

| Field Name | API Name | Type | Purpose |
|------------|----------|------|---------|
| Transaction ID | `Id` | ID | Unique identifier |
| Member ID | `MemberId` | Lookup | Loyalty program member |
| Activity Date | `ActivityDate` | DateTime | Transaction date |
| Points | `Points_Mobile__c` | Number | Points earned/redeemed |
| Transaction Type | `TransactionType` | Picklist | EARN/REDEEM/BONUS |
| Points Info | `Points_Info_Mobile__c` | Text | Transaction description |

**Product2 / PricebookEntry (Products & Promotions):**

| Field Name | API Name | Type | Purpose |
|------------|----------|------|---------|
| Product ID | `Product2.Id` | ID | Unique identifier |
| Product Name | `Product2.Name` | Text | Product name |
| Description | `Product2.Description` | Text | Product description |
| Unit Price | `PricebookEntry.UnitPrice` | Currency | Promotional price |
| Pricebook ID | `Pricebook2Id` | Lookup | Pricebook (TBO/CBN/Porters) |

## 8.3 Data Retention

### 8.3.1 Data Retention Procedures

**APIM Data Retention:**

| Data Type | Retention Period | Storage Location |
|-----------|------------------|------------------|
| **API Request/Response Logs** | 90 days | Application Insights |
| **Error Logs (SMC)** | 90 days | SMC Database |
| **Integration User Token Cache** | 15 minutes | APIM In-Memory Cache |
| **JWT Public Key Cache** | 24 hours | APIM In-Memory Cache |

**Salesforce Data Retention:**

| Data Type | Retention Period | Storage Location |
|-----------|------------------|------------------|
| **User Accounts** | Indefinite (until user requests deletion) | Salesforce |
| **Transaction History** | 7 years (tax compliance) | Salesforce |
| **Audit Logs** | 1 year | Salesforce Event Monitoring |

### 8.3.2 Data Backup

**APIM Configuration Backup:**
- **Frequency:** Daily automated backup via Azure Backup
- **Retention:** 30 days (for PROD), 7 days (for non-PROD)
- **Scope:** APIM policies, APIs, operations, backends, named values

**Salesforce Data Backup:**
- **Frequency:** Daily automated backup via Salesforce Data Export
- **Retention:** Per Salesforce backup policies (depends on Salesforce edition)
- **Scope:** Account, User, LoyaltyProgramMember, TransactionJournal

### 8.3.3 Data Destruction Procedures

**GDPR Compliance (Right to be Forgotten):**

When a user requests account deletion:

1. **Salesforce:** User account and associated data (Account, LoyaltyProgramMember, TransactionJournal) are anonymized or deleted per Salesforce retention policies.
2. **APIM Logs (Application Insights):** User data in logs automatically purged after 90-day retention period. Immediate deletion not supported (Application Insights limitation).
3. **SMC Logs:** User data in error logs automatically purged after 90-day retention period.

**Data Destruction Timeline:**
- **Immediate:** Salesforce User.IsActive set to false (login disabled)
- **Within 30 days:** Salesforce data anonymization (PII removed, transaction history preserved for tax compliance)
- **Within 90 days:** APIM/SMC logs purged automatically

---

# 9 Deployment and Monitoring Considerations

## 9.1 Transition Period

**Legacy to New System Transition:**

Since this is a new integration (no legacy API gateway to replace), the transition involves:

1. **Mobile App Update:**
   - **Phase 1 (Soft Launch):** 10% of users receive updated mobile app (pointing to APIM) via phased rollout (TestFlight/Firebase)
   - **Phase 2 (Beta):** 50% of users after 1 week of successful Phase 1
   - **Phase 3 (General Availability):** 100% of users after 2 weeks of successful Phase 2

2. **Parallel Running:**
   - During phased rollout, old mobile app version (direct Salesforce calls) and new version (via APIM) coexist
   - Salesforce backend remains unchanged (both approaches use same data)

3. **Rollback Plan:**
   - If APIM experiences critical issues, mobile app update can be paused/rolled back via app store rollback
   - Users revert to direct Salesforce calls (no data loss)

## 9.2 Deployment Preference

**Blue-Green Deployment Strategy:**

1. **Blue Environment (Current):**
   - APIM configuration deployed but not yet receiving traffic
   - Smoke tests executed (synthetic transactions)

2. **Green Environment (New):**
   - Traffic gradually shifted from Blue to Green (10% → 50% → 100%)
   - Monitoring dashboards active, alerts configured

3. **Rollback:**
   - If issues detected, traffic shifted back to Blue environment
   - Investigation and fix applied, redeployment attempted

**Deployment Automation:**

- **Infrastructure as Code:** Azure Resource Manager (ARM) templates or Bicep for APIM provisioning
- **CI/CD Pipeline:** Azure DevOps or GitHub Actions for policy deployment
- **Configuration Management:** Named values (Key Vault references) managed via ARM templates

## 9.3 Deployment Environment Migration

**Environment Promotion Path:**

```
DEV → SIT → UAT → PREPROD → PROD
```

**Promotion Checklist:**

| Step | DEV | SIT | UAT | PREPROD | PROD |
|------|-----|-----|-----|---------|------|
| APIM policies deployed | ✓ | ✓ | ✓ | ✓ | ✓ |
| Named values configured | ✓ | ✓ | ✓ | ✓ | ✓ |
| Key Vault secrets populated | ✓ | ✓ | ✓ | ✓ | ✓ |
| Salesforce Integration User created | ✓ | ✓ | ✓ | ✓ | ✓ |
| External Client Apps configured | ✓ | ✓ | ✓ | ✓ | ✓ |
| Application Insights configured | ✓ | ✓ | ✓ | ✓ | ✓ |
| SMC onboarding completed | - | ✓ | ✓ | ✓ | ✓ |
| Smoke tests passed | ✓ | ✓ | ✓ | ✓ | ✓ |
| Regression tests passed | - | ✓ | ✓ | ✓ | - |
| Performance tests passed | - | ✓ | - | ✓ | - |
| UAT sign-off obtained | - | - | ✓ | - | - |
| CAB approval obtained | - | - | - | - | ✓ |

---

# 10 Disaster Recovery

**Recovery Time Objective (RTO):** 1 hour  
**Recovery Point Objective (RPO):** 15 minutes (Integration User token cache)

**Disaster Scenarios:**

1. **APIM Gateway Failure:**
   - **Mitigation:** Azure APIM Premium tier with multi-region deployment (active-active)
   - **Failover:** Automatic DNS failover to secondary region (< 5 minutes)
   - **Recovery:** Restore from Azure Backup (< 1 hour)

2. **Salesforce Outage:**
   - **Mitigation:** No APIM-side mitigation (Salesforce is single source of truth)
   - **Impact:** All API endpoints unavailable (returns 503 Service Unavailable)
   - **Communication:** Status page updated, mobile app shows "Service temporarily unavailable" message

3. **Key Vault Unavailability:**
   - **Mitigation:** APIM caches Integration User credentials (15-minute TTL)
   - **Impact:** Token refresh fails after cache expiration; API continues with cached token until expiration
   - **Recovery:** Key Vault geo-redundancy (automatic failover within Azure region)

4. **Application Insights Outage:**
   - **Mitigation:** Non-blocking; telemetry failures do not affect API requests
   - **Impact:** Temporary loss of diagnostic data (recovered when Application Insights restored)

**Backup and Restore:**

- **APIM Configuration:** Daily automated backup (ARM template export)
- **Restore Procedure:**
  1. Provision new APIM instance via ARM template
  2. Import policies, APIs, operations from backup
  3. Update Key Vault references (named values)
  4. Update DNS (custom domain CNAME)
  5. Smoke test and validate

**Communication Plan:**

- **Incident Commander:** Integration team lead
- **Stakeholders:** Mobile app team, Salesforce admin, Business stakeholders
- **Communication Channels:** Slack (#iba-loyalty-incidents), Email (integration@metcash.com), Status Page (status.metcash.com)

---

# 11 Support Documentation

**Runbooks:**

1. **JWT Validation Failures (401 Unauthorized):**
   - **Symptom:** Mobile app users unable to access API (401 errors)
   - **Diagnosis:** Check Application Insights for JWT validation error messages
   - **Resolution:** Verify Salesforce JWKS endpoint availability, check APIM JWT policy configuration

2. **Salesforce 5xx Errors:**
   - **Symptom:** API returns 500 Internal Server Error
   - **Diagnosis:** Check SMC logs for Salesforce error details, review Salesforce debug logs
   - **Resolution:** Escalate to Salesforce support if persistent

3. **Rate Limit Exceeded (429 Too Many Requests):**
   - **Symptom:** High-volume users receiving 429 errors
   - **Diagnosis:** Identify user/IP with excessive API usage (Application Insights)
   - **Resolution:** Review if legitimate usage (increase rate limit) or abuse (block IP)

4. **Integration User Token Refresh Failures:**
   - **Symptom:** All API requests fail with 401 Unauthorized
   - **Diagnosis:** Check Key Vault for Integration User credentials, test refresh token manually
   - **Resolution:** Regenerate refresh token in Salesforce, update Key Vault secret

**Monitoring Dashboards:**

- **Application Insights Dashboard:** [Link to Azure Portal dashboard]
- **SMC Dashboard:** [Link to SMC portal]
- **Salesforce Event Monitoring:** [Link to Salesforce Setup → Event Monitoring]

**Contacts:**

| Role | Contact | Escalation |
|------|---------|------------|
| **APIM Support (L1)** | integration-support@metcash.com | On-call engineer (PagerDuty) |
| **Salesforce Support (L2)** | salesforce-admin@metcash.com | Salesforce Premium Support |
| **Business Owner** | iba-loyalty@metcash.com | Retail Operations Manager |

---

# 12 Limitations and Constraints

**Technical Limitations:**

1. **JWT Expiration:** JWT tokens expire after 2 hours (Salesforce configuration); mobile apps must handle token refresh.
2. **Rate Limiting:** Authenticated users limited to 50 calls/60 seconds; power users may require rate limit increase.
3. **SOQL Complexity:** Salesforce SOQL does not support nested semi-joins beyond 1 level; some queries require two-step resolution.
4. **Salesforce API Limits:** Daily API call limits shared across all Salesforce integrations; excessive usage may impact other systems.
5. **Cache TTL:** Integration User token cached for 15 minutes; token refresh adds latency (~200ms) on cache miss.

**Business Constraints:**

1. **Three Brands Only:** Current implementation supports TBO, Cellarbrations, Porters; additional brands require APIM configuration updates.
2. **Salesforce Dependency:** API availability dependent on Salesforce uptime (99.9% SLA); no offline mode.
3. **Schema Stability:** Changes to Salesforce custom field names (e.g., `Loyalty_Program_Member_TBO__pc`) require APIM policy updates.

**Operational Constraints:**

1. **Deployment Windows:** PROD deployments require CAB approval and after-hours deployment window (to minimize user impact).
2. **Testing Environments:** SIT/UAT Salesforce sandboxes have limited data refresh frequency (weekly); test data may become stale.

---

# 13 Assumptions

The following assumptions underpin this technical design:

1. **Salesforce Availability:** Salesforce Experience Cloud maintains 99.9% uptime SLA.
2. **JWT Token Issuance:** Mobile apps successfully implement OAuth 2.1 Authorization Code + PKCE flow with Salesforce Headless Identity.
3. **User Behavior:** Average user makes 20 API calls per day; rate limits (50/60s) accommodate burst usage.
4. **Data Quality:** Salesforce data (Account, LoyaltyProgramMember) is accurate and up-to-date; no data validation required beyond Salesforce schema constraints.
5. **Network Reliability:** Mobile devices have stable internet connectivity; APIM assumes synchronous request/response pattern (no queuing).
6. **Salesforce Schema Stability:** Custom field names remain stable across Salesforce releases; breaking changes communicated in advance.
7. **Azure Region:** APIM and Key Vault deployed in Australia East region for latency optimization.
8. **Monitoring Tools:** Application Insights and SMC available and operational for diagnostics/alerting.

---

# 14 Reference Documents and Architecture

**Reference Documents:**

1. **Functional Specification:** EAII-472 - APIM to Salesforce API Technical Mapping (v2.4)
2. **Salesforce REST API Documentation:** [Salesforce REST API Developer Guide (v66.0)](https://developer.salesforce.com/docs/atlas.en-us.api_rest.meta/api_rest/)
3. **Salesforce Headless Identity:** [Headless Identity API Guide](https://developer.salesforce.com/docs/atlas.en-us.externalidentityImplGuide.meta/externalidentityImplGuide/external_identity_headless.htm)
4. **OAuth 2.1 + PKCE:** [RFC 7636 - Proof Key for Code Exchange](https://datatracker.ietf.org/doc/html/rfc7636)
5. **JWT (RFC 7519):** [JSON Web Token Specification](https://datatracker.ietf.org/doc/html/rfc7519)
6. **Azure APIM Policies:** [Azure API Management Policy Reference](https://learn.microsoft.com/en-us/azure/api-management/api-management-policies)
7. **Metcash Integration Design Principles:** [Integration Design Principles - v1.0](https://metcash.atlassian.net/wiki/spaces/MAP/pages/1656913947)
8. **AIS Environments Strategy:** [Environments Strategy](https://metcash.atlassian.net/wiki/spaces/MAP/pages/1637548092)

**Architecture Diagrams:**

- Physical Architecture Diagram (Section 6.1)
- Process Flow Diagram (Section 5.1.2.2)
- Sequence Diagram (Section 5.1.2.3)
- Data Model (Section 8.1)

---

# 15 TSD Approval Checklist

| Approval Item | Status | Approver | Date |
|---------------|--------|----------|------|
| **Technical Design Reviewed** | ☐ Pending | Solution Architect | |
| **Security Review Completed** | ☐ Pending | Security Team | |
| **Salesforce Schema Validated** | ☐ Pending | CRM Team | |
| **Performance Testing Plan Approved** | ☐ Pending | Test Manager | |
| **Monitoring and Alerting Configured** | ☐ Pending | Integration Platform Operations | |
| **Disaster Recovery Plan Reviewed** | ☐ Pending | Infrastructure Team | |
| **CAB Submission Prepared** | ☐ Pending | Release Manager | |
| **Business Stakeholder Sign-Off** | ☐ Pending | Retail Operations Manager | |

---

**Document Control:**

- **Version:** 1.0
- **Status:** Draft
- **Next Review Date:** March 24, 2026
- **Document Owner:** Integration Team - Metcash

---

**END OF TECHNICAL SPECIFICATION DOCUMENT**
