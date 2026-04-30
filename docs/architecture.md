# Title2Token — Architecture Notes

## Backend priorities (build in this order)

### 1. Role + permission system
Every portal depends on this. Must support dynamic role assignment without a code deploy.

- Roles: `admin`, `developer`, `broker`, `issuer`, `legal`, `backend`
- Access levels: `full` | `view`
- Endpoint: `PATCH /users/:id/role` — admin only

### 2. Workflow engine (state machine)
The reusable checklist logic that powers every deal.

**Data model:**
```
WorkflowTemplate
  id, name, deal_type, jurisdiction
  stages: Stage[]

Stage
  id, template_id, name, order

Step
  id, stage_id, text, role, required (bool), order (int)

DealChecklist (instance of template applied to a deal)
  id, deal_id, template_id

ChecklistItem (instance of step)
  id, checklist_id, step_id, status (pending|complete|overdue), assigned_to
```

**Rules:**
- Required steps block stage advancement
- Optional steps can be skipped
- Template edits create a new version — active deals warn admin before applying
- `order` integer on steps/stages enables drag-to-reorder without DB restructure

### 3. Audit log (event sourcing)
Every action platform-wide emits an event. Do not retrofit this later.

```
AuditEvent
  id, timestamp, actor_id, actor_type, action, entity_type, entity_id, metadata (json), deal_id?
```

- Actor types: `user`, `system`, `docconnect`
- Action examples: `stage.advanced`, `user.suspended`, `document.signed`, `checklist.item.overdue`
- Cursor-based pagination only — never load full history
- Filter by: actor, deal, action type, date range

### 4. DocConnect integration (adapter pattern)
```
interface SigningProvider {
  sendEnvelope(dealId, documentIds, signers): EnvelopeResult
  getStatus(envelopeId): EnvelopeStatus
  getAuditTrail(envelopeId): AuditEvent[]
}

class DocConnectAdapter implements SigningProvider { ... }
```

Webhook handler updates `ChecklistItem.status` on signature completion.

### 5. Provider-agnostic handoff
Same adapter pattern for:
- Chain provider (token issuance, smart contract)
- Custodian (asset custody, registry)
- KYC/AML provider (identity verification)

Credentials stored in Settings > Integrations. Swap provider = reconnect in UI, no code change.

---

## API endpoint map (admin portal)

| Method | Endpoint | Description |
|---|---|---|
| GET | `/admin/stats` | Dashboard metrics (deals, AUM, pending, users) |
| GET | `/deals` | Pipeline list with filters + pagination |
| GET | `/deals/:id` | Deal detail |
| PATCH | `/deals/:id/stage` | Advance or override stage |
| GET | `/deals/:id/checklist` | Checklist items for deal |
| PATCH | `/checklist-items/:id` | Update item status |
| GET | `/workflow-templates` | List all templates |
| GET | `/workflow-templates/:id` | Template with stages + steps |
| PUT | `/workflow-templates/:id` | Save template changes |
| GET | `/users` | All users with roles, grouped by portal |
| POST | `/users/invite` | Send invite |
| PATCH | `/users/:id` | Update role, access, status |
| DELETE | `/users/:id` | Suspend user |
| GET | `/audit-log` | Paginated events (cursor-based) |
| GET | `/settings` | Platform config |
| PUT | `/settings` | Save config |
| POST | `/settings/integrations/:provider/connect` | Connect integration |

---

## Key decisions for the developer conversation

1. **How do we model roles so admin can configure them without a code deploy?**
   → Role + access stored on the user record, checked at API middleware level

2. **Template versioning — how do we handle edits against active deals?**
   → Version number on template, deals store `template_id + version_at_start`. UI warns on edit if active deals exist.

3. **Status (active/away/offline) — manual or derived?**
   → Derived from `last_seen_at` timestamp. Active = <5min, Away = <30min, Offline = >30min

4. **Invite expiry**
   → Configurable in Settings (default 7 days). Cron job marks expired invites.

5. **Overdue flagging**
   → Cron job runs hourly, compares `ChecklistItem.due_at` against now, emits `checklist.item.overdue` audit event, notifies admin via push + email.
