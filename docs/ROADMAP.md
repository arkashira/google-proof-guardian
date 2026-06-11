# ROADMAP.md – google-proof-guardian
*Version: 0.1 – Draft (2026‑06‑11)*  

---  

## Vision  
Empower website owners to stay **Google‑compliant** while automatically showcasing **verified social proof** that drives conversions and protects against costly SEO penalties.

---  

## 📅 Milestones Overview  

| Milestone | Target Date | Scope | MVP‑Critical? |
|-----------|-------------|-------|---------------|
| **MVP – Launch Ready** | **2024‑09‑30** | Core compliance engine, automated proof updater, basic dashboard, CI/CD pipeline | **Yes** |
| **v1 – Enterprise Enablement** | 2025‑02‑15 | Multi‑site management, policy rule editor, analytics & reporting, SSO integration | No |
| **v2 – Ecosystem & AI‑Assist** | 2025‑08‑01 | AI‑driven proof generation, marketplace of proof widgets, plug‑in SDK, GDPR/CCPA extensions | No |

---  

## 🚀 MVP – Launch Ready *(Must‑have for launch)*  

| # | Feature | Description | Acceptance Criteria |
|---|---------|-------------|----------------------|
| 1 | **Compliance Engine** | Scrapes page content & schema.org markup, validates against Google’s “Social Proof” and “SEO” guidelines. | • Detects 100 % of known violation patterns (e.g., fake reviews, hidden markup).<br>• Returns a clear pass/fail with line‑number references. |
| 2 | **Automated Social‑Proof Updater** | Pulls verified reviews/ratings from supported APIs (Google My Business, Trustpilot, Yelp) and injects them into the site via a lightweight JS widget. | • Updates run on a configurable schedule (hourly/daily).<br>• No page‑render blocking; widget loads < 200 ms. |
| 3 | **Web Dashboard (Admin UI)** | Single‑page React (or Svelte) UI for site owners to view compliance status, schedule updates, and view recent changes. | • Auth via email/password (bcrypt).<br>• Dashboard loads < 2 s on 3G. |
| 4 | **CI/CD & Deployment Scripts** | Docker‑based container, Helm chart for K8s, GitHub Actions for lint, test, and release. | • One‑click `docker compose up -d` works locally.<br>• Production image published to Docker Hub on merge to `main`. |
| 5 | **Testing Suite** | Unit tests (Jest/PyTest), integration tests (Cypress), compliance‑rule regression tests. | • ≥ 80 % code coverage.<br>• All tests pass on CI. |
| 6 | **Documentation** | README, quick‑start guide, API reference (OpenAPI spec). | • New developer can spin up a sandbox in ≤ 30 min. |
| 7 | **Legal & Licensing** | MIT license, privacy policy template, GDPR compliance checklist for data handling. | • All third‑party SDKs have compatible licenses. |

### MVP Success Metrics  
- **Compliance Pass Rate** ≥ 98 % on a curated test set of 200 live sites.  
- **Conversion Lift** ≥ 10 % in pilot customers (measured via A/B test).  
- **Penalty Avoidance** documented for ≥ 5 early adopters within 3 months.  
- **Time‑to‑Deploy** ≤ 15 min from clone to running dashboard.

---  

## 🌟 v1 – Enterprise Enablement  

| Theme | Key Features | Target Release |
|-------|--------------|----------------|
| **Multi‑Site & Team Management** | • Manage unlimited domains from one account.<br>• Role‑based access (Admin, Editor, Viewer).<br>• Bulk policy scans. | Q1 2025 |
| **Policy Rule Editor** | • UI to add custom compliance rules (regex, JSON‑LD checks).<br>• Rule versioning & audit log. | Q1 2025 |
| **Analytics & Reporting** | • Daily compliance scorecards.<br>• Proof‑impact heatmaps (conversion vs. proof placement).<br>• Exportable CSV/PDF reports. | Q1 2025 |
| **SSO & Enterprise Auth** | • SAML 2.0 / OIDC integration.<br>• SCIM provisioning for large orgs. | Q2 2025 |
| **Extended Data Connectors** | • Additional review sources (Facebook, G2, Capterra).<br>• Webhook API for custom data feeds. | Q2 2025 |
| **Performance Optimizations** | • Edge‑cache invalidation via Cloudflare Workers.<br>• Lazy‑load fallback for low‑bandwidth users. | Q2 2025 |

