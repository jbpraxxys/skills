---
name: figma-tailwind-design-system
description: Translate Figma design systems into Tailwind CSS v4 configurations. Use when creating, updating, or converting design tokens, themes, color systems, typography scales, or component variants from Figma to Tailwind CSS. Also use when a user asks to create or update a design system, or when Figma variables/tokens need to be implemented in code.
compatibility: opencode
metadata:
  owner: frontend
  stack: nuxt, vue, tailwindcss-v4
  vision-required: true
---

# Figma to Tailwind Design System

A complete workflow for translating Figma design systems into Tailwind CSS v4 configurations. This skill is self-contained and includes all sub-workflows.

## When to Use This Skill

Use this skill when:
- The user asks to create a design system from Figma
- The user asks to update existing design tokens from Figma changes
- The user asks to convert Figma colors/typography/spacing to Tailwind
- The user provides a Figma file URL and mentions "design system", "tokens", "styles", or "theme"
- You detect Figma variables in the file that need to be implemented as CSS custom properties

## Prerequisites

- **Vision-capable model** (Kimi k2.6, Kimi k2.7) — Required for Figma screenshot comparison
- Figma file URL with dev mode access
- Project using Tailwind CSS v4 with CSS-first configuration

## Phase 1: Discovery & Extraction

### 1.1 Get Figma Metadata

Use `figma-dev-mode_get_metadata` to understand the design system structure:
1. Identify pages: "Design System", "Tokens", "Styles", "Colors", "Typography", "Components"
2. Note the node IDs for design system pages
3. Look for pages with names like "Tokens", "Design System", "Colors", "Typography"

### 1.2 Extract Figma Variables

Use `figma-dev-mode_get_variable_defs` on the design system page node ID.

The response is a JSON object. Categorize variables by type:

```json
{
  "colors": {
    "Brand/primary/base": "#0a2132",
    "Brand/secondary/base": "#af937f",
    "Fill/white": "#ffffff",
    "Fill/neutral-primary": "#161616"
  },
  "typography": {
    "Heading/M": {
      "fontFamily": "Styrene A Trial",
      "fontWeight": "Regular",
      "fontSize": "20px",
      "lineHeight": "26px",
      "letterSpacing": "0"
    }
  },
  "spacing": {
    "Radius/L": "16",
    "Radius/XS": "4",
    "Radius/Pill": "999"
  }
}
```

**Variable naming convention:**
- Replace slashes with hyphens: `Brand/primary/base` → `brand-primary`
- Convert to kebab-case
- Use semantic names, not literal values

**Common Figma patterns:**

| Pattern | Description | Action |
|---------|-------------|--------|
| `Brand/*` | Brand colors | Map to `--color-brand-*` |
| `Fill/*` | Surface colors | Map to `--color-fill-*` or `--color-surface-*` |
| `System/*` | System colors | Map to `--color-system-*` |
| `Type/*` | Typography primitives | Extract as base values |
| `Heading/*` | Heading styles | Map to standard `--text-*` scale (e.g. `Heading/M` → `--text-xl`) |
| `Paragraph/*` | Paragraph styles | Map to standard `--text-*` scale (e.g. `Paragraph/S` → `--text-sm`) |
| `Label/*` | Label styles | Map to standard `--text-*` scale (e.g. `Label/M` → `--text-base`) |
| `Display/*` | Display styles | Map to standard `--text-*` scale (e.g. `Display/M` → `--text-6xl`) |
| `Radius/*` | Border radius | Map to `--radius-*` |

### 1.3 Take Screenshots

Use `figma-dev-mode_get_screenshot` on key design system pages:
- Color swatches page
- Typography specimens page
- Component examples page
- Save screenshots for reference during implementation

### 1.4 Analyze Existing Project

Read existing files:
- `app/assets/css/design-system.css` or similar
- `app/assets/css/main.css`
- `nuxt.config.ts` for Tailwind integration
- Document existing tokens to avoid conflicts

## Phase 2: Token Mapping & Conversion

### 2.1 Color System (OKLCH)

**Why OKLCH?**
- Perceptually uniform (unlike hex/HSL)
- Better accessibility (lightness is consistent)
- Wide gamut support

