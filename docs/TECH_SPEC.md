# TECH_SPEC.md
**Project:** google-proof-guardian  
**Owner:** AxentX – Autonomous AI‑Workforce  
**Version:** 1.0.0 (initial release)  
**Last Updated:** 2026‑06‑11  

---  

## 1. Overview  

google‑proof‑guardian is a SaaS service that continuously monitors a website’s “social‑proof” elements (reviews, ratings, testimonials, user‑generated content) and automatically updates them to remain **compliant with Google’s Search‑Quality and Structured‑Data policies**.  

* **Compliance Engine** – parses pages, validates schema.org markup, detects prohibited patterns (e.g., hidden text, manipulated review counts).  
* **Social‑Proof Updater** – fetches verified user‑generated content from configured sources (e.g., Trustpilot, Google My Business, internal DB) and injects it into the site via API or static‑site generation hooks.  
* **Penalty‑Avoidance Alerts** – real‑time webhook notifications and a dashboard that surface compliance violations before Google can penalize the site.  

The service is delivered as a **multi‑tenant, containerised micro‑service** that can be self‑hosted or run on AxentX’s managed platform.

---  

## 2. Architecture Diagram  

```
+-------------------+          +-------------------+          +-------------------+
|   Customer Site   | <--API--> |   Guardian API   | <--gRPC--> |   Compliance Svc |
| (static or CMS)  |          | (FastAPI / Nginx) |          | (Python)          |
+-------------------+          +-------------------+          +-------------------+
          |                               |                               |
          |                               |                               |
          v                               v                               v
+-------------------+          +-------------------+          +-------------------+
|   Update Engine   | <--REST--> |   Scheduler (Celery) | <--Redis--> |   Task Queue    |
| (Node/TS)         |          | (Python)               |          | (Redis)         |
+-------------------+          +-------------------+          +-------------------+
          |                               |
          |                               |
          v                               v
+-------------------+          +-------------------+
|   Data Store      | <--SQL-->|   Audit Log (ELK) |
| (PostgreSQL)      |          | (Elastic)         |
+-------------------+          +-------------------+

```

* **Guardian API** – public entry point (HTTPS, JWT auth).  
* **Compliance Service** – pure‑Python library that runs the policy engine (uses `vLLM` for LLM‑assisted rule interpretation).  
* **Update Engine** – Node.js/TypeScript worker that rewrites HTML/JSON files or calls CMS APIs (WordPress REST, Shopify GraphQL, etc.).  
* **Scheduler** – Celery beat + workers that orchestrate periodic scans (default: every 6 h).  
* **Data Store** – PostgreSQL 15 (JSONB for flexible schema).  
* **Audit Log** – ElasticSearch + Kibana for compliance audit trails.  

---  

## 3. Component Details  

### 3.1 Guardian API (FastAPI)  

| Aspect | Detail |
|--------|--------|
| **Framework** | FastAPI 0.115 (Python 3.12) |
| **Auth** | OAuth2 Client‑Credentials flow + JWT signed with RS256 |
| **Endpoints** | `POST /scan` – trigger immediate scan (tenant‑scoped) <br> `GET /status/{job_id}` – poll job status <br> `GET /dashboard` – UI data (metrics, violations) |
| **Rate Limiting** | 60 req/min per tenant (Redis‑based token bucket) |
| **OpenAPI** | Auto‑generated, hosted at `/docs` |

### 3.2 Compliance Service  

* **Rule Engine** – a collection of 120+ JSON‑encoded rules derived from Google’s “Search‑Quality Evaluator Guidelines” and “Structured Data Testing Tool”.  
* **LLM‑Assist** – `vLLM` (v0.5) runs a distilled Llama‑3‑8B model to interpret ambiguous markup and suggest remediation.  
* **Output** – `ComplianceReport` (JSON) containing: <br> `passed: bool`, `violations: [{code, description, elementXPath, severity}]`, `remediation: [{action, snippet}]`.

### 3.3 Update Engine (Node.js)  

| Item | Spec |
|------|------|
| **Runtime** | Node 20 LTS |
| **Language** | TypeScript 5 |
| **Core libs** | `axios` (HTTP), `cheerio` (HTML parsing), `graphql-request` (CMS APIs) |
| **Plugins** | `wordpress-rest`, `shopify-graphql`, `static-site-hook` (writes to `_site` folder) |
| **Idempotency** | All updates are versioned; previous version stored in PostgreSQL `updates` table for rollback. |
| **Safety** | Dry‑run mode (`X-Guardian-DryRun: true`) returns diff without applying changes. |

### 3.4 Scheduler & Task Queue  

* **Celery 5** with **Redis 7** broker.  
* Periodic tasks: `scan_all_tenants`, `purge_old_logs`, `refresh_llm_cache`.  
* Each scan spawns a **Celery chain**: `fetch_pages → run_compliance → generate_report → apply_updates`.  

### 3.5 Data Model (PostgreSQL)  

