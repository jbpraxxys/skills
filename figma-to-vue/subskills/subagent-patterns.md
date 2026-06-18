# Subagent Patterns Subskill

## Purpose

Defines when and how to use subagents for context preservation during the Figma-to-code workflow.

---

## Why Use Subagents?

The Figma-to-code workflow is complex and involves multiple phases:
1. Design discovery (Figma metadata, screenshots)
2. Component architecture decisions
3. Code generation
4. QA and visual comparison

**Problem:** Each tool call consumes context window. Long workflows can exceed context limits.

**Solution:** Use subagents to delegate phases, preserving context and enabling parallel work.

---

## Subagent Strategy

### Rule 1: Use Subagents for Parallel Research

When gathering information that doesn't depend on each other, use parallel subagents:

**Example:**
```
Main Agent:
  ├─ Subagent A: Explore existing component patterns
  ├─ Subagent B: Analyze Figma design structure
  └─ Subagent C: Check design system tokens
```

**Tools to use in parallel:**
- `task` with `subagent_type: explore` — For codebase research
- `figma-dev-mode_get_metadata` — For Figma structure
- `figma-dev-mode_get_design_context` — For design tokens
- `read` — For checking existing conventions

### Rule 2: Use Subagents for Context Preservation

When context would be lost between phases, delegate to a subagent:

**Example:**
```
Phase 1: Design Discovery (subagent)
  → Returns: Figma screenshot, metadata summary, design tokens

Phase 2: Component Architecture (main agent)
  → Uses: Results from Phase 1

Phase 3: Code Generation (subagent)
  → Returns: Component files, TypeScript interfaces

Phase 4: QA (subagent)
  → Returns: Screenshot comparison, issues found
```

### Rule 3: Don't Duplicate Work

Once you delegate to a subagent, don't repeat the same work yourself. Wait for results and continue with dependent tasks.

---

## Subagent Types by Phase

### Phase 1: Design Discovery

**Subagent type:** `explore`
**Purpose:** Gather all design information from Figma

**Prompt template:**
```
Analyze the Figma design at node {nodeId} for the {projectName} project.

1. Get metadata using figma-dev-mode_get_metadata with nodeId "{nodeId}"
2. Get design context using figma-dev-mode_get_design_context with nodeId "{nodeId}"
3. Get variable definitions using figma-dev-mode_get_variable_defs with nodeId "{nodeId}"
4. Take a screenshot using figma-dev-mode_get_screenshot with nodeId "{nodeId}"

Return a comprehensive summary including:
- Visual layout description (what sections exist)
- Typography details (font sizes, weights, families)
- Color palette (primary, secondary, accent, neutrals)
- Spacing values (padding, margins, gaps)
- Component boundaries (what should be separate components)
- Asset requirements (images, icons needed)
- Responsive considerations

Include the screenshot in your analysis for visual reference.
```

### Phase 2: Component Architecture

**Subagent type:** `explore` or `general`
**Purpose:** Analyze codebase and recommend component structure

**Prompt template:**
```
Analyze the existing component patterns in {projectPath}/app/components/ to recommend the best structure for a new {componentType} component.

Check:
1. Existing similar components in components/modular/{category}/
2. Existing shared components in components/shared/{category}/
3. TypeScript interfaces in app/types/modular/
4. Component mapping in app/composables/useComponentMap.ts

Recommend:
- Which directory to place the component
- Whether to extract sub-components
- Props interface shape
- Which existing components to reuse
- Naming convention following project standards

Return a detailed recommendation with file paths and code examples.
```

### Phase 3: Code Generation

**Subagent type:** `prx-nuxt`
**Purpose:** Generate Vue/Nuxt components following project conventions

**Prompt template:**
```
Create a Vue component for the {componentName} following the Filinvest project conventions.

Project path: {projectPath}
Component location: {componentPath}
Interface location: {typesPath}

Design context: {designSummary}
Component architecture: {architectureRecommendation}

Requirements:
- Use <script lang="ts" setup>
- Follow the project's script section structure (PROPS, COMPOSABLES, COMPUTED, ACTIONS)
- Use Tailwind CSS utilities only (no scoped CSS)
- Use existing design system tokens
- Use nuxt-img for images
- Use getAssetUrl() for asset paths
- Create TypeScript interface in app/types/modular/{category}.ts
- Follow kebab-case in templates
- Include section comment blocks
- 4-space indentation, semicolons

Return:
1. The complete component file
2. The TypeScript interface file
3. Any additional files needed (sample page, sub-components)
```

### Phase 4: QA and Comparison

**Subagent type:** `prx-qa` or `general`
**Purpose:** Test component and compare with Figma design

**Prompt template:**
```
QA the {componentName} component against the Figma design.

1. Start the dev server: npm run dev in {projectPath}
2. Navigate to the sample page: http://localhost:3000/sample/{component-name}
3. Take a screenshot of the rendered component
4. Compare visually with the Figma screenshot: {figmaScreenshot}

Check:
- Layout alignment
- Typography (size, weight, line-height)
- Colors (background, text, borders)
- Spacing (padding, margins, gaps)
- Responsive behavior (resize and test)
- Console errors

Return:
1. QA checklist results
2. List of discrepancies with severity
3. Screenshots of coded component
4. Recommendations for fixes
```