**Convert hex to OKLCH:**

```bash
npm install -D culori

# Single color
npx culori convert '#0a2132' oklch
# Output: oklch(0.2389 0.0436 243.5126)

# Batch conversion
npx culori convert '#af937f' oklch
npx culori convert '#FF9033' oklch
```

**Color token structure:**

```css
@theme {
  /* Brand Colors */
  --color-brand-primary: oklch(0.2389 0.0436 243.5126);        /* #0a2132 */
  --color-brand-secondary-warm: oklch(0.9459 0.0076 61.45);     /* #f6f3f1 */
  --color-brand-secondary-tan: oklch(0.6382 0.0402 57.6847);    /* #af937f */
  --color-brand-gold: oklch(0.6427 0.0491 84.42);               /* #a67c52 */

  /* Neutral Colors */
  --color-white: oklch(1 0 0);
  --color-neutral-secondary: oklch(0.3829 0 0);                 /* #434343 */
  --color-neutral-tertiary: oklch(0.2478 0 0 / 0.3216);         /* #21212152 */
  --color-neutral-50: oklch(0.9851 0 0);                        /* #fafafa */
  --color-neutral-dark: oklch(0.2002 0 89.88);                  /* #161616 */

  /* Accent Colors */
  --color-tango: oklch(0.7588 0.167 55.5274);                   /* #FF9033 */

  /* Opacity Variants */
  --color-opaque-primary: oklch(0 0 0 / 0.2);                   /* #00000033 */
  --color-opaque-primary-inv: oklch(1 0 0 / 0.2);               /* #ffffff33 */

  /* CTA Colors (keep hex for exact brand match) */
  --color-cta-dark: #0b202f;
  --color-cta-dark-hover: #1a3a4f;
  --color-cta-dark-pressed: #0a1c2a;
  --color-cta-light-hover: #f5f5f5;
  --color-cta-light-pressed: #e8e8e8;

  /* Semantic UI Colors */
  --color-border-light: oklch(0.8805 0.0043 -123.5);
  --color-text-muted: oklch(0.15 0 89.88 / 0.4);
}
```

**Color mapping from Figma:**

| Figma Variable | Hex Value | OKLCH Value | CSS Token |
|----------------|-----------|-------------|-----------|
| `Brand/primary/base` | `#0a2132` | `oklch(0.2389 0.0436 243.5126)` | `--color-brand-primary` |
| `Brand/secondary/base` | `#af937f` | `oklch(0.6382 0.0402 57.6847)` | `--color-brand-secondary-tan` |
| `Fill/white` | `#ffffff` | `oklch(1 0 0)` | `--color-white` |
| `Fill/neutral-primary` | `#161616` | `oklch(0.2085 0 0)` | `--color-neutral-primary` |
| `tango/base` | `#FF9033` | `oklch(0.7588 0.167 55.5274)` | `--color-tango` |

**Rules:**
- All new colors in OKLCH format
- Use 4 decimal places: `oklch(0.2389 0.0436 243.5126)`
- Document hex source in comments
- Use semantic naming (purpose, not color description)
- Alpha in OKLCH: `oklch(L C H / alpha)`

### 2.2 Typography Scale

**Extract from Figma:**

```json
{
  "Heading/M": {
    "fontFamily": "Styrene A Trial",
    "fontWeight": "Regular",
    "fontSize": "20px",
    "lineHeight": "26px",
    "letterSpacing": "0"
  },
  "Paragraph/S": {
    "fontFamily": "Avenir LT STD",
    "fontWeight": "35 Light",
    "fontSize": "14px",
    "lineHeight": "22px",
    "letterSpacing": "0"
  }
}
```

**Convert to CSS tokens:**

Use Tailwind's standard `--text-*` namespace. Each `--text-<size>` can pair with `--text-<size>--line-height`, `--text-<size>--letter-spacing`, and `--text-<size>--font-weight` to set defaults for that utility.

