# TriageX.AI — AI-Powered Cloud Incident Triage Platform

![AWS](https://img.shields.io/badge/AWS-FF9900?style=for-the-badge&logo=amazonwebservices&logoColor=white)
![Terraform](https://img.shields.io/badge/Terraform-5835CC?style=for-the-badge&logo=terraform&logoColor=white)
![React](https://img.shields.io/badge/React-20232A?style=for-the-badge&logo=react&logoColor=61DAFB)
![Microsoft Entra ID](https://img.shields.io/badge/Microsoft%20Entra%20ID-0078D4?style=for-the-badge&logo=microsoftentra&logoColor=white)
![Python](https://img.shields.io/badge/Python-3670A0?style=for-the-badge&logo=python&logoColor=FFDD54)
![Amazon Bedrock](https://img.shields.io/badge/Amazon%20Bedrock-Generative%20AI-FF9900?style=for-the-badge)
![Serverless Design](https://img.shields.io/badge/Architecture-Serverless%20Design-2563EB?style=for-the-badge)
![Project Status](https://img.shields.io/badge/Status-In%20Development-EAB308?style=for-the-badge)

**TriageX.AI** is an AI-powered cloud incident triage platform designed to help Cloud Support Engineers, Site Reliability Engineers (SREs), and operations teams investigate AWS incidents faster.

The project brings incident telemetry, generative AI analysis, relevant operational documentation, and engineer review into one investigation workflow. Its target architecture combines AWS serverless services, Amazon Bedrock, Retrieval-Augmented Generation (RAG), and Microsoft Entra ID to produce evidence-backed incident reports and recommended next steps.

> **Under development:** The existing project documentation describes a core incident-summary flow. Event ingestion, RAG, structured reporting, and human review are planned extensions unless confirmed by the repository implementation. Reducing investigation time and Mean Time To Resolution (MTTR) is a project goal; no measured improvement is claimed.

<a id="architecture-diagrams"></a>

## 🗺️ Architecture Diagrams

### Visual Project Blueprint

![TriageX.AI frontend architecture blueprint](TriageX.AI%20Frontend%20Blueprint.PNG)

### Backend Blueprint

![TriageX.AI backend architecture blueprint](TriageX.AI%20Backend%20Blueprint.PNG)

*Diagram placeholders: add the corresponding PNG files to `docs/` when publishing this README at the repository root. The blueprints represent the intended architecture.*

<a id="contents"></a>

## 📑 Contents

- [🎯 Problem](#problem)
- [💡 Solution](#solution)
- [✨ Key Capabilities](#key-capabilities)
- [📚 Retrieval-Augmented Generation](#retrieval-augmented-generation)
- [👤 Human-in-the-Loop Review](#human-in-the-loop-review)
- [🏗️ Architecture](#architecture)
- [🔄 Incident Processing Workflow](#incident-processing-workflow)
- [🧰 Technology Stack](#technology-stack)
- [🔐 Authentication and Security](#authentication-and-security)
- [🗄️ Data Model](#data-model)
- [🌐 API Design](#api-design)
- [🧪 Example Incident](#example-incident)
- [📁 Project Structure](#project-structure)
- [📋 Prerequisites](#prerequisites)
- [🚀 Deployment Guide](#deployment-guide)
- [🛣️ Roadmap](#roadmap)
- [🚧 Project Status](#project-status)
- [⚠️ Disclaimer](#disclaimer)

<a id="problem"></a>

## 🎯 Problem

Cloud incidents leave evidence across application logs, infrastructure metrics, network records, and security findings. Engineers must correlate those signals, locate the right runbook, assess customer impact, and explain the incident while working under time pressure.

Common investigation tasks include:

- Searching CloudWatch logs and metrics for relevant events.
- Reviewing GuardDuty and Security Hub findings.
- Examining VPC traffic, EC2 behavior, and RDS errors.
- Connecting events across services and identifying affected resources.
- Finding troubleshooting procedures and previous incident reports.
- Separating observed facts from root-cause hypotheses.
- Preparing a remediation plan and communicating it to stakeholders.

Fragmented evidence and inconsistent investigation notes can slow response and make handoffs difficult.

<a id="solution"></a>

## 💡 Solution

TriageX is designed to turn raw incident data into a structured investigation that an engineer can inspect and validate.

The intended workflow collects and normalizes incident evidence, retrieves relevant operational knowledge, asks Amazon Bedrock to analyze the combined context, and stores the resulting report for review in a React dashboard.

Each report is intended to include:

- Technical and executive summaries.
- Suggested severity and potential impact.
- Probable root cause, alternative explanations, and missing evidence.
- A confidence assessment with supporting rationale.
- Evidence references and retrieved document citations.
- Recommended investigation and remediation steps.
- Engineer review status and notes.

The engineering focus spans event-driven processing, API development, identity integration, NoSQL persistence, AI orchestration, observability, and reproducible infrastructure with Terraform.

<a id="key-capabilities"></a>

## ✨ Key Capabilities

| Capability | Purpose | Documentation status |
|---|---|---|
| Incident summarization | Retrieve a stored incident and generate a readable Bedrock summary | Described in the original README |
| Authenticated dashboard access | Use Microsoft Entra ID, API Gateway, and Lambda token validation | Described in the original README; authorization details require implementation verification |
| Infrastructure as Code | Provision AWS resources with Terraform | Described in the original README |
| Multi-source ingestion | Normalize operational events, logs, and security findings | Planned extension |
| Structured investigation | Present severity, probable cause, impact, confidence, and evidence | Planned extension |
| RAG knowledge retrieval | Ground analysis in runbooks, SOPs, and prior incidents | Planned extension |
| Human review and feedback | Record approval, requested changes, false positives, and notes | Planned extension |
| Incident management UI | View incident status, raw evidence, analysis versions, and review history | Planned extension beyond the summary dashboard |

These labels reflect the available project documentation rather than a code audit or deployment test. Recommended remediation steps do not imply tested AWS CLI commands or automated execution.

<a id="retrieval-augmented-generation"></a>

## 📚 Retrieval-Augmented Generation

The planned RAG pipeline supplies relevant operational knowledge before the model generates an incident analysis. Candidate sources include approved incident-response runbooks, Standard Operating Procedures (SOPs), troubleshooting guides, AWS documentation, and reviewed historical incident reports.

```text
Approved documents → Amazon S3 → Parse and chunk → Embedding model
                                                        │
                                                        ▼
                                               OpenSearch vector index
                                                        ▲
Incident context → Query embedding → Retrieve relevant chunks
                                                        │
                                                        ▼
                         Incident evidence + cited context → Bedrock analysis
```

Amazon S3 stores source documents; Amazon OpenSearch is the proposed vector search layer. An embedding model must be selected separately from the generative model. Indexing and query embeddings must use compatible models and dimensions.

The retrieval design should preserve document identifiers, versions, chunk identifiers, and source locations so engineers can inspect citations. Document access controls should apply during retrieval. If relevant documentation is unavailable, the analysis should disclose that limitation and identify additional evidence to collect.

Retrieved context can improve relevance, but it does not establish that a diagnosis is correct. Logs and documents must be treated as evidence, including potentially untrusted text, rather than instructions that override the analysis policy.

<a id="human-in-the-loop-review"></a>

## 👤 Human-in-the-Loop Review

The planned review workflow keeps the engineer responsible for the final decision. After inspecting the evidence and recommendations, an engineer can:

- **Approve** the analysis as useful guidance.
- **Request changes** when findings need revision or additional evidence.
- **Mark a false positive** when the incident does not warrant the suggested response.
- **Add notes** documenting investigation findings and decisions.

Review records should identify the reviewer, timestamp, and exact analysis version reviewed. Incident lifecycle status and analysis review status are separate: approving an analysis does not resolve an incident or execute a change.

Feedback can support later evaluation and prompt improvements. Automatic model retraining from feedback is outside the documented scope.

<a id="architecture"></a>

## 🏗️ Architecture

### Core Flow Described in the Existing README

1. An engineer signs in to the React SPA through Microsoft Entra ID using MSAL.
2. The frontend sends an API access token with a request to Amazon API Gateway.
3. A Python Lambda validates the token and retrieves the requested incident from DynamoDB.
4. Lambda constructs the analysis prompt and invokes Amazon Bedrock. The original design names Anthropic Claude 3 Haiku.
5. The generated Markdown summary returns to the dashboard.

Terraform is the documented provisioning approach for the AWS backend. This flow is the starting point for the broader triage design.

### Target Architecture

```text
AWS operational events / security findings       CloudWatch / VPC log data
                  │                                         │
                  ▼                                         ▼
             EventBridge                         Source-specific log adapter
                  └──────────────────┬──────────────────────┘
                                     ▼
                          Lambda ingestion / normalization
                                     │
                                     ▼
                               DynamoDB incidents
                                     │
                                     ▼
                           Lambda analysis orchestration
                                     │
                    ┌────────────────┴────────────────┐
                    ▼                                 ▼
            S3 / OpenSearch RAG              Incident evidence
                    └────────────────┬────────────────┘
                                     ▼
                              Amazon Bedrock
                                     │
                                     ▼
                         Validate and persist analysis
                                     │
                                     ▼
                        DynamoDB analysis / review records

Engineer → React + Entra ID → API Gateway → Authorized API Lambdas
                                                 │
                                                 ▼
                                      Incident / analysis / review data
```

EventBridge is intended for supported service events and findings. Raw log delivery requires a source-specific integration; it should not be assumed that every log entry arrives on EventBridge. CloudWatch Logs supports subscription-based processing, and VPC Flow Logs can be delivered to supported log destinations. See the AWS guides for [CloudWatch Logs subscriptions](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/Subscriptions.html) and [VPC Flow Logs](https://docs.aws.amazon.com/vpc/latest/userguide/flow-logs.html).

The compute and API layers follow a serverless design. The OpenSearch deployment mode and frontend hosting choice remain implementation decisions; the complete target system should be described as fully serverless only when those choices support that claim.

<a id="incident-processing-workflow"></a>

## 🔄 Incident Processing Workflow

1. **Detect and ingest:** Receive a supported event, finding, or log batch through its configured integration.
2. **Normalize and enrich:** Extract source, timestamps, resource identifiers, and relevant evidence. Deduplicate repeated deliveries using a stable source event identifier where available.
3. **Persist the incident:** Store metadata in DynamoDB and retain a reference to larger raw payloads when needed.
4. **Retrieve context:** Search approved knowledge sources for relevant document chunks and preserve their citations.
5. **Generate analysis:** Send bounded incident evidence and retrieved context to Bedrock with a versioned prompt.
6. **Validate and store:** Check the response against the expected schema, retain model and prompt metadata, and record failures without treating malformed output as a completed report.
7. **Present findings:** Return incident details, analysis, evidence, and review state through authorized API endpoints.
8. **Review and follow up:** Record engineer feedback and any incident status changes. The engineer validates and carries out appropriate remediation through the normal operational process.

Retries, failure handling, and duplicate delivery protection are target reliability requirements, not verified capabilities of the current implementation.

<a id="technology-stack"></a>

## 🧰 Technology Stack

| Layer | Technology | Role |
|---|---|---|
| Frontend | React, Vite, Tailwind CSS | Incident dashboard and analysis presentation |
| Identity | Microsoft Entra ID, MSAL, OAuth 2.0 / OIDC | User sign-in and API access tokens |
| API | Amazon API Gateway | Protected HTTP endpoints |
| Compute | AWS Lambda, Python, `boto3` | API handling and analysis orchestration |
| Persistence | Amazon DynamoDB | Incident metadata and planned analysis/review records |
| Generative AI | Amazon Bedrock | Incident analysis; Claude 3 Haiku in the original design |
| Event routing | Amazon EventBridge | Planned supported event and finding ingestion |
| Knowledge storage | Amazon S3 | Planned source documents and larger evidence objects |
| Retrieval | Amazon OpenSearch and a selected embedding model | Planned semantic search |
| Observability | Amazon CloudWatch | Logs, metrics, and planned alarms |
| Access control | AWS IAM | Scoped deployment and runtime permissions |
| Infrastructure | HashiCorp Terraform | Reproducible resource provisioning |

Model identifiers, inference profiles, runtime versions, and dependency versions should follow the actual repository configuration and regional availability.

<a id="authentication-and-security"></a>

## 🔐 Authentication and Security

Microsoft Entra ID provides user identity; backend authorization must determine which incidents and operations that identity can access. The following are security requirements for the target implementation:

- Use MSAL's SPA sign-in flow with authorization code and PKCE. Do not place a client secret in the browser application.
- Send an **access token issued for the TriageX API**, rather than using an ID token as API authorization.
- Validate the token signature against trusted signing keys, expected issuer and tenant, API audience, and validity period. Enforce required scopes or app roles and resource-level access in the backend. Follow Microsoft's guidance on [access tokens](https://learn.microsoft.com/en-us/entra/identity-platform/access-tokens) and [claims validation](https://learn.microsoft.com/en-us/entra/identity-platform/claims-validation).
- Give each Lambda role only the data, model, and logging permissions it needs. Keep deployment permissions separate from application runtime permissions.
- Restrict `iam:PassRole` to the specific execution roles and services required by deployment.
- Use HTTPS for API traffic and restrict CORS to configured frontend origins.
- Encrypt stored data with appropriate AWS encryption settings and define evidence retention and deletion policies.
- Keep credentials, Terraform state, sensitive variable files, access tokens, and raw production logs out of Git. Redact sensitive data before prompting or logging.
- Audit analysis requests, review decisions, and status changes without recording tokens or unnecessary incident payloads.
- Apply prompt boundaries, output validation, and retrieval access controls to reduce misuse of untrusted log and document content.

Authentication integration alone does not establish role-based access control or production security readiness.

<a id="data-model"></a>

## 🗄️ Data Model

The proposed model separates incident facts, generated analysis, engineer feedback, and knowledge metadata. These are logical entities; they may use separate DynamoDB tables or a single-table design depending on access patterns.

| Entity | Suggested key | Purpose and representative fields |
|---|---|---|
| `incidents` | `incident_id` | Source, source event ID, timestamps, affected resources, lifecycle status, observed severity, raw evidence reference |
| `analysis_results` | `incident_id` + `analysis_id` | Summary, proposed severity, probable cause, confidence rationale, evidence, recommendations, model/prompt versions, generation status |
| `user_feedback` | `analysis_id` + `feedback_id` | Incident ID, reviewer identity, decision, notes, review timestamp |
| `rag_documents` | `doc_id` + version | Title, document type, S3 URI, index/chunk references, access metadata, update timestamp |

Example incident record:

```json
{
  "incident_id": "INC-1042",
  "source": "CloudWatch",
  "source_event_id": "example-event-1042",
  "service": "RDS",
  "resource_id": "db-01",
  "status": "open",
  "observed_severity": "high",
  "created_at": "2026-10-01T16:42:01Z",
  "raw_evidence_ref": "s3://example-triagex-evidence/INC-1042/events.json"
}
```

Example proposed analysis record:

```json
{
  "analysis_id": "A-123",
  "incident_id": "INC-1042",
  "generation_status": "completed",
  "review_status": "pending",
  "summary": "Application database connections are failing during connection saturation.",
  "proposed_severity": "high",
  "probable_root_cause": "Connection pool exhaustion or leaked application connections",
  "confidence": {
    "label": "medium",
    "rationale": "Connection saturation supports the hypothesis; session-level evidence is still needed."
  },
  "evidence_refs": ["metric-db-connections", "log-connection-errors"],
  "document_refs": ["runbook-rds-connections:v1:chunk-03"],
  "recommendations": [
    "Inspect active sessions and application connection pool metrics.",
    "Confirm connection lifecycle behavior before selecting a remediation."
  ]
}
```

These records are illustrative. Model and prompt identifiers should be retained with each real analysis. A model's confidence assessment is not a calibrated probability. Large logs and document bodies should be stored outside the incident metadata record, with controlled references.

<a id="api-design"></a>

## 🌐 API Design

The following endpoints describe a **proposed contract**. Exact routes and response schemas must match the API Gateway and Lambda implementation.

| Method | Endpoint | Purpose |
|---|---|---|
| `GET` | `/incidents` | List authorized incidents with filters and pagination |
| `GET` | `/incidents/{id}` | Retrieve incident details and available analyses |
| `POST` | `/analyze` | Request analysis for an existing incident |
| `GET` | `/incidents/{id}/analyses/{analysis_id}` | Retrieve analysis results or processing state |
| `PATCH` | `/incidents/{id}/status` | Update the incident lifecycle status |
| `POST` | `/incidents/{id}/reviews` | Record feedback for a specific analysis |

Example analysis request:

```http
POST /analyze
Authorization: Bearer <access-token-for-triagex-api>
Content-Type: application/json

{
  "incident_id": "INC-1042"
}
```

The original flow returns a generated summary synchronously. For a longer-running RAG workflow, the proposed asynchronous response is `202 Accepted`:

```json
{
  "analysis_id": "A-123",
  "incident_id": "INC-1042",
  "status": "queued"
}
```

The client would then retrieve the analysis until it reaches `completed` or `failed`. This asynchronous pattern requires an implemented background execution mechanism; returning `202` alone does not provide one.

Example engineer review payload:

```json
{
  "analysis_id": "A-123",
  "decision": "request_changes",
  "notes": "Check connection pool metrics before accepting the diagnosis."
}
```

Reviewer identity must come from the validated token, not a client-supplied user ID. Target API behavior includes input validation, pagination, throttling, and consistent errors for invalid requests (`400`), missing or invalid authentication (`401`), insufficient authorization (`403`), and missing resources (`404`).

<a id="example-incident"></a>

## 🧪 Example Incident

**Synthetic scenario:** An application's database requests begin failing. This example illustrates the intended report format; it is not a measured result from a deployed system.

```text
Application logs: repeated database connection timeout errors
Database connections: 100
Configured connection limit in this scenario: 100
Application request errors: increasing
Affected resource: db-01
```

| Report field | Illustrative finding |
|---|---|
| Technical summary | New database connections are failing while connection usage is at the configured limit. |
| Executive summary | A database connectivity issue may be preventing users from completing application requests. |
| Suggested severity | High; confirm affected users, duration, and service objectives before final classification. |
| Probable cause | Connection saturation, potentially caused by pool exhaustion or connections not being released. |
| Confidence | Medium; saturation is observed, but its underlying cause remains unverified. |
| Supporting evidence | Connection-count metrics and application connection errors within the same incident window. |
| Missing evidence | Active-session details, pool settings, recent deployments, and network connectivity checks. |

Recommended investigation:

1. Verify the incident window and confirm the database's effective connection limit.
2. Inspect active sessions and application connection pool metrics.
3. Check recent changes for connection leaks or increased concurrency.
4. Examine network connectivity and database health for alternative causes of timeouts.
5. Select a remediation after validating the cause, then monitor connection usage and request errors.

The engineer reviews the report before making changes. Suggested actions are investigation guidance; no tested AWS CLI remediation commands are implied.

<a id="project-structure"></a>

## 📁 Project Structure

The following is a **recommended layout** for the target architecture. It is not a verified inventory of existing repository files.

```text
TriageX/
├── frontend/
│   ├── public/
│   └── src/
│       ├── components/
│       ├── pages/                 # Login, dashboard, incident, analysis, review
│       ├── services/              # API and identity clients
│       ├── App.jsx
│       └── main.jsx
├── backend/
│   ├── lambda/
│   │   ├── ingest_logs/
│   │   ├── get_incidents/
│   │   ├── get_incident/
│   │   ├── analyze_incident/
│   │   ├── update_status/
│   │   └── submit_review/
│   ├── services/                 # DynamoDB, Bedrock, and RAG clients
│   ├── auth/
│   ├── prompts/
│   ├── utils/
│   └── tests/
├── terraform/
│   ├── modules/                  # IAM, Lambda, API, storage, events, monitoring
│   ├── main.tf
│   ├── variables.tf
│   └── outputs.tf
├── docs/
│   ├── triagex-visual-blueprint.png
│   └── triagex-backend-blueprint.png
├── docs/                        # Proposed additional documentation
├── TriageX.AI Frontend Blueprint.PNG
├── TriageX.AI Backend Blueprint.PNG
├── README.md
└── .gitignore
```

<a id="prerequisites"></a>

## 📋 Prerequisites

- An AWS account and selected deployment Region.
- A deployment identity with **appropriately scoped permissions** to provision the resources actually defined in Terraform: IAM roles/policies, Lambda, API Gateway, DynamoDB, CloudWatch, and any enabled S3, EventBridge, OpenSearch, or encryption resources. Scope actions and resource access wherever supported.
- AWS CLI configured with the deployment identity, preferably using temporary credentials or an assumed role.
- Access to the configured Amazon Bedrock model or inference profile in the selected Region. The original design uses Claude 3 Haiku; confirm its availability and lifecycle before deployment. Follow the current [Bedrock model access requirements](https://docs.aws.amazon.com/bedrock/latest/userguide/model-access.html), including applicable Anthropic first-use and one-time Marketplace setup requirements. Runtime roles need model invocation permissions, not general subscription-management permissions.
- A Microsoft Entra ID tenant and permissions to configure the required app registrations, API scopes, and consent.
- [Terraform CLI](https://developer.hashicorp.com/terraform/install) compatible with the repository's version constraints.
- Node.js and npm compatible with the frontend's declared dependencies; use a supported Node.js release.
- Python compatible with the configured Lambda runtime and backend dependencies.
- Git and access to the project repository.

For the RAG extension, also prepare approved knowledge documents, an embedding model, and the configured vector index. Review expected AWS costs before provisioning; serverless services and managed search can still incur charges.

<a id="deployment-guide"></a>

## 🚀 Deployment Guide

This guide preserves the original `terraform/` and `frontend/` deployment flow and expands the required configuration. Paths, environment names, packaging, and optional services must be aligned with the actual repository; these instructions have not been validated against a deployed environment.

### 1. Configure Microsoft Entra ID

1. Register the frontend as a Single-Page Application.
2. Configure its redirect URI, such as `http://localhost:5173` for local development, and the actual HTTPS frontend URL for a hosted environment.
3. Configure an API app registration or the existing equivalent. Expose the delegated scope used by TriageX and grant the SPA permission to request it, with tenant consent where required.
4. Record the tenant ID, SPA client ID, API audience, and requested API scope. Configure any required app roles and assignments.
5. Ensure backend validation and authorization use the same tenant, audience, and permissions.

### 2. Prepare AWS and Backend Configuration

1. Confirm the AWS CLI uses the intended deployment identity and Region:

   ```bash
   aws sts get-caller-identity
   aws configure get region
   ```

2. Confirm model availability and access for the configured Bedrock model or inference profile.
3. Review `terraform/variables.tf` and any supplied variable examples. Set the required Region, identity settings, model identifier, and application resource settings using the names actually declared there.
4. Prepare Lambda dependencies and deployment artifacts according to the repository's build process. The required JWT-validation library and Python dependencies must be included in the deployed package or layer.
5. Configure protected Terraform state storage for shared environments. Keep credentials, state files, plan files, and sensitive local variable values out of version control.

No Terraform variable names or Lambda packaging command are prescribed here because the underlying files were not available for verification.

### 3. Provision the AWS Infrastructure

From the repository root:

```bash
cd terraform
terraform init
terraform fmt -check
terraform validate
terraform plan -out=tfplan
```

Review the plan, including IAM permissions, resource changes, optional search resources, and estimated costs. Apply the reviewed plan:

```bash
terraform apply tfplan
terraform output
```

Record the API endpoint and other outputs defined by the configuration. Provision optional RAG resources only when their corresponding application integration is ready.

### 4. Configure and Run the Frontend

From the repository root, navigate to `frontend/` and install dependencies:

```bash
cd frontend
npm install
```

Use the repository's environment template if provided. Otherwise, the following is an **illustrative** `.env.local` configuration; adapt the names to those consumed by the frontend:

```env
VITE_ENTRA_CLIENT_ID=<spa-client-id>
VITE_ENTRA_TENANT_ID=<tenant-id>
VITE_API_BASE_URL=<api-gateway-base-url>
VITE_ENTRA_API_SCOPE=api://<api-app-client-id>/<delegated-scope-name>
```

All `VITE_` values are exposed to the browser bundle. They must contain only public configuration, never AWS credentials or client secrets.

Start local development:

```bash
npm run dev
```

Confirm that the frontend requests the configured API scope and that the redirect URI and API CORS settings match the actual development origin. Use the repository's build and hosting procedure for a deployed frontend.

### 5. Configure Implemented Ingestion and RAG Integrations

- For the core summary flow, load a sanitized sample incident through the implemented data-loading mechanism.
- If event ingestion is implemented, configure its supported event rules or log subscriptions and verify normalization and duplicate handling.
- If RAG is implemented, upload approved documents, run the indexing process, and confirm that retrieval returns inspectable source citations.
- Configure CloudWatch log retention, error monitoring, and any available alarms.

### 6. Validate the Deployed Flow

Confirm that a signed-in user can retrieve an authorized sample incident and request analysis. Check that missing or invalid tokens are rejected and that unauthorized users cannot access protected operations or incident data.

Inspect the generated report against the sample evidence. For implemented extensions, also verify citations, analysis failure handling, and review persistence. Retain deployment and validation evidence before changing feature status to implemented or tested.

<a id="roadmap"></a>

## 🛣️ Roadmap

The phases below describe intended delivery milestones, not completed work. Security and observability apply throughout development.

| Phase | Focus | Completion evidence |
|---|---|---|
| 1 | Core authenticated summary flow | Reproducible Terraform deployment, token validation, sample incident retrieval, Bedrock summary |
| 2 | Event ingestion and structured analysis | Supported source adapters, normalization, deduplication, validated report schema, failure handling |
| 3 | RAG | Document ingestion, compatible embeddings, retrieval access controls, source citations, retrieval evaluation |
| 4 | Incident UI and human review | Incident views, analysis history, review decisions tied to analysis versions |
| 5 | Evaluation and operational hardening | Curated incident dataset, groundedness and severity checks, authorization tests, metrics and alarms |

Potential future enhancements include incident correlation, historical comparisons, trend analysis, Jira or ServiceNow ticket creation, Slack or Microsoft Teams integrations, and additional cloud providers. Controlled remediation automation would require a separate implementation with explicit authorization, action validation, auditability, and rollback planning.

<a id="project-status"></a>

## 🚧 Project Status

**TriageX.AI is under active development as an educational and portfolio engineering project.**

The original README describes a React/Entra ID frontend and a Lambda-based DynamoDB-to-Bedrock summary flow managed with Terraform. This README expands that foundation into the intended incident triage architecture.

Event-driven ingestion, RAG, structured analysis, and engineer review remain planned capabilities unless their implementation and validation are demonstrated in the repository. API contracts, data records, directory layout, and incident output shown here are design examples. Deployment validation, production readiness, calibrated confidence scores, tested remediation commands, and measured MTTR improvements are not established by this documentation.

As development progresses, update capability status with links to the relevant implementation, reproducible setup steps, and validation results.

<a id="disclaimer"></a>

## ⚠️ Disclaimer

TriageX provides AI-assisted investigation and remediation guidance. Generated findings may be incomplete or incorrect and must be checked against source evidence by qualified engineers before production changes are made.

Engineer approval records a review decision; it does not authorize or execute infrastructure changes. Use synthetic or appropriately sanitized incident data for demonstrations, and follow organizational policies for handling operational and security information.
