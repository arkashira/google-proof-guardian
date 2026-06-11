# 📄 Product Requirements Document (PRD)  
**Project:** google-proof-guardian  
**Owner:** Senior Product/Engineering Lead – AxentX  
**Date:** 2026‑06‑11  
**Version:** 1.0  

---  

## 1. Problem Statement  

Website owners who display social‑proof elements (reviews, ratings, testimonials, user counts, etc.) risk violating Google’s **Search Quality Rating Guidelines** and **Google Ads Policies** when those signals are inaccurate, outdated, or presented in a manipulative way.  

* **Financial risk:** Non‑compliance can trigger manual penalties ranging from **$500‑$5,000** (≈ THB 17.5k‑175k) per violation.  
* **Conversion loss:** Untrusted or stale social proof reduces conversion rates by **10‑20 %** on average.  
* **Operational overhead:** Maintaining up‑to‑date, compliant proof manually consumes engineering and marketing resources.  

There is a validated market need for a **plug‑and‑play service** that continuously audits, updates, and validates social‑proof content to keep sites both **Google‑compliant** and **conversion‑optimized**.

---  

## 2. Target Users  

| Segment | Persona | Primary Pain Point | Willingness to Pay |
|---------|---------|--------------------|--------------------|
| **E‑commerce SMEs** | “Mia, Founder” – runs a $2M/year shop on Shopify/WooCommerce | Fear of Google penalties; limited dev resources | $49‑$199/mo for compliance SaaS |
| **Digital Agencies** | “Luis, SEO Lead” – manages 30 client sites | Need a repeatable, audit‑ready workflow | $299‑$999/mo per agency bundle |
| **Enterprise Brands** | “Anita, VP of Marketing” – global brand with 200+ domains | Governance & audit trails for legal/compliance teams | Custom contract (>$5k/mo) |
| **Web‑dev freelancers** | “Sam, Full‑stack dev” – builds sites for clients | Quick, code‑light integration | $19‑$49/mo (lite) |

---  

## 3. Goals & Success Metrics  

| Goal | Success Metric | Target (12 mo) |
|------|----------------|----------------|
| **Compliance Assurance** | % of monitored sites passing Google’s social‑proof audit | ≥ 95 % |
| **Conversion uplift** | Avg. lift in conversion rate after activation (A/B) | 12 % |
| **Time saved** | Hours per month saved on manual social‑proof updates | 80 h |
| **Revenue** | Monthly Recurring Revenue (MRR) | $150 k |
| **Adoption** | Active paying sites | 1,200 |
| **Reliability** | System uptime (SLA) | 99.9 % |

---  

## 4. Scope  

### 4.1 In‑Scope (MVP)  

| Feature | Description | Priority |
|---------|-------------|----------|
| **Automated Compliance Engine** | Crawl configured pages, parse social‑proof snippets, validate against Google’s policy JSON (e.g., no fabricated counts, no hidden schema). | ★★★★★ |
| **Real‑time Social‑Proof Updater** | Pull verified data from supported sources (Google My Business, Trustpilot, Yelp, internal DB) and inject via CDN‑edge or JS widget. | ★★★★ |
| **Penalty Alerting** | Email/SMS webhook when a violation is detected, with remediation steps. | ★★★★ |
| **Dashboard & Reporting** | Per‑site health score, audit log, compliance trend, conversion impact estimate. | ★★★ |
| **Integrations** | Plugins for WordPress, Shopify, Wix, and a generic JS SDK for custom sites. | ★★★ |
| **Role‑Based Access Control** | Admin, Analyst, Viewer permissions. | ★★ |
| **Data Privacy & GDPR** | Store only hashes of user‑generated proof; provide export/delete tools. | ★★ |
| **CI/CD Ready Deployment** | Docker image + Helm chart for Kubernetes; optional serverless (Vercel/Netlify) mode. | ★★ |

### 4.2 Out‑of‑Scope (Post‑MVP)  

| Feature | Reason |
|---------|--------|
| AI‑generated synthetic reviews | High legal risk, not core compliance. |
| Full‑fledged SEO audit (backlinks, schema beyond social proof) | Separate product line. |
| Multi‑language proof translation | Deferred to later releases after core engine stabilises. |
| On‑premise self‑hosted version | SaaS model is primary revenue driver; on‑prem will be enterprise‑only later. |
| Direct Google Ads API integration for penalty removal | Requires partnership; out of scope for MVP. |

---  

## 5. Detailed Requirements  

### 5.1 Functional Requirements  

1. **Site Registration**  
   - API endpoint `/api/v1/sites` (POST) to add a site with: domain, verification token, source credentials.  
   - Email verification + DNS TXT record for ownership proof.  

2. **Compliance Engine**  
   - Scheduler runs every 6 h per site.  
   - Ruleset expressed in JSON (easily extensible). Example rule: `{"type":"count","max":999999,"field":"reviewCount","source":"schema.org"}`.  
   - Detects: fabricated numbers, hidden schema, mismatched timestamps, duplicate content.  