```css
@theme {
  /* Font Families */
  --font-sans: 'Avenir', sans-serif;
  --font-primary: 'Styrene A', sans-serif;
  --font-secondary: 'Avenir', sans-serif;

  /* Standard Tailwind text scale — override with design system values */
  --text-xs: 0.75rem;                        /* eyebrow, label-xs */
  --text-xs--line-height: 1rem;

  --text-sm: 0.875rem;                       /* paragraph-s, label-s */
  --text-sm--line-height: 1.375rem;

  --text-base: 1rem;                         /* heading-xs, paragraph-m, label-m */
  --text-base--line-height: 1.5rem;

  --text-lg: 1.125rem;                       /* heading-s, paragraph-l, label-l */
  --text-lg--line-height: 1.625rem;

  --text-xl: 1.25rem;                        /* heading-m */
  --text-xl--line-height: 1.625rem;

  --text-2xl: 1.375rem;                      /* display-s */
  --text-2xl--line-height: 1.625rem;
  --text-2xl--letter-spacing: -0.0133em;

  --text-3xl: 1.75rem;                       /* heading-l */
  --text-3xl--line-height: 2rem;

  --text-4xl: 2rem;                          /* heading-xl */
  --text-4xl--line-height: 2.25rem;

  --text-5xl: 2.5rem;                        /* heading-2xl */
  --text-5xl--line-height: 2.75rem;

  --text-6xl: 3rem;                          /* display-m, heading-3xl */
  --text-6xl--line-height: 3.25rem;
  --text-6xl--letter-spacing: -0.0133em;
}
```

**Mapping Figma text styles to Tailwind utilities:**

| Figma Style | Tailwind Utility | Notes |
|-------------|------------------|-------|
| Eyebrow | `text-eyebrow` | Composite style; defined as an `@utility` in `design-system.css` |
| Heading/XS | `text-base font-primary` | |
| Heading/S | `text-lg font-primary` | |
| Heading/M | `text-xl font-primary` | |
| Heading/L | `text-3xl font-primary` | |
| Heading/XL | `text-4xl font-primary` | |
| Heading/2XL | `text-5xl font-primary` | |
| Heading/3XL | `text-6xl font-primary` | |
| Paragraph/S | `text-sm font-secondary` | |
| Paragraph/M | `text-base font-secondary` | |
| Paragraph/L | `text-lg font-secondary` | |
| Label/XS | `text-xs font-primary` | |
| Label/S | `text-sm font-primary` | |
| Label/M | `text-base font-primary` | |
| Label/L | `text-lg font-primary` | |
| Display/S | `text-2xl font-primary` | Includes tight letter-spacing |
| Display/M | `text-6xl font-primary` | Includes tight letter-spacing |

**Font weight mapping:**

| Figma Weight | CSS | Numeric |
|--------------|-----|---------|
| Light | font-light | 300 |
| Regular | font-normal | 400 |
| Medium | font-medium | 500 |
| Bold | font-bold | 700 |

**Font declarations:**

```css
@font-face {
  font-family: 'Styrene A';
  src: url('/fonts/styrene-a/styrene-a-light.woff2') format('woff2');
  font-weight: 300;
  font-style: normal;
  font-display: swap;
}

@font-face {
  font-family: 'Styrene A';
  src: url('/fonts/styrene-a/styrene-a-regular.woff2') format('woff2');
  font-weight: 400;
  font-style: normal;
  font-display: swap;
}

@font-face {
  font-family: 'Styrene A';
  src: url('/fonts/styrene-a/styrene-a-medium.woff2') format('woff2');
  font-weight: 500;
  font-style: normal;
  font-display: swap;
}

@font-face {
  font-family: 'Avenir';
  src: url('/fonts/avenir/avenir-light.woff2') format('woff2');
  font-weight: 300;
  font-style: normal;
  font-display: swap;
}
```

**Fluid typography:**

```css
@layer base {
  html, body {
    font-size: max(0.988vw, 16px);
  }
}
```

### 2.3 Spacing & Layout

Extract spacing values from Figma and map to Tailwind:

```css
@theme {
  /* Border Radius */
  --radius-xs: 0.25rem;    /* 4px */
  --radius-l: 1rem;        /* 16px */
  --radius-pill: 999px;    /* Pill shape */

  /* Shadows (if defined in Figma) */
  --shadow-card: 0 4px 6px -1px oklch(0 0 0 / 0.1);
  --shadow-lg: 0 10px 15px -3px oklch(0 0 0 / 0.1);
}
```

