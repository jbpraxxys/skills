---
name: figma-to-vue
description: Translate Figma designs into Vue/Nuxt components for Nuxt 4 + Vue 3 + TypeScript + Tailwind CSS v4 projects. Use when a user provides a Figma design link or node ID and asks to create a page, section, or component, or when implementing CMS-driven modular sections and reusable shared components from design specs.
compatibility: opencode
metadata:
    stack: nuxt, vue, typescript, tailwindcss-v4
    vision-required: true
---

# Figma to Vue/Nuxt Code Translation

## Overview

This skill defines the workflow for translating Figma designs into Vue/Nuxt components. This skill is designed for vision-capable models that can analyze screenshots and Figma metadata.

## Nuxt 4 Auto-Import Conventions (CRITICAL)

Nuxt 4 auto-imports virtually everything. You MUST follow these rules:

### What is Auto-Imported (NO import statement needed)

| Category | Examples | Location |
|---|---|---|
| **Components** | All `.vue` files in `app/components/` | Auto-imported by filename + path |
| **Vue APIs** | `ref`, `computed`, `watch`, `onMounted`, `defineProps`, `defineEmits` | Built-in |
| **Nuxt composables** | `useFetch`, `useAsyncData`, `useRoute`, `useRouter`, `useRuntimeConfig`, `useNuxtApp` | Built-in |
| **Project composables** | Everything in `app/composables/` | Directory-based |
| **Project utils** | Everything in `app/utils/` | Directory-based |
| **Nuxt built-in components** | `<NuxtImg>`, `<NuxtLink>`, `<NuxtPicture>`, `<NuxtPage>`, `<ClientOnly>` | Built-in |

### What is NOT Auto-Imported (MUST use explicit `import`)

| Category | Explanation |
|---|---|
| **TypeScript types/interfaces** | Types are erased at compile time; must `import type { ... }` |
| **Third-party packages** | Anything from `node_modules` (unless the package provides a Nuxt module) |
| **Server utils** | `server/utils/` is auto-imported on server-side only |

### Component Naming & Nested Directories

By default, Nuxt 4 prefixes component names with their directory path. Example:
- `app/components/base/Button.vue` → auto-imported as `<BaseButton />`
- `app/components/modular/banners/HeroBanner.vue` → auto-imported as `<ModularBannersHeroBanner />`

**The project MUST configure `pathPrefix: false` in `nuxt.config.ts`** to keep component names clean while allowing nested directories for organization:

```ts
// nuxt.config.ts
export default defineNuxtConfig({
  components: [
    { path: '~/app/components', pathPrefix: false },
  ],
})
```

With `pathPrefix: false`, only the filename matters:
- `app/components/modular/banners/HeroBanner.vue` → `<HeroBanner />`
- `app/components/shared/buttons/BaseButton.vue` → `<BaseButton />`

### Kebab-Case in Templates

Auto-imported PascalCase components are used as kebab-case in `<template>`:
```vue
<template>
  <hero-banner :data="data" />
  <base-button @click="handleClick">Click</base-button>
  <nuxt-img src="/hero.jpg" />
</template>
```

## When to Use This Skill

Use this skill when:

- A user provides a Figma design link and asks to create a page or component
- You need to translate a visual design into working Vue/Nuxt code
- Implementing new CMS-driven modular sections
- Creating reusable shared components based on design specs

## Prerequisites

- Figma Dev Mode MCP tools available (`figma-dev-mode_*`)
- Browser screenshot tools available (`chrome-devtools_*` or `playwright_browser_*`)
- Project uses Nuxt 4 + Vue 3 + TypeScript + Tailwind CSS v4
- Access to the project's component library and design system

---

## Phase 1: Design Discovery (MANDATORY)

### 1.1 Extract Figma Data

Use ONLY `figma-dev-mode` MCP tools. Do NOT use any other method.

**Steps:**

