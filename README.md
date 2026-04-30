# Title2Token — V1 Operational Control Tower

Static HTML/CSS/JS portal system for managing tokenized real estate deals end-to-end.
No build step. Served via `python -m http.server 8080`.

> **Current session context:** Building 30 screens across 5 portals. Admin tower (10 screens) is the priority — 3 complete, 7 remaining. Then issuer (5), legal (5), backend (5), broker (4).

---

## Local Dev

```bash
cd title2token/admin/screens
python -m http.server 8080
# open http://localhost:8080/dashboard.html
```

---

## What This Is

A deal activation command center that answers:
- What is preventing this deal from reaching tokenization launch?
- Who owns that blocker?
- What will it cost?
- What approval/signoff is needed next?

---

## Portal Map (30 screens total)

### Admin Control Tower — `admin/screens/`
| File | Status | Description |
|---|---|---|
| `dashboard.html` | ✅ Built | Command Center — KPIs, today's actions, blockers, deal health cards |
| `pipeline.html` | ✅ Built | Deal Pipeline — stage bar, filter bar, full deal table |
| `deal-detail.html` | ✅ Built | Deal Detail — 7-tab view (Overview, Checklist, Docs, Costs, Responsibility, Activity, Timeline) |
| `blocker-board.html` | 🔲 Next | Cross-deal blocker view sorted by severity |
| `intake-queue.html` | 🔲 Pending | Pending deal submissions awaiting admin approval |
| `notifications.html` | 🔲 Pending | Notification inbox with tiered alert escalation |
| `partner-management.html` | 🔲 Pending | Partner roster, View As mode, deal assignments, vendor address book |
| `workflow-builder.html` | 🔲 Pending | Checklist template editor — 10 categories, 7 gates, drag-to-reorder |
| `audit-log.html` | 🔲 Pending | Platform-wide activity feed with filters |
| `settings.html` | 🔲 Pending | Config, integrations, notification rules, integration status panel |

### Issuer Portal — `issuer/screens/`
| File | Status | Description |
|---|---|---|
| `dashboard.html` | 🔲 Pending | Issuer deal overview, readiness progress |
| `deal.html` | 🔲 Pending | Deal detail (issuer-scoped view) |
| `checklist.html` | 🔲 Pending | Checklist — issuer-owned items only |
| `documents.html` | 🔲 Pending | Document upload center |
| `activity.html` | 🔲 Pending | Deal thread + activity feed |

### Legal Partner Portal — `legal/screens/`
| File | Status | Description |
|---|---|---|
| `dashboard.html` | 🔲 Pending | Legal workload view |
| `deal.html` | 🔲 Pending | Deal detail (legal-scoped) |
| `document-review.html` | 🔲 Pending | Document review queue + approval actions |
| `signoffs.html` | 🔲 Pending | Outstanding signoffs needed |
| `activity.html` | 🔲 Pending | Legal activity log |

### Backend Partner Portal — `backend/screens/`
| File | Status | Description |
|---|---|---|
| `dashboard.html` | 🔲 Pending | Technical workload view |
| `deal.html` | 🔲 Pending | Deal detail (backend-scoped) |
| `technical-checklist.html` | 🔲 Pending | Smart contract, token issuance, chain config items |
| `vendor-status.html` | 🔲 Pending | Provider connection status (chain, custodian, KYC) |
| `activity.html` | 🔲 Pending | Backend activity log |

### Broker Portal — `broker/screens/`
| File | Status | Description |
|---|---|---|
| `dashboard.html` | 🔲 Pending | Broker overview — submitted deals, commissions |
| `submit-deal.html` | 🔲 Pending | Deal submission form |
| `my-deals.html` | 🔲 Pending | Deals submitted by this broker |
| `activity.html` | 🔲 Pending | Activity feed |

---

## Design System

**Fonts:** DM Sans (UI) + DM Mono (data/numbers) — Google Fonts  
**Tokens:** `public/design-tokens.css` — all colors, spacing, type scale, radii, shadows  
**Admin components:** `admin/styles/admin.css` — full component library

### Color palette
| Token | Value | Use |
|---|---|---|
| `--color-bg` | `#f5f3ef` | Page background |
| `--color-gold` | `#b8952a` | Brand accent, active states |
| `--color-alert` | `#c9907a` | Red — hard stops, critical |
| `--color-warn` | `#c9a050` | Yellow — warnings, medium risk |
| `--color-green` | `#7aab8a` | Green — complete, approved |