| Table | Primary Key | Important Columns |
|-------|-------------|-------------------|
| `tenants` | `tenant_id` (uuid) | `name`, `api_key`, `plan`, `created_at` |
| `sites` | `site_id` (uuid) | `tenant_id`, `base_url`, `cms_type`, `last_scan_at` |
| `pages` | `page_id` (uuid) | `site_id`, `url`, `html_hash`, `last_checked_at` |
| `reports` | `report_id` (uuid) | `site_id`, `status`, `generated_at`, `json_blob` |
| `updates` | `update_id` (uuid) | `page_id`, `diff_blob`, `applied_at`, `rollback_id` |
| `audit_events` | `event_id` (bigserial) | `tenant_id`, `event_type`, `payload`, `timestamp` |

JSONB columns (`json_blob`, `diff_blob`) store LLM‑generated suggestions and diffs.

---  

## 4. Key APIs / Interfaces  

### 4.1 Public REST API (Guardian API)

```http
POST /v1/scan
Authorization: Bearer <jwt>
Content-Type: application/json

{
  "site_id": "c1a2b3d4‑e5f6‑7890‑abcd‑1234567890ef",
  "options": {
    "dry_run": false,
    "force": false
  }
}
```

**Response (202 Accepted)**  

```json
{
  "job_id": "9f8e7d6c‑5b4a‑3210‑fedc‑ba9876543210",
  "status_url": "/v1/status/9f8e7d6c‑5b4a‑3210‑fedc‑ba9876543210"
}
```

### 4.2 Internal gRPC – Compliance Service  

```proto
service ComplianceEngine {
  rpc Evaluate (EvaluateRequest) returns (ComplianceReport);
}

message EvaluateRequest {
  string html = 1;
  repeated string url_path = 2;
}

message Violation {
  string code = 1;
  string description = 2;
  string xpath = 3;
  string severity = 4; // LOW, MEDIUM, HIGH
}

message Remediation {
  string action = 1; // e.g., "replace", "remove", "add"
  string snippet = 2;
}

message ComplianceReport {
  bool passed = 1;
  repeated Violation violations = 2;
  repeated Remediation remediation = 3;
}
```

### 4.3 CMS Connectors (Node plugins)  

* **WordPress** – uses `/wp-json/wp/v2/posts/{id}` PATCH with `meta._guardian_compliance = true`.  
* **Shopify** – GraphQL mutation `productUpdate` to set `metafield` `guardianCompliance`.  

All connectors expose a unified **`applyDiff(diff: Diff): Promise<Result>`** interface.

---  

## 5. Technology Stack  

| Layer | Technology | Version | Rationale |
|-------|------------|---------|-----------|
| API | FastAPI | 0.115 | High performance, automatic OpenAPI, async support |
| Auth | PyJWT + OAuth2 | – | Industry‑standard token flow |
| LLM | vLLM (distilled Llama‑3‑8B) | 0.5 | Low‑latency inference, GPU‑optimised, open‑source |
| Scheduler | Celery + Redis | 5 / 7 | Proven distributed task queue |
| Worker | Node.js | 20 LTS | Rich ecosystem for HTML/CMS manipulation |
| DB | PostgreSQL | 15 | Strong ACID, JSONB, native UUID |
| Search / Audit | ElasticSearch + Kibana | 8.13 | Full‑text search on compliance logs |
| Containerisation | Docker + Docker‑Compose | – | Reproducible dev/prod environments |
| Orchestration | Kubernetes (Helm chart) | 1.30 | Horizontal scaling, auto‑heal |
| CI/CD | GitHub Actions | – | Automated lint, test, build, push to registry |
| Monitoring | Prometheus + Grafana | – | Metrics on scan latency, error rates |
| Logging | Structured JSON (Python `structlog`, Node `pino`) | – | Centralised log aggregation |

---  

## 6. Dependencies  

| Dependency | License | Purpose |
|------------|---------|---------|
| `fastapi`, `uvicorn` | MIT | API server |
| `pydantic` | MIT | Data validation |
| `vllm` | Apache‑2.0 | LLM inference |
| `celery`, `redis` | BSD‑3 | Task queue |
| `psycopg2-binary` | LGPL‑3 | PostgreSQL driver |
| `elasticsearch` | Apache‑2.0 | Audit log storage |
| `axios`, `cheerio`, `typescript` | MIT | Update engine |
| `graphql-request` | MIT | Shopify connector |
| `docker`, `k8s` | Apache‑2.0 | Deployment |
| `prometheus-client`, `grafana` | Apache‑2.0 | Monitoring |

All dependencies are vetted for compatibility with AxentX’s security policy (no known CVEs > 7 days).

---  

## 7. Deployment & Operations  

### 7.1 Container Images  