1. Parse the node ID from the Figma URL. For `https://www.figma.com/design/:fileKey/:fileName?node-id=1-2`, the nodeId is `1:2`.
2. Call `figma-dev-mode_get_metadata` with the nodeId to understand structure
3. Call `figma-dev-mode_get_design_context` to get design tokens, colors, typography, and reference code
4. Call `figma-dev-mode_get_variable_defs` to get design system variables
5. Call `figma-dev-mode_get_screenshot` to capture the visual representation

**Critical:** Always take a screenshot of the Figma design. Visual analysis is essential for accurate translation.

### 1.2 Asset Discovery & Download

**Philosophy:** Always prefer original assets over screenshots. Effects (border radius, shadows, overlays, gradients) should be implemented at the **code level** using Tailwind CSS, not baked into raster images.

**Why direct download over screenshot:**
- **Original quality:** SVGs remain vectors, PNGs keep transparency
- **Editable:** Can modify colors, apply hover states, animate
- **Smaller file size:** No unnecessary rasterization
- **Responsive:** Effects via CSS adapt to container size
- **Themeable:** Can swap colors via CSS variables

**Priority 1: Direct asset URLs from design context (ALWAYS TRY FIRST)**

The `figma-dev-mode_get_design_context` tool exposes original asset files as localhost URLs.

**Workflow:**
1. Call `figma-dev-mode_get_design_context` on the node containing images
2. In the response, look for asset constants like:
   ```javascript
   const imgShape = "http://localhost:3845/assets/9da174f8bc933e8e422f917c89922e5f61f1c762.svg";
   const cardImage = "http://localhost:3845/assets/abc123...png";
   ```
3. Use `fetch()` in a sandbox script to download the file:
   ```javascript
   const response = await fetch('http://localhost:3845/assets/9da174f8...svg');
   const buffer = await response.arrayBuffer();
   fs.writeFileSync('public/images/arrow-icon.svg', Buffer.from(buffer));
   ```
4. Save to the project's `public/images/` directory with descriptive names

**Handling Figma Effects in Code:**

When the design has effects on image nodes, implement them in the Vue component:

| Figma Effect | Code Implementation |
|-------------|-------------------|
| Border radius | `rounded-lg`, `rounded-xl`, `rounded-2xl` |
| Drop shadow | `shadow-lg`, `shadow-xl`, custom `shadow-*` utilities |
| Inner shadow | Use `ring` or `border` with subtle colors |
| Gradient overlay | Use `bg-gradient-to-*` with `from-*` / `to-*` / `via-*` |
| Opacity/Blend modes | Use `bg-opacity-*` or `mix-blend-mode-*` |
| Mask/clipping | Use `overflow-hidden` with `rounded-*` |

Example:
```vue
<!-- Image with rounded corners and shadow -->
<div class="rounded-xl shadow-lg overflow-hidden aspect-[296/320]">
    <nuxt-img
        src="/images/downloaded-asset.jpg"
        alt="Description"
        class="w-full h-full object-cover" />
</div>

<!-- Image with gradient overlay -->
<div class="relative rounded-xl overflow-hidden aspect-[296/320]">
    <nuxt-img
        src="/images/downloaded-asset.jpg"
        alt="Description"
        class="w-full h-full object-cover" />
    <div class="absolute inset-0 bg-gradient-to-t from-black/60 to-transparent" />
</div>
```

**Priority 2: Screenshot capture (ONLY when no asset URL exists)**

Use `figma-dev-mode_get_screenshot` only when:
- The design context returns no asset URLs
- The image is a complex composition that cannot be separated (e.g., photo with embedded text)
- The asset is a raster image with no vector/source version available

**Priority 3: Lorem Picsum (absolute last resort)**
- Only use `https://picsum.photos` if both Figma methods fail
- Use seed-based URLs for consistency: `https://picsum.photos/seed/<name>/<width>/<height>`