## Phase 3: Tailwind v4 Configuration

### 3.1 Project Setup

**Nuxt integration:**

```typescript
// nuxt.config.ts
import tailwindcss from '@tailwindcss/vite';

export default defineNuxtConfig({
  vite: {
    plugins: [tailwindcss()],
  },
  css: ['./app/assets/css/main.css'],
  modules: [
    '@nuxt/fonts',
    '@nuxt/icon',
    '@nuxt/image',
  ],
});
```

**Package dependencies:**

```json
{
  "dependencies": {
    "tailwindcss": "^4.3.0",
    "@tailwindcss/vite": "^4.3.0",
    "tailwind-merge": "^3.6.0",
    "cva": "^0.0.0",
    "clsx": "^2.1.1"
  },
  "devDependencies": {
    "culori": "^4.0.0"
  }
}
```

### 3.2 Main CSS Entry

```css
/* app/assets/css/main.css */
@import 'tailwindcss';
@import './design-system.css';

@layer base {
  html, body {
    font-size: max(0.988vw, 16px);
  }

  html {
    scroll-behavior: smooth;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
  }
}
```

### 3.3 Design System CSS

```css
/* app/assets/css/design-system.css */

/* Font Declarations */
@font-face {
  font-family: 'Styrene A';
  src: url('/fonts/styrene-a/styrene-a-regular.woff2') format('woff2');
  font-weight: 400;
  font-style: normal;
  font-display: swap;
}

@font-face {
  font-family: 'Avenir';
  src: url('/fonts/avenir/avenir-light.woff2') format('woff2');
  font-weight: 300;
  font-style: normal;
  font-display: swap;
}

/* Theme Configuration */
@theme {
  /* Font Families */
  --font-sans: 'Avenir', sans-serif;
  --font-primary: 'Styrene A', sans-serif;
  --font-secondary: 'Avenir', sans-serif;

  /* Brand Colors */
  --color-brand-primary: oklch(0.2389 0.0436 243.5126);
  --color-brand-secondary-warm: oklch(0.9459 0.0076 61.45);
  --color-brand-secondary-tan: oklch(0.6382 0.0402 57.6847);
  --color-brand-primary-muted-0: oklch(0.5095 0.0244 241.7231);
  --color-brand-primary-muted-1: oklch(0.5832 0.021 243.184);
  --color-brand-primary-muted-3: oklch(0.7822 0.0088 236.5975);

  /* Neutral Colors */
  --color-white: oklch(1 0 0);
  --color-neutral-secondary: oklch(0.3829 0 0);
  --color-neutral-tertiary: oklch(0.2478 0 0 / 0.3216);
  --color-neutral-50: oklch(0.9851 0 0);
  --color-line-icon: oklch(0.8643 0.0244 270.3107);

  /* Accent Colors */
  --color-tango: oklch(0.7588 0.167 55.5274);

  /* Opacity Colors */
  --color-opaque-primary: oklch(0 0 0 / 0.2);
  --color-opaque-primary-inv: oklch(1 0 0 / 0.2);

  /* CTA Colors */
  --color-cta-dark: #0b202f;
  --color-cta-dark-hover: #1a3a4f;
  --color-cta-dark-pressed: #0a1c2a;
  --color-cta-light-hover: #f5f5f5;
  --color-cta-light-pressed: #e8e8e8;

  /* Additional UI Colors */
  --color-border-light: oklch(0.8805 0.0043 -123.5);
  --color-brand-gold: oklch(0.6427 0.0491 84.42);
  --color-neutral-dark: oklch(0.2002 0 89.88);
  --color-text-muted: oklch(0.15 0 89.88 / 0.4);

  /* Typography Scale — standard Tailwind text sizes */
  --text-xs: 0.75rem;
  --text-xs--line-height: 1rem;

  --text-sm: 0.875rem;
  --text-sm--line-height: 1.375rem;

  --text-base: 1rem;
  --text-base--line-height: 1.5rem;

  --text-lg: 1.125rem;
  --text-lg--line-height: 1.625rem;

  --text-xl: 1.25rem;
  --text-xl--line-height: 1.625rem;

  --text-2xl: 1.375rem;
  --text-2xl--line-height: 1.625rem;
  --text-2xl--letter-spacing: -0.0133em;

  --text-3xl: 1.75rem;
  --text-3xl--line-height: 2rem;

  --text-4xl: 2rem;
  --text-4xl--line-height: 2.25rem;

  --text-5xl: 2.5rem;
  --text-5xl--line-height: 2.75rem;

  --text-6xl: 3rem;
  --text-6xl--line-height: 3.25rem;
  --text-6xl--letter-spacing: -0.0133em;

  /* Border Radius */
  --radius-xs: 0.25rem;
  --radius-l: 1rem;
  --radius-pill: 999px;
}

/* Custom Utilities */
@utility font-primary {
  font-family: var(--font-primary);
}

@utility font-secondary {
  font-family: var(--font-secondary);
}

@utility text-eyebrow {
  font-size: var(--text-xs);
  line-height: var(--text-xs--line-height);
  letter-spacing: 0.32em;
  font-family: var(--font-primary);
  text-transform: uppercase;
  font-weight: 300;
}

@utility container-site {
  max-width: 95rem;
  margin-left: auto;
  margin-right: auto;
  padding-left: 1.5rem;
  padding-right: 1.5rem;
}

@utility btn-primary {
  background-color: var(--color-cta-dark);
  color: var(--color-white);
  padding: 0.75rem 1.5rem;
  border-radius: var(--radius-pill);
  font-family: var(--font-primary);
  font-size: var(--text-base);
  transition: background-color 0.2s ease;

  &:hover {
    background-color: var(--color-cta-dark-hover);
  }

  &:active {
    background-color: var(--color-cta-dark-pressed);
  }
}

@utility card-base {
  background-color: var(--color-white);
  border-radius: var(--radius-l);
  overflow: hidden;
}

@utility scrollbar-hide {
  -ms-overflow-style: none;
  scrollbar-width: none;
  &::-webkit-scrollbar {
    display: none;
  }
}
```

