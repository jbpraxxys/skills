# Output Structure (Chunked Docs)

The final deliverable is a **directory of markdown files**, not a single monolith. The structure is two-tier: **fixed files** (always present) and **module files** (one per module from the merge doc).

## Fixed Files (Always Produced)

| File | Contents |
|---|---|
| `00-README.md` | Full index with 1-line descriptions + quick nav + key numbers |
| `01-overview.md` | Executive summary, system overview, module map, all roles with scope |
| `02-tech-stack.md` | Tech stack, conventions, visual design tokens from Figma, package deps |
| `{NN}-cross-cutting.md` | Multi-tenancy, multi-portal, auth strategy, file storage, jobs, caching, search, audit |
| `{NN}-api-contracts.md` | Route conventions, request/response formats, errors, rate limiting, filtering keys |
| `{NN}-permissions.md` | Full RBAC matrix as installable config array (both admin and portal guards) |
| `{NN}-migration-order.md` | All migrations in dependency order — numbered, one per table |
| `{NN}-erd.md` | Mermaid ERDs (master + per-domain islands), state machines, permission hierarchy |
| `{NN}-process-flows.md` | Every process flow from the merge doc: trigger → services → state transitions → notifications → audit |
| `{NN}-open-questions.md` | Assumptions, risks, gaps, validation questions, glossary, sample API workflows |
| `99-master.md` | Concatenated master doc for printing/offline reference |

The `{NN}` prefixes start after the module files and depend on how many modules exist.

## Module Files (Generated from Merge Doc)

One file per module. **The file list is never pre-defined — it is derived** from the module map built in Phase 2.

**Naming**: `{number}-{module-slug}.md` (e.g., `03-auth.md`, `04-employees.md`).

**Grouping**: Closely related modules may share a file if small. Large modules get dedicated files.

**Every module file** follows this internal structure:
1. Module purpose (1 sentence)
2. Figma screen reference with node IDs
3. Database tables with full `Schema::create()` code
4. Model class with traits, relationships, casts
5. Status enums with `BaseEnum` + `meta()`
6. Mermaid state machine (if the entity has >2 statuses)
7. Service class (if complex business logic — payroll, leave, commission)
8. Permission names matching the merge doc matrix
9. UI elements extracted from Figma

## Example: 8-Module HRIS

```
docs/{project}/
├── 00-README.md
├── 01-overview.md
├── 02-tech-stack.md
├── 03-auth.md
├── 04-employees.md
├── 05-patients.md
├── 06-attendance.md
├── 07-payroll.md
├── 08-leave.md
├── 09-loans-ca.md
├── 10-assets.md
├── 11-cross-cutting.md
├── 12-api-contracts.md
├── 13-permissions.md
├── 14-migration-order.md
├── 15-erd.md
├── 16-process-flows.md
├── 17-open-questions.md
└── 99-master.md
```

## Example: 3-Module CMS

```
docs/{project}/
├── 00-README.md
├── 01-overview.md
├── 02-tech-stack.md
├── 03-auth.md
├── 04-content.md
├── 05-media.md
├── 06-cross-cutting.md
├── 07-api-contracts.md
├── 08-permissions.md
├── 09-migration-order.md
├── 10-erd.md
├── 11-open-questions.md
└── 99-master.md
```

**Key principle**: The chunked structure reflects what the project actually contains. Build the file list from the module map, not from a fixed template.