**Aspect Ratio Rule:** Every image container must preserve the original Figma dimensions using Tailwind's `aspect-[w/h]` utility.

- Calculate the ratio from the Figma node's width and height (e.g., `296 × 320` → `aspect-[296/320]`).
- Wrap `<nuxt-img>` in a parent `<div>` with `w-full` and the computed aspect ratio.
- Apply `w-full h-full object-cover` to the image itself.

**Asset Documentation:**
- Save all extracted assets to `public/images/`
- Document the mapping in a comment block:
```vue
<!-- Assets from Figma node 1-63002:
     - Arrow icon: /images/arrow-icon.svg (direct download from design context)
     - Card background: /images/card-bg.jpg (direct download from design context)
     - Effects applied in code: rounded-xl, shadow-lg, gradient overlay
-->
```

---

## Phase 2: Component Architecture Decision

### 2.1 Analyze Design Structure

Determine component boundaries by asking:

1. **Is this a page section or a standalone component?**
    - Sections → `components/modular/<category>/`
    - Reusable atoms → `components/shared/<category>/`

2. **Does it contain repeating child elements?**
    - Cards in a grid → Extract `Card` component
    - List items → Extract `Item` component
    - Form fields → Use existing `BaseInput`, `BaseSelect`, etc.

3. **Is it CMS-driven or static?**
    - CMS-driven → Requires typed `data` prop interface in `app/types/modular/`
    - Static → Can use inline data or props

### 2.2 Component Hierarchy Decision Tree

```
Design Input
    |
    v
Is it a full page layout? ---YES--> Create page in app/pages/
    | NO
    v
Is it a page section/frame? ---YES--> Create in components/modular/<category>/
    | NO
    v
Is it reusable across sections? ---YES--> Create in components/shared/<category>/
    | NO
    v
Is it a one-off unique section? ---YES--> Create in components/modular/unique/
```

### 2.3 Sub-component Extraction Rules

Extract sub-components when:

- Elements repeat 3+ times (cards, items, slides)
- The element has distinct props that vary per instance
- The element could be reused in other sections
- The element is complex enough to warrant isolation

**Example:** A "Restaurant Listing" section with cards should have:

- `RestaurantListFrame.vue` (modular section)
- `RestaurantCard.vue` (shared card component)

---

## Phase 3: Code Generation

### 3.1 File Structure Convention

Follow the project's established structure:

```
app/
  components/
    layout/          # Header, Footer, NavDropdown
    modular/         # CMS-driven page sections
      banners/       # Hero banners, image sliders
      cards/         # Card grid sections
      description/   # Text/image split sections
      sliders/       # Carousel sections
      unique/        # One-off sections (forms, FAQs, etc.)
    shared/          # Reusable atomic components
      buttons/       # BaseButton, NavigationButton, etc.
      cards/         # BlogCard, ExperienceCard, etc.
      fields/        # BaseInput, BaseSelect, etc.
      skeletons/     # Loading states
  types/
    modular/         # TypeScript interfaces for component props
  pages/
    [...slug].vue    # Dynamic CMS page (catch-all)
    blog/[slug].vue  # Blog detail
```

### 3.2 Component Code Standards

**Vue File Structure (MANDATORY):**

The `<script>` block must always come **before** the `<template>` block:

```vue
<script lang="ts" setup>
// ... imports and logic
</script>

<template>
<!-- ... markup -->
</template>
```

**Script Structure:**

```vue
<script lang="ts" setup>
// Only TypeScript type imports — everything else is auto-imported by Nuxt 4
import type { FrameNameData } from '~/types/modular/category';

/*==============================================================================
 * PROPS — defineProps is auto-imported
 *============================================================================*/
const props = defineProps<{ data: FrameNameData }>();

/*==============================================================================
 * COMPOSABLES — useAssetUrl() is auto-imported from app/composables/
 *============================================================================*/
const { getAssetUrl } = useAssetUrl();

/*==============================================================================
 * COMPUTED — computed() is auto-imported (Vue API)
 *============================================================================*/
const hasContent = computed(() => !!props.data.title);

/*==============================================================================
 * ACTIONS
 *============================================================================*/
const handleClick = () => { ... };
</script>
```

