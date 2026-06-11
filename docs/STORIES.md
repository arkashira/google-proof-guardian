# STORIES.md  

## Project: google-proof-guardian  
**Goal:** Deliver a Minimum Viable Product (MVP) that automatically validates a website’s compliance with Google’s social‑proof and SEO policies, updates social‑proof widgets, and provides clear reporting to prevent penalties and boost conversion rates.

---  

## Epics & Backlog  

| Epic | Description | Priority (MVP) |
|------|-------------|----------------|
| **E1 – Compliance Engine** | Detect and flag violations of Google’s social‑proof and SEO guidelines in real‑time. | ✅ |
| **E2 – Automated Social‑Proof Updater** | Pull verified testimonials, reviews, and ratings from configured sources and inject them into the site safely. | ✅ |
| **E3 – Reporting & Alerting Dashboard** | Show site owners a concise compliance score, change history, and actionable alerts. | ✅ |
| **E4 – Integration & Deployment Kit** | Provide easy‑to‑use SDK / npm / Docker package and CI/CD hooks for quick adoption. | ✅ |
| **E5 – Security & Data Privacy** | Ensure all external data is sanitized, stored securely, and GDPR‑compliant. | ✅ |
| **E6 – Extensibility & Plug‑in System** | Allow third‑party data sources and custom rule sets to be added without code changes. | ❌ |
| **E7 – Analytics & Conversion Impact** | Correlate compliance status with conversion metrics and generate ROI reports. | ❌ |

> **MVP Scope** – All stories in Epics E1‑E5 are required for the first release. Epics E6‑E7 are slated for Phase 2.

---  

## User Story Cards  

### Epic E1 – Compliance Engine  

| # | Story | Acceptance Criteria |
|---|-------|----------------------|
| **E1‑US1** | **As a website owner, I want the system to scan my pages for prohibited “fake review” markup, so that I can avoid Google penalties.** | - Scan every HTML page reachable from the root URL.<br>- Detect schema.org `Review` markup with `author` or `rating` values that are not sourced from a verified provider.<br>- Return a list of offending elements with line numbers.<br>- Provide a “pass/fail” compliance badge. |
| **E1‑US2** | **As an SEO manager, I want a daily compliance cron job, so that any new content is automatically validated.** | - Scheduler runs once every 24 h (configurable).<br>- Generates a compliance report and stores it in `/reports/YYYYMMDD.json`.<br>- Sends an email (or webhook) if any violations are found. |
| **E1‑US3** | **As a compliance auditor, I want a rule‑engine UI to toggle specific Google policy checks, so that I can adapt to policy updates.** | - UI lists all built‑in rules (e.g., “Hidden text”, “Manipulated schema”).<br>- Each rule can be enabled/disabled.<br>- Changes persist across restarts.<br>- Disabled rules are omitted from scans. |
| **E1‑US4** | **As a developer, I want clear error codes and messages for each violation, so that I can fix issues programmatically.** | - Each violation includes a machine‑readable code (e.g., `GPG-001`).<br>- Message explains the rule and suggested fix.<br>- JSON schema for the output is documented. |

### Epic E2 – Automated Social‑Proof Updater  

| # | Story | Acceptance Criteria |
|---|-------|----------------------|
| **E2‑US1** | **As a content manager, I want to connect my Google Business Profile, so that reviews are fetched automatically.** | - OAuth flow to authorize Google Business Profile.<br>- Store token securely (encrypted at rest).<br>- Pull latest 10 reviews, cache for 12 h.<br>- Mark each review with `source: "google"` and a verification timestamp. |
| **E2‑US2** | **As a site editor, I want the system to replace placeholder testimonial blocks with verified reviews, so that visitors see authentic proof.** | - Detect HTML elements with class `gpg‑placeholder`.<br>- Replace inner HTML with a templated review card (author, rating, snippet, date).<br>- Preserve original styling via CSS variables.<br>- Fallback to original content if no verified review is available. |
| **E2‑US3** | **As a marketer, I want to schedule periodic refreshes of social‑proof widgets, so that the content never becomes stale.** | - Configurable refresh interval (default 24 h).<br>- On refresh, fetch new data, re‑render widgets, and purge old cache entries.<br>- Log each refresh with timestamp and number of items updated. |
| **E2‑US4** | **As a QA engineer, I want a preview mode that shows how updated widgets will appear before publishing, so that I can validate layout.** | - `/preview?url=<target>` endpoint renders the page with injected widgets without persisting changes.<br>- Preview respects the same sanitisation pipeline as production. |

### Epic E3 – Reporting & Alerting Dashboard  