---

## Context Preservation Strategies

### Strategy 1: Summarize and Pass

When delegating, summarize findings concisely:

```
Main Agent → Subagent:
  "Create a RestaurantCard component. Key requirements:
   - Card with image (aspect 4:3), title, cuisine type
   - Hover: scale image 1.05x
   - Uses card-base utility for border-radius
   - Receives data prop typed as RestaurantCardData
   - Project uses Tailwind v4, no scoped CSS"
```

### Strategy 2: Reference Files, Not Content

Instead of pasting file contents, reference paths:

```
"Follow the conventions in {projectPath}/prompt-library/COMPONENT_FORMAT.md
and match the style of {projectPath}/app/components/shared/cards/BlogCard.vue"
```

### Strategy 3: Include Screenshots

When vision capabilities are available, include screenshots in prompts:

```
"Match the visual style shown in this Figma screenshot [attach screenshot]
Specifically:
- The card border radius appears to be 16px (use rounded-2xl or custom)
- Image has gradient overlay at bottom
- Title uses Styrene A font, size appears to be 20px (text-xl font-primary)
```

---

## Subagent Communication Patterns

### Pattern 1: Research Subagent

```
User Request → Main Agent
    |
    v
[Decide what research is needed]
    |
    v
Launch parallel subagents for research
    ├─ Subagent A: Explore existing patterns
    ├─ Subagent B: Get Figma data
    └─ Subagent C: Check design system
    |
    v
[Wait for all results]
    |
    v
Synthesize findings
    |
    v
[Generate or delegate code]
```

### Pattern 2: Implementation Subagent

```
[Have all design + architecture info]
    |
    v
Delegate code generation to prx-nuxt subagent
    |
    v
[Subagent returns files]
    |
    v
Review and apply files
    |
    v
[Delegate QA to another subagent]
    |
    v
[Subagent returns comparison]
    |
    v
Iterate or finalize
```

### Pattern 3: Iteration Subagent

```
[QA finds issues]
    |
    v
Delegate fixes to subagent with specific instructions
    |
    v
[Subagent makes changes]
    |
    v
[Re-run QA]
    |
    v
Repeat until resolved
```

---

## Context Window Management

### Warning Signs of Context Overflow

- Responses become slower
- Model starts forgetting earlier decisions
- Repeated questions about already-discussed topics
- Tool outputs truncated

### Mitigation Strategies

1. **Summarize frequently** — After each phase, summarize key decisions
2. **Use subagents early** — Don't wait until context is full
3. **Store decisions in files** — Write intermediate decisions to temp files
4. **Limit parallel tool calls** — Batch intelligently

### Example: Managing a Complex Component

```
Phase 1 (Subagent A):
  Input: Figma URL
  Output: Design summary file (design-summary.md)
  Context saved: ~50% of window

Phase 2 (Subagent B):
  Input: Design summary + project path
  Output: Architecture recommendation (architecture.md)
  Context saved: ~30% of window

Phase 3 (Subagent C):
  Input: Design summary + architecture + project path
  Output: Component files
  Context saved: ~20% of window

Phase 4 (Subagent D):
  Input: Component files + Figma screenshot
  Output: QA report
  Context fresh: 100% available
```

---

## Subagent Best Practices

1. **Always provide complete context** — Subagents don't see your conversation
2. **Be specific about outputs** — Tell subagent exactly what files to create
3. **Include verification steps** — Ask subagent to verify their work
4. **Use appropriate subagent types:**
   - `explore` — Research, analysis, information gathering
   - `prx-nuxt` — Vue/Nuxt component implementation
   - `prx-qa` — Testing and QA
   - `general` — Multi-step tasks, writing, refactoring

5. **Don't chain too deeply** — 2-3 levels max. Deeper chains lose context
6. **Verify subagent outputs** — Check files before accepting
7. **Save intermediate results** — Write summaries to files for later reference

---

## Example: Complete Workflow with Subagents

```
User: "Create the restaurant listing section from Figma node 386:35576"

Step 1: Parallel research subagents
├─ Subagent A (explore): Get Figma metadata, screenshot, design context
└─ Subagent B (explore): Analyze existing card components and patterns

Step 2: Main agent synthesizes results
└─ Decides: RestaurantListFrame + RestaurantCard architecture

Step 3: Delegate implementation
└─ Subagent C (prx-nuxt): Generate RestaurantListFrame and RestaurantCard

Step 4: Main agent reviews code
└─ Applies files to project

Step 5: Delegate QA
└─ Subagent D (prx-qa): Build, screenshot, compare with Figma

Step 6: Main agent reviews QA report
└─ If issues: Delegate fixes to Subagent C
└─ If passed: Report completion to user
```

This pattern preserves context at each phase and enables parallel work where possible.