**Key Rules:**

- Use `<script lang="ts" setup>` (Composition API only)
- **NO manual imports for Vue APIs** — `ref`, `computed`, `watch`, `defineProps`, `defineEmits`, `onMounted` are auto-imported
- **NO manual imports for Nuxt composables** — `useFetch`, `useAsyncData`, `useRoute`, `useRouter` are auto-imported
- **NO manual imports for project composables** — everything in `app/composables/` (e.g., `useAssetUrl`) is auto-imported
- **NO manual imports for components** — all `app/components/` are auto-imported; use kebab-case in `<template>`
- **ONLY `import type`** — TypeScript interfaces must be explicitly imported since types are erased at compile time
- Single `data` prop typed from `~/types/modular/<category>.ts`
- No scoped CSS — 100% Tailwind utilities
- No inline styles except for dynamic runtime values
- Use `<nuxt-img>` (auto-imported built-in component) instead of `<img>`
- Use `getAssetUrl()` for all asset paths (returned from auto-imported `useAssetUrl()`)
- Kebab-case in templates (`<blog-card>` not `<BlogCard>`)
- 4-space indentation, semicolons required

### 3.3 Tailwind & Design System Compliance

**CRITICAL RULES:**

1. **No arbitrary values** — Use `gap-1.5` not `gap-[5px]`
2. **No hex codes in classes** — Use `text-brand-primary` not `text-[#0b202f]`
3. **OKLCH for new colors** — All theme additions must be OKLCH format
4. **Use existing utilities** — Check `app/assets/css/design-system.css` first
5. **Use design tokens** — Typography via Tailwind's standard `--text-*` scale (e.g. `text-xl`, `text-6xl`) plus `font-primary` / `font-secondary` for font family

### 3.4 TypeScript Interface Generation

Every modular component MUST have a corresponding interface:

```ts
// app/types/modular/category.ts
export interface FrameNameData {
    heading?: string;
    description?: string;
    image?: string;
    items?: Array<{
        title: string;
        description: string;
        image: string;
    }>;
}
```

---

## Phase 4: API Readiness & Documentation

### 4.1 When API Exists

If the backend API is ready:

- Use `useFetch()` (auto-imported) with proper typing — no need to import from `nuxt/app`
- Implement `transform` for data normalization
- Use `getCachedData` for payload caching
- Match the CMS section_type to component mapping in `useComponentMap.ts`

### 4.2 When API Does NOT Exist (MOST COMMON)

If the API is not ready:

1. **Create the component with static inline data** (sample object matching the interface)
2. **Add a sample page** at `app/pages/sample/<component-name>.vue`
3. **Document the expected API shape** in a comment block
4. **Mark TODOs** where dynamic data will replace static values

Example:

```vue
<script lang="ts" setup>
// TODO: Replace with API call when endpoint is ready
const sampleData: FrameNameData = {
    heading: 'Sample Heading',
    items: [{ title: 'Item 1', description: '...', image: '/images/placeholder.jpg' }],
};
</script>
<template>
    <frame-name :data="sampleData" />
</template>
```

### 4.3 Documentation Template

Create a brief markdown doc alongside the component:

```markdown
# Component: FrameNameFrame

## Purpose

[Brief description of what this section displays]

## CMS Configuration

- **section_type**: `frame_name`
- **Expected fields**: heading, description, items[]

## Sub-components

- `FrameNameCard.vue` (shared/cards/)

## API Endpoint

- `GET /api/v1/dynamic-pages/{slug}`
- Section data shape: [document the expected JSON structure]

## Assets Required

- [ ] Hero background image
- [ ] Card thumbnails (variable count)

## Responsive Behavior

- Desktop: [describe]
- Tablet: [describe]
- Mobile: [describe]
```