| # | Story | Acceptance Criteria |
|---|-------|----------------------|
| **E3‑US1** | **As a site owner, I want a dashboard showing my overall compliance score, so that I can quickly assess risk.** | - Score = (total passed checks / total checks) * 100.<br>- Visual gauge (green/yellow/red).<br>- Last scan timestamp displayed. |
| **E3‑US2** | **As an operations lead, I want to receive Slack/webhook alerts when a high‑severity violation is detected, so that I can act immediately.** | - Severity levels: `critical`, `warning`, `info`.<br>- Configurable webhook URL.<br>- Alert payload includes page URL, rule code, and remediation link. |
| **E3‑US3** | **As a compliance officer, I want historical trend charts of violations over time, so that I can demonstrate improvement to stakeholders.** | - Store each scan result in a time‑series DB (e.g., SQLite or PostgreSQL).<br>- Chart shows count of violations per rule per week for the past 12 weeks.<br>- Export CSV button. |
| **E3‑US4** | **As a developer, I want the dashboard to be accessible via a single‑page app (SPA) that can be embedded in any admin portal.** | - Built with a lightweight framework (e.g., React or Svelte).<br>- Exposes a `<gpg-dashboard>` custom element.<br>- No external authentication required beyond the existing site session. |

### Epic E4 – Integration & Deployment Kit  

| # | Story | Acceptance Criteria |
|---|-------|----------------------|
| **E4‑US1** | **As a DevOps engineer, I want a Docker image that runs the scanner as a side‑car, so that I can add it to existing Kubernetes pods.** | - Dockerfile builds a minimal image (Alpine + Python/Node).<br>- Image size < 120 MB.<br>- Exposes port `8080` for API and `/healthz` endpoint.<br>- Tags: `latest` and `vX.Y.Z`. |
| **E4‑US2** | **As a front‑end developer, I want an npm package `google-proof-guardian` that provides a `initGuardian()` function, so that I can drop it into my build.** | - Published to npm (private registry for now).<br>- `initGuardian({apiKey, siteId})` injects the client script and registers the preview mode.<br>- Typescript definitions included. |
| **E4‑US3** | **As a CI/CD manager, I want a GitHub Action that runs the compliance scan on every PR, so that violations are caught early.** | - Action name: `google-proof-guardian/scan`. <br>- Accepts `site-url` input.<br>- Fails the workflow if any `critical` violations are found.<br>- Posts a summary comment on the PR. |
| **E4‑US4** | **As a product owner, I want clear documentation (README, quick‑start guide, API reference), so that new customers can adopt the tool without assistance.** | - README contains installation, configuration, and troubleshooting sections.<br>- API reference generated via Swagger/OpenAPI.<br>- Docs versioned with each release tag. |

### Epic E5 – Security & Data Privacy  

| # | Story | Acceptance Criteria |
|---|-------|----------------------|
| **E5‑US1** | **As a security analyst, I want all external data (reviews, ratings) to be sanitized against XSS, so that injected scripts cannot run on my site.** | - Use a proven sanitisation library (e.g., DOMPurify).<br>- Run sanitisation on every field before injection.<br>- Unit tests covering common XSS vectors. |
| **E5‑US2** | **As a GDPR officer, I need the ability to delete all stored personal data on request, so that we stay compliant with data‑subject rights.** | - Provide an API endpoint `DELETE /data/:siteId` that removes cached reviews, logs, and tokens.<br>- Endpoint requires admin authentication.<br>- Returns a confirmation JSON with deletion timestamp. |
| **E5‑US3** | **As a system admin, I want all secrets (API keys, OAuth tokens) encrypted at rest and never logged, so that breach impact is minimized.** | - Secrets stored using `dotenv` + `libsodium` encryption.<br>- Logging middleware redacts any fields matching `*key*`, `*token*`.<br>- Audit log records secret access attempts. |

---  

## Ordering for MVP Release  

1. **E1‑US1, E1‑US2, E1‑US4** – Core compliance scanning & reporting.  
2. **E2‑US1, E2‑US2** – First data source (Google Business) and widget injection.  
3. **E3‑US1, E3‑US2** – Dashboard overview + critical alerts.  
4. **E4‑US1, E4‑US2** – Deployment artifacts (Docker, npm).  
5. **E5‑US1, E5‑US2** – Security hardening.  
6. **Remaining stories** (E1‑US3, E2‑US3‑US4, E3‑US3‑US4, E4‑US3‑US4, E5‑US3) – polish, extensibility, CI/CD, documentation.  

---  

*All stories are written to be **shippable**: they include clear acceptance criteria, are testable, and map to concrete code artifacts.*