## Phase 4: Component Variants (CVA)

### 4.1 Setup

```bash
npm install cva tailwind-merge clsx
```

```typescript
// utils/cn.ts
import { type ClassValue, clsx } from 'clsx';
import { twMerge } from 'tailwind-merge';

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}
```

### 4.2 Button Component Example

```vue
<script setup lang="ts">
import { cva, type VariantProps } from 'cva';
import { cn } from '~/utils/cn';

const buttonVariants = cva(
  'inline-flex items-center justify-center rounded-pill font-primary transition-colors duration-200',
  {
    variants: {
      variant: {
        primary: 'bg-cta-dark text-white hover:bg-cta-dark-hover active:bg-cta-dark-pressed',
        secondary: 'bg-white text-brand-primary border border-brand-primary hover:bg-cta-light-hover',
        ghost: 'bg-transparent text-brand-primary hover:bg-cta-light-hover',
      },
      size: {
        sm: 'h-9 px-3 text-sm',
        md: 'h-11 px-6 text-base',
        lg: 'h-14 px-8 text-lg',
      },
    },
    defaultVariants: {
      variant: 'primary',
      size: 'md',
    },
  }
);

type ButtonVariants = VariantProps<typeof buttonVariants>;

interface Props {
  variant?: ButtonVariants['variant'];
  size?: ButtonVariants['size'];
  class?: string;
}

const props = withDefaults(defineProps<Props>(), {
  variant: 'primary',
  size: 'md',
});
</script>

<template>
  <button :class="cn(buttonVariants({ variant, size }), props.class)">
    <slot />
  </button>
</template>
```

### 4.3 Mapping Figma States

```
Figma Structure:
├── Button / Primary / Default
├── Button / Primary / Hover
├── Button / Primary / Pressed
├── Button / Primary / Disabled
└── Button / Secondary / ...
```

Map to CVA variants with compound variants for state combinations.

## Phase 5: QA & Validation

### 5.1 Create Test Page