3. **Social‑Proof Updater**  
   - Pulls latest data via OAuth2 from supported APIs.  
   - Normalises to a canonical schema (`proof.id, proof.type, proof.value, proof.source, proof.lastUpdated`).  
   - Updates via:  
     - **Edge injection** (Cloudflare Workers / Fastly Compute@Edge) for static sites.  
     - **JS widget** that replaces placeholders (`<div data-gpg-id="..."></div>`).  

4. **Alerting & Remediation**  
   - On violation: send templated email + optional Slack webhook.  
   - Provide actionable diff and one‑click “Apply Fix” button (if auto‑fixable).  

5. **Dashboard**  
   - Site list with health badge (green/yellow/red).  
   - Timeline view of violations & fixes.  
   - Export CSV/JSON of audit logs.  

6. **Billing**  
   - Stripe integration; tiered plans based on number of monitored pages (≤ 50, ≤ 200, Unlimited).  

### 5.2 Non‑Functional Requirements  

| Requirement | Acceptance Criteria |
|-------------|---------------------|
| **Performance** | Crawl ≤ 200 ms per page; update latency ≤ 5 s after source change. |
| **Scalability** | Support 10 k concurrent sites; auto‑scale workers via Kubernetes HPA. |
| **Security** | All external API calls over TLS 1.3; data at rest encrypted with AES‑256. |
| **Observability** | Export metrics to Prometheus; logs to Loki; alerts via Grafana. |
| **Compliance** | GDPR‑ready data handling; SOC‑2 Type II audit ready by Q4 2026. |
| **Reliability** | 99.9 % SLA; automated canary deployments with rollback. |
| **Usability** | Onboarding flow ≤ 5 min; documentation with code samples for each integration. |

---  

## 6. Assumptions  

* Google’s social‑proof policy will remain stable for the next 12 months.  
* Third‑party review platforms provide stable, documented APIs (OAuth2).  
* Customers will have at least one DNS‑editable record for site verification.  

---  

## 7. Risks & Mitigations  

| Risk | Impact | Mitigation |
|------|--------|------------|
| **Policy change** – Google tightens rules mid‑release. | High – could invalidate engine rules. | Design rules engine to be data‑driven; maintain a policy‑update feed. |
| **API rate limits** from review platforms. | Medium – could delay updates. | Cache results for 15 min; implement exponential back‑off; negotiate enterprise API keys for high‑volume customers. |
| **False positives** causing unnecessary alerts. | Medium – erodes trust. | Include a “confidence score” and allow manual override; pilot with 5 beta customers before full launch. |
| **Data privacy breaches** (e.g., storing raw reviews). | High – legal & reputational. | Store only hashed identifiers; never persist raw user‑generated content unless explicitly consented. |
| **Integration friction** for non‑standard CMS. | Low‑Medium – could limit adoption. | Provide a generic JS SDK and clear docs; open‑source sample adapters. |

---  

## 8. Milestones & Timeline  

| Milestone | Deliverable | Owner | Target Date |
|-----------|-------------|-------|-------------|
| **Kickoff & Architecture** | Architecture diagram, tech‑stack decision (Node 18 + TypeScript, PostgreSQL, Redis, Cloudflare Workers) | Lead Architect | 2026‑06‑21 |
| **Compliance Engine Prototype** | Rule engine + basic crawler for 5 test sites | Backend Team | 2026‑07‑15 |
| **Social‑Proof Updater (JS SDK)** | NPM package, demo page | Frontend Team | 2026‑08‑01 |
| **Dashboard MVP** | React + Ant Design UI, auth, site list | Full‑Stack Team | 2026‑09‑01 |
| **Beta Launch** | 10 pilot customers, monitoring, feedback loop | PM/Customer Success | 2026‑10‑01 |
| **Public SaaS Release** | Billing, docs, CI/CD pipeline, SLA | All Teams | 2026‑12‑01 |
| **Post‑Launch Optimisation** | Edge‑only injection, additional integrations (Magento, BigCommerce) | Engineering | 2027‑02‑01 |

---  

## 9. Acceptance Criteria (MVP)  

1. **Compliance Engine** correctly flags at least 95 % of known policy violations in a curated test suite.  
2. **Updater** can refresh social‑proof data on a live WordPress site within 5 seconds of source change.  
3. **Dashboard** shows health badge and audit log for every registered site.  
4. **Alerting** delivers email within 2 minutes of detection, with a clear remediation step.  
5. **Billing** processes a test subscription end‑to‑end without errors.  
6. **Uptime** ≥ 99.9 % during the 30‑day beta period.  

---  

## 10. Appendices  

### A. Glossary  

* **Social Proof** – User‑generated signals (reviews, ratings, testimonials, user counts) displayed on a website.  
* **Compliance Engine** – Automated rule‑based scanner that validates social‑proof against Google policies.  
* **Edge Injection** – Modifying HTML at the CDN edge before it reaches the browser.  

### B. References  

1. Google Search Quality Rating Guidelines – Section “User‑Generated Content”.  
2. Google Ads Policies – “Misleading Content”.  
3. OWASP Top 10 – Secure Coding Practices.  

---  

*Prepared by the AxentX product team. This document is the authoritative source for the development, testing, and launch of **google-proof-guardian**.*