---

## Phase 5: QA & Visual Comparison

### 5.1 Build and Verify

1. Run `npm run dev` to start the dev server
2. Navigate to the sample page or the page where the component is used
3. Take a screenshot using `chrome-devtools_take_screenshot` or `playwright_browser_take_screenshot`

### 5.2 Visual Comparison

**MANDATORY:** Compare the coded version against the Figma screenshot:

1. **Layout alignment** — Check padding, margins, grid alignment
2. **Typography** — Verify font sizes, weights, line heights match
3. **Colors** — Ensure brand colors are correct (OKLCH values)
4. **Spacing** — Check gaps, section padding
5. **Responsive** — Verify mobile/tablet breakpoints

### 5.3 QA Checklist

- [ ] Component renders without errors
- [ ] Screenshot matches Figma design within acceptable tolerance
- [ ] All images load (or placeholders show correctly)
- [ ] Responsive breakpoints work correctly
- [ ] No console errors or warnings
- [ ] TypeScript compiles without errors
- [ ] Tailwind classes follow project conventions (no arbitrary values)
- [ ] Component is registered in `useComponentMap.ts` if CMS-driven

---

## Subskills Reference

This skill is composed of several subskills for detailed guidance:

1. **[Component Architecture](subskills/component-architecture.md)** — Detailed component hierarchy, extraction rules, and naming conventions
2. **[Frontend Conventions](subskills/fe-conventions.md)** — Tailwind v4 rules, OKLCH colors, typography system, and coding standards
3. **[QA & Visual Comparison](subskills/qa-visual-comparison.md)** — Screenshot comparison workflow, tolerance thresholds, and debugging
4. **[API Readiness](subskills/api-readiness.md)** — Handling missing APIs, sample data patterns, and documentation templates
5. **[Subagent Patterns](subskills/subagent-patterns.md)** — When and how to use subagents for context preservation

---

## Tool Reference

### Figma Dev Mode Tools (USE THESE ONLY)

- `figma-dev-mode_get_metadata` — Get node structure
- `figma-dev-mode_get_design_context` — Get design tokens and reference code
- `figma-dev-mode_get_variable_defs` — Get design system variables
- `figma-dev-mode_get_screenshot` — Capture visual screenshot

### Browser/Screenshot Tools

- `chrome-devtools_take_screenshot` — Capture page screenshot
- `playwright_browser_take_screenshot` — Alternative screenshot method
- `chrome-devtools_navigate_page` — Navigate to specific URL

### Subagents

- Use `task` tool with `subagent_type: explore` for research
- Use `task` tool with `subagent_type: prx-nuxt` for component implementation
- Always provide complete context in subagent prompts

---

## Vision Model Notes

This skill is optimized for vision-capable models. Key capabilities leveraged:

1. **Visual analysis** — Screenshot comparison between Figma and coded output
2. **Layout detection** — Identifying grids, flex layouts, spacing from screenshots
3. **Color extraction** — Matching brand colors from visual inspection
4. **Typography matching** — Identifying font sizes, weights, and families

When working with vision models:

- Always provide the Figma screenshot alongside metadata
- Point out specific visual elements that need precise matching
- Use the screenshot as the primary truth source, metadata as secondary

---

## Example Workflow

```
User: "Create the Restaurant Listing section from Figma node 386:35576"

1. Extract node ID → "386-35576" or "386:35576"
2. Get metadata → Understand structure (header, cards, etc.)
3. Get screenshot → Visual reference
4. Get design context → Colors, typography tokens
5. Decide architecture → RestaurantListFrame.vue + RestaurantCard.vue
6. Create TypeScript interface in app/types/modular/restaurant.ts
7. Generate components following project conventions
8. Create sample page with static data
9. Build and take screenshot
10. Compare with Figma screenshot
11. Iterate on differences
12. Finalize and document
```