| Service | Dockerfile | Registry |
|---------|------------|----------|
| guardian-api | `Dockerfile.api` (python:3.12‑slim) | `ghcr.io/google-proof-guardian/api:1.0.0` |
| compliance-svc | `Dockerfile.llm` (nvidia/cuda:12.4‑runtime) | `ghcr.io/google-proof-guardian/compliance:1.0.0` |
| update-engine | `Dockerfile.worker` (node:20‑alpine) | `ghcr.io/google-proof-guardian/worker:1.0.0` |
| scheduler | `Dockerfile.celery` (python:3.12‑slim) | `ghcr.io/google-proof-guardian/scheduler:1.0.0` |
| postgres | official `postgres:15-alpine` | — |
| redis | official `redis:7-alpine` | — |
| elasticsearch | official `elasticsearch:8.13` | — |

All images are built with **SBOM** generation (`syft`) and signed with **cosign**.

### 7.2 Helm Chart (chart/google-proof-guardian)  

* `values.yaml` includes:  
  * `replicaCount.api: 2`  
  * `replicaCount.worker: 3`  
  * `resources` (CPU/Memory limits)  
  * `autoscaling.enabled: true` (HPA based on CPU > 70 %)  
  * `ingress.enabled: true` (TLS via cert‑manager)  

Deploy with:  

```bash
helm repo add axentx https://helm.axentx.io
helm install gpg axentx/google-proof-guardian -n gpg-prod --create-namespace
```

### 7.3 CI/CD Pipeline (GitHub Actions)

1. **Lint** – `ruff` (Python) + `eslint` (TS).  
2. **Unit Tests** – `pytest` (coverage ≥ 90 %) + `jest`.  
3. **Integration Tests** – spin up a temporary k8s namespace, run end‑to‑end scan against a mock site.  
4. **Build & Push** – multi‑arch (`linux/amd64,linux/arm64`) images.  
5. **Helm Deploy** – on `main` merge to `prod` environment (requires manual approval).  

---  

## 8. Security & Compliance  

| Area | Controls |
|------|----------|
| **Authentication** | JWT signed with RSA‑4096; short‑lived access tokens (15 min). |
| **Authorization** | Tenant‑scoped RBAC; API keys limited to specific site IDs. |
| **Data Protection** | All at‑rest data encrypted (PostgreSQL `pgcrypto`). TLS 1.3 for all network traffic. |
| **Secret Management** | HashiCorp Vault integration for DB passwords, LLM model keys. |
| **Vulnerability Scanning** | Trivy scan on every image push; fail CI on CVE > 5. |
| **GDPR** | No personal data stored; only hashed page fingerprints and compliance metadata. |
| **Audit** | Immutable audit logs stored in Elasticsearch with write‑once index policy. |

---  

## 9. Monitoring & Alerting  

| Metric | Tool | Alert Threshold |
|--------|------|-----------------|
| `scan_duration_seconds` | Prometheus | > 30 s per page (warning) |
| `compliance_violations_total` | Prometheus | Spike > 20 % week‑over‑week |
| `worker_error_rate` | Grafana | > 0.5 % |
| `api_4xx_rate` | Prometheus | > 2 % |
| `llm_inference_latency_ms` | Prometheus | > 200 ms (critical) |

Alerts routed to PagerDuty and Slack `#gpg-ops`.

---  

## 10. Testing Strategy  

| Layer | Tool | Coverage Goal |
|-------|------|---------------|
| Unit (Python) | pytest + hypothesis | 90 % |
| Unit (TS) | jest + ts‑jest | 90 % |
| Integration | testcontainers (Postgres, Redis) + Docker‑compose | 80 % |
| End‑to‑End | Cypress (via headless Chrome) simulating a real site | 70 % |
| Load | k6 script – 500 concurrent scans | ≤ 2 s avg latency |
| Security | OWASP ZAP, Snyk | No high‑severity findings |

---  

## 11. Release Process  

1. **Feature Branch** → PR → Mandatory reviewers (PM, Security, QA).  
2. **Automated Checks** (lint, unit, integration) must pass.  
3. **Staging Deploy** – Helm upgrade to `gpg-staging`. Run smoke tests.  
4. **Canary Release** – 10 % of tenants switched to new version via feature flag.  
5. **Full Rollout** – after 48 h of no incidents, promote to `prod`.  
6. **Post‑Release** – monitor for 72 h, then tag `vX.Y.Z`.  

---  

## 12. Open Issues & Future Work  

| Issue | Owner | Target Milestone |
|-------|-------|------------------|
| Decide final CMS connector list (e.g., Wix, Squarespace) | Product Lead | v1.1 |
| Evaluate LLM‑on‑CPU fallback for low‑cost deployments | ML Engineer | v1.2 |
| Add support for **Google Rich Results Test** API integration | DevOps | v1.3 |
| Multi‑region deployment for EU GDPR compliance | Infra | v2.0 |

---  

**Prepared by:**  
Senior Product/Engineering Lead – AxentX  
*All specifications are aligned with the current repo state (initial commit d9c35f1) and the company’s validated need for automated Google‑compliance tooling.*