```vue
<!-- pages/design-system.vue -->
<template>
  <div class="container-site py-12 space-y-16">
    <section id="colors">
      <h2 class="text-3xl font-primary mb-8">Colors</h2>
      <div class="grid grid-cols-4 gap-4">
        <div class="space-y-2">
          <div class="h-24 rounded-lg bg-brand-primary" />
          <p class="text-sm">Brand Primary</p>
        </div>
        <!-- More color swatches -->
      </div>
    </section>

    <section id="typography">
      <h2 class="text-3xl font-primary mb-8">Typography</h2>
      <div class="space-y-4">
        <p class="text-6xl font-primary">Display Text</p>
        <p class="text-6xl font-primary">Heading 3XL</p>
        <p class="text-base font-secondary">Paragraph text</p>
      </div>
    </section>
  </div>
</template>
```

### 5.2 Screenshot Comparison

**Capture Figma screenshots:**
- Use `figma-dev-mode_get_screenshot` on design system pages
- Save as `figma-screenshots/colors.png`, `figma-screenshots/typography.png`

**Capture implementation screenshots:**
```typescript
// Using Playwright
import { chromium } from 'playwright';

async function capture() {
  const browser = await chromium.launch();
  const page = await browser.newPage();
  await page.goto('http://localhost:3000/design-system');
  
  await page.screenshot({
    path: 'qa-screenshots/implementation/colors.png',
    fullPage: false
  });
  
  await browser.close();
}
```

**Visual comparison checklist:**
- [ ] Colors match Figma hex values (converted to OKLCH)
- [ ] Typography: font family, weight, size, line height, letter spacing
- [ ] Spacing: padding, margin, gap values
- [ ] Border radius matches
- [ ] Shadows and effects match
- [ ] Responsive behavior
- [ ] Accessibility: color contrast ratios

### 5.3 Style Verification

```javascript
// In browser console or Playwright
const element = document.querySelector('.btn-primary');
const styles = window.getComputedStyle(element);

console.log({
  backgroundColor: styles.backgroundColor,
  color: styles.color,
  fontSize: styles.fontSize,
  fontFamily: styles.fontFamily,
  padding: styles.padding,
  borderRadius: styles.borderRadius,
});
```

## Output Structure

```
app/assets/css/
├── main.css              # Entry point
├── design-system.css     # All tokens and utilities
├── ckeditor.css          # Rich text editor styles (if needed)
└── fonts/
    ├── avenir/
    └── styrene-a/

utils/
└── cn.ts                 # Class merging utility
```

## Token Naming Convention

| Figma Pattern | CSS Custom Property | Tailwind Class |
|---------------|---------------------|----------------|
| `Brand/primary/base` | `--color-brand-primary` | `bg-brand-primary`, `text-brand-primary` |
| `Brand/secondary/base` | `--color-brand-secondary` | `bg-brand-secondary` |
| `Fill/neutral-primary` | `--color-neutral-primary` | `text-neutral-primary` |
| `Heading/size/M` | `--text-xl` | `text-xl` |
| `Paragraph/size/S` | `--text-sm` | `text-sm` |
| `Label/size/M` | `--text-base` | `text-base` |
| `Display/size/M` | `--text-6xl` | `text-6xl` |
| `Radius/L` | `--radius-l` | `rounded-l` |

## Rules

1. **Never use arbitrary values** — No `text-[#71717A]`, `gap-[5px]`
2. **All colors in OKLCH** — Convert hex to OKLCH before using
3. **Semantic naming** — Use purpose-based names, not color-based
4. **CSS-first configuration** — No `tailwind.config.js`, use `@theme` block
5. **Document decisions** — Comment why tokens are mapped certain ways
6. **Preserve existing** — Don't break existing design tokens without approval
7. **Use subagents** — Delegate extraction tasks to preserve context

## Questions to Ask

Before starting, clarify:

1. Is this a new design system or updating existing?
2. Should we preserve existing tokens or replace entirely?
3. Are there dark mode / theme variants in Figma?
4. Which pages in Figma contain the design system?
5. Are there responsive breakpoints defined in Figma?

## Deliverables

After implementation, report:

- Design tokens created/updated
- Colors converted to OKLCH
- Typography scale configured
- Custom utilities added
- Component variants created
- Files changed
- Screenshots compared (Figma vs Implementation)
- Remaining discrepancies
