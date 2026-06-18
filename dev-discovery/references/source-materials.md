# Source Material Reference

How to extract information from each source type during Phase 1.

## Figma

Start with the **root node ID** from the Figma URL. Extract the full canvas metadata first, then drill into individual pages.

**Tools in order:**
1. `figma-dev-mode_get_variable_defs(nodeId)` — design tokens (once)
2. `figma-dev-mode_get_screenshot(nodeId)` — canvas overview (once)
3. `figma-dev-mode_get_metadata(nodeId)` — full page inventory (once)
4. `figma-dev-mode_get_design_context(nodeId, forceCode=true)` — per representative page (many times, prioritized)

Always set `forceCode: true` and `artifactType: "WEB_PAGE_OR_APP_SCREEN"`. Set `clientFrameworks` and `clientLanguages` to the actual project stack.

**Metadata strategy**: The metadata dump can be enormous (5MB+). Always dispatch to a subagent to grep/read the truncated output. Never read the full metadata file yourself.

**Page sampling**: Call `get_design_context` on one representative frame per page family — NOT on all 90 frames. The variants are typically just height/width differences (compact vs full, with/without sidebar).

**Prioritized pages**: Login, Dashboard, the main CRUD list pages, Settings, Profile, Roles.

## Merge / Spec Document

**Google Sheet:**
1. `google-drive_getSpreadsheetInfo(spreadsheetId)` — sheet inventory
2. `google-drive_listSheets(spreadsheetId)` — tab list
3. `google-drive_getGoogleSheetContent(spreadsheetId, range)` — per-tab content

**Google Doc:**
1. `google-drive_getDocumentInfo(documentId)`
2. `google-drive_readGoogleDoc(documentId, format="markdown")`
3. `google-drive_listDocumentTabs(documentId)` — if multi-tab

The merge doc typically contains: project scope, glossary, permission matrices, process flows, entity definitions. **Extract ALL tabs** — missing a tab means missing a module.

## Boilerplate / Reference Codebase

The boilerplate is a **pre-existing reference**, not something to re-explore each session.

**If `CONTEXT.md` exists** in the boilerplate directory: load it directly. It already contains all conventions.

**If `CONTEXT.md` does NOT exist**: dispatch an `explore` subagent once to walk the directory and produce the file. Key sections the subagent must cover:
- Composer dependencies and versions
- Project structure (DDD? Modular? Default Laravel?)
- Migration conventions (naming, column types, FK strategy, soft deletes)
- Model conventions (BaseModel, traits, casts, fillable)
- Controller pattern (ResourceController? Invokable? Trait composition?)
- Service layer (CrudService? Actions? Repositories?)
- Auth strategy (Fortify? Sanctum? JWT?)
- Permission system (Spatie? Custom?)
- API conventions (versioning, envelope, filtering)
- Code style (pint.json, .editorconfig)

**Never re-explore a boilerplate that already has CONTEXT.md.** The file is the single source of truth for that codebase's conventions.
