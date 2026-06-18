# Example: DGT ONE Clinic HRIS (Full Output)

A real dev discovery output for a multi-branch dental clinic HRIS with 15 modules, 70+ tables, 9 roles, and 115+ permissions.

**Sources:** Figma admin prototype (`Admin v4.0.0`, 90 page frames) + Google Sheets merge document (7 tabs: scope, glossary, 3x permission matrices, process flows, approvals).

**Output:** [`docs/dgt/`](../docs/dgt/) — 23 files, 12,646 total lines.

## What the Output Looks Like

| File | Lines | What It Contains |
|---|---|---|
| `00-README.md` | 78 | Index, quick nav, key numbers, reviewer checklist |
| `01-overview.md` | 153 | Executive summary, 3-portal architecture, module map, 9 roles |
| `02-tech-stack.md` | 583 | Laravel 13 + Vue 3 + Inertia 2, conventions, Tailwind v4 theme |
| `03-auth.md` through `14-monitoring.md` | 249–740 each | Full database schemas, models, enums, services, permissions per module |
| `15-cross-cutting.md` | 114 | Multi-branch scoping, multi-portal auth, S3 storage, Horizon queues |
| `16-api-contracts.md` | 173 | Routes, request/response formats, errors, rate limiting |
| `17-permissions.md` | 353 | Installable `config/permissions.php` with 115+ permissions |
| `18-migration-order.md` | 120 | 77 migrations in dependency order |
| `19-erd.md` | 209 | 5 Mermaid diagrams (master ERD, domain islands, state machines) |
| `20-process-flows.md` | 226 | 7 process flows as code skeletons |
| `21-open-questions.md` | 239 | 37 questions with defaults, glossary, sample API workflows |
| `99-master.md` | 6,204 | Full concatenated master |

## Key Architecture Decisions

- **Multi-branch**: Row-level `branch_id` scoping on all operational tables
- **Multi-portal**: Two guards (`admin`, `portal`), mobile overtime restricted via middleware
- **Dentist commissions**: `ServiceLog` → `CommissionService` → `PayrollEntry.commission_pay`
- **Payroll immutability**: Cutoff approval locks all entries; voiding requires reversal audit trail
- **Boilerplate compliance**: Anonymous-class migrations, `archives()` macro, `BaseEnum`, trait-based CRUD

## What to Notice

- The **module files** (03–14) are the meat — each is a self-contained developer reference for that domain
- The **ERD** (19) is the most valuable for visual understanding — Mermaid renders natively
- The **permissions** (17) is an installable config — copy-paste into `config/permissions.php`
- The **open questions** (21) is the contract with the client — every assumption is explicit