**Principles:** Off-white base, gold used sparingly, 1px borders, status = dot + label (never background flooding)

---

## Deal Pipeline — 10 Stages

```
Sourced → Qualified → Intake → Readiness Review → Legal/Compliance
→ Structuring → Implementation → Pre-Launch → Active → Ongoing Admin
```

### 7 Activation Gates
1. Deal Profile Complete
2. Legal Entity Verified
3. Compliance Cleared
4. Asset Verified + Valued
5. Offering Structured
6. Technical Stack Ready
7. Launch Authorized

---

## 10 Checklist Categories

1. Deal Profile
2. Legal / Entity
3. Local Jurisdiction / CR
4. Compliance / KYC-AML
5. Asset / Property
6. Financial / Valuation
7. Commercial / Offering
8. Technical / Tokenization
9. Vendors / Partners
10. Post-Launch

---

## Deal Health Logic

| State | Condition |
|---|---|
| 🔴 Red | Any hard stop present, OR readiness < 40% |
| 🟡 Yellow | Readiness 40–69%, no hard stops |
| 🟢 Green | Readiness ≥ 70%, no hard stops |

**Activation Readiness Score:** simple % of checklist items complete (no weighting)

---

## Risk Assessment — 4 Dimensions

Tracked independently per deal (severity: Critical → High → Medium → Low):
- **Legal** — entity structure, title, contract risk
- **Financial** — valuation gaps, cap structure, cost overruns
- **Technical** — chain config, smart contract, integration
- **Jurisdictional** — CR regulatory, HNWI requirements, foreign ownership

---

## Alert Escalation Rules

| Day | Action |
|---|---|
| Day 1 | Notify assigned partner |
| Day 3 | Escalate to admin |
| Day 7 | Critical flag — deal at risk |

---

## Admin Capabilities

- **View As** — admin can preview any partner portal as that role (issuer, legal, backend, broker)
- **Deal Threads** — per-deal message thread visible to assigned parties
- **@mentions** — tag a partner directly in a comment
- **Broadcast Messages** — admin sends to all parties on a deal
- **Notification Inbox** — tiered by severity
- **PDF Export** — deal summary + internal memo
- **Internal Deal Memo** — admin-only notes (gold background, not visible to partners)

---

## Live Deals (V1 Data)

### T2T-001 — Hotel Arco Iris
- Type: Acquisition · Tamarindo, Costa Rica
- Stage: Readiness Review
- Health: 🔴 Red (0% complete)
- Value: $2.32M acquisition + $500K renovation
- Hard Stops (3): CR attorney not engaged · No legal entity formed · No legal title review
- Risk: Legal Critical, Jurisdictional Critical, Financial High
- Blockers: 3 open · Days in stage: 14d
- Next action: CR Legal Counsel

### T2T-002 — Grande Beach Club & Villas
- Type: Development · Playa Grande, Costa Rica
- Stage: Intake
- Health: 🟡 Yellow (28% complete)
- Value: $35M–$70M+ (15 villa lots + beach club)
- Risk: Technical High, Commercial Medium
- Blockers: 2 open · Days in stage: 7d
- Next action: Admin / T2T

---

## Backend Architecture

See `docs/architecture.md` for:
- Full data model (WorkflowTemplate, Stage, Step, DealChecklist, ChecklistItem, AuditEvent)
- 16 API endpoints
- Integration adapter pattern (DocConnect, chain provider, custodian, KYC)
- Key decisions on roles, template versioning, status derivation, invite expiry, overdue flagging

**Token standard:** ERC-3643 on Base blockchain  
**Document signing:** DocConnect (adapter — swappable)  
**KYC/AML:** Provider-agnostic adapter pattern

---

## File Structure

```
title2token/
├── public/
│   └── design-tokens.css       # Shared design system (all portals)
├── admin/
│   ├── styles/admin.css        # Admin component library
│   └── screens/                # Admin HTML screens
├── issuer/
│   ├── styles/
│   └── screens/
├── legal/
│   ├── styles/
│   └── screens/
├── backend/
│   ├── styles/
│   └── screens/
├── broker/
│   ├── styles/
│   └── screens/
└── docs/
    └── architecture.md         # Backend data models + API map
```