### v1 Success Metrics  
- **NPS** ≥ 70 from enterprise pilots.  
- **Average Scan Time** ≤ 5 seconds per page.  
- **Retention** ≥ 80 % after 6 months.  

---  

## 🤖 v2 – Ecosystem & AI‑Assist  

| Theme | Key Features | Target Release |
|-------|--------------|----------------|
| **AI‑Generated Social Proof** | • LLM‑driven suggestion of review excerpts (human‑reviewed before publish).<br>• Sentiment‑aware placement recommendations. | Q3 2025 |
| **Marketplace & Plug‑in SDK** | • Public marketplace for custom widgets (carousel, video testimonials).<br>• SDK for third‑party developers (npm, PyPI). | Q3 2025 |
| **Privacy & Global Compliance** | • Built‑in GDPR, CCPA, LGPD consent management for proof data.<br>• Data‑locality selector per region. | Q4 2025 |
| **Adaptive SEO Optimizer** | • Real‑time schema.org micro‑data tuning based on Google Search Console feedback.<br>• Automated A/B testing of proof variants. | Q4 2025 |
| **Community & Support Portal** | • Self‑service knowledge base, ticketing system, Slack/Discord community bots. | Q4 2025 |

### v2 Success Metrics  
- **AI‑Proof Acceptance Rate** ≥ 90 % after manual review.  
- **Marketplace Revenue** ≥ $15k/month by end‑2025.  
- **Compliance Incident Rate** < 1 % across all active sites.  

---  

## 📦 Release Process  

1. **Feature Freeze** – 2 weeks before target date.  
2. **Beta Rollout** – Internal + 5 external pilot accounts.  
3. **Security Review** – OWASP Top 10 + static analysis (SonarQube).  
4. **Compliance Sign‑off** – Legal team validates data‑handling & licensing.  
5. **Production Deploy** – Blue‑green rollout with health‑checks.  

---  

## 📚 Dependencies & Tech Stack Decisions  

| Area | Current Choice | Rationale | Open Decision |
|------|----------------|-----------|---------------|
| Backend | **Node.js (NestJS)** or **Python (FastAPI)** – TBD | Both have strong async HTTP client support for external review APIs. | Choose by Q‑Oct 2024 |
| Frontend | **React + Vite** (or **SvelteKit**) | Component ecosystem for widget injection. | Finalize by Q‑Oct 2024 |
| Database | **PostgreSQL** + **Redis** (caching) | ACID compliance for audit logs; fast cache for scans. | Fixed |
| Containerization | Docker + Helm (K8s) | Aligns with existing AxentX CI/CD. | Fixed |
| LLM Provider (v2) | **vLLM** (open‑source) or **OpenAI** API | vLLM matches internal framework; cost‑effective for scaling. | Decision Q‑Jan 2025 |
| CI/CD | GitHub Actions + Dependabot | Already used across AxentX repos. | Fixed |

---  

## 📈 Growth & Validation Plan  

| Phase | Validation Method | Success Threshold |
|-------|-------------------|-------------------|
| MVP | Pilot with 10 SaaS sites (e‑commerce, SaaS, blogs). | ≥ 10 % conversion lift, 0 compliance penalties. |
| v1 | Enterprise beta (3‑5 mid‑size firms). | NPS ≥ 70, < 5 min scan per 100 pages. |
| v2 | Public marketplace launch. | $15k/month revenue, AI‑proof acceptance ≥ 90 %. |

---  

## 🛡️ Risk Mitigation  

| Risk | Impact | Mitigation |
|------|--------|------------|
| Google policy changes | High – could break compliance engine | Subscribe to Google’s policy RSS; modular rule engine for rapid updates. |
| Data‑privacy regulations | Medium – legal exposure | Built‑in consent layer; regular privacy audits. |
| Third‑party API rate limits | Medium | Cache responses, exponential back‑off, fallback to static proof pool. |
| Performance on high‑traffic sites | High | Edge‑caching, async widget loading, CDN integration. |

---  

## 📌 Next Steps (Immediate)  

1. **Tech‑stack lock** – decision by 2024‑10‑15.  
2. **Prototype compliance engine** – proof‑of‑concept using a subset of Google guidelines (2 weeks).  
3. **Create CI pipeline** – lint, unit tests, Docker build (by 2024‑10‑31).  
4. **Recruit pilot partners** – outreach to 5 existing AxentX customers (by 2024‑11‑15).  

---  

*Prepared by the Senior Product/Engineering Lead – AxentX*
