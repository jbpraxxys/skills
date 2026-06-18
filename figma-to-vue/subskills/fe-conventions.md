# Frontend Conventions Subskill

## Purpose

Defines all coding conventions, styling rules, and patterns for Vue/Nuxt components in the Filinvest project.

---

## Tech Stack Summary

| Technology | Version / Details |
|------------|-------------------|
| **Framework** | Nuxt 4.4.6 (Vue 3.5.34) |
| **Language** | TypeScript (strict mode) |
| **Styling** | Tailwind CSS v4 (`@tailwindcss/vite` plugin) |
| **API Style** | Composition API with `<script setup lang="ts">` |
| **Build Tool** | Vite |
| **Images** | `@nuxt/image` (provider: `none`) |
| **Icons** | `@nuxt/icon` + `lucide-vue-next` |
| **Fonts** | `@nuxt/fonts` (Avenir, Styrene A) |
| **Carousels** | Swiper.js (`swiper/vue`) |
| **Animation** | GSAP with `useAnimation()` composable |

---

## Script Structure Convention

Components MUST follow this flow structure (unless a specific flow is needed):

```vue
<script lang="ts" setup>
    // 1. IMPORTS (NO section comment for imports)
    import type { FrameNameData } from '~/types/modular/category';
    import { SomeIcon } from '@lucide/vue';

    /*==============================================================================
     * PROPS
     *============================================================================*/
    const props = withDefaults(defineProps<{ data: FrameNameData }>(), {
        // defaults only for optional props
    });

    /*==============================================================================
     * EMITS
     *============================================================================*/
    defineEmits<{
        click: [event: MouseEvent];
        'update:modelValue': [value: string];
    }>();

    /*==============================================================================
     * COMPOSABLES
     *============================================================================*/
    const { getAssetUrl } = useAssetUrl();
    const { formatDate } = useDateFormat();

    /*==============================================================================
     * STATE (REFS, REACTIVE, ETC)
     *============================================================================*/
    const isOpen = ref(false);
    const formData = reactive({ name: '', email: '' });

    /*==============================================================================
     * COMPUTED
     *============================================================================*/
    const hasContent = computed(() => !!(props.data.title || props.data.description));
    const classes = computed(() => twMerge(base, conditional));

    /*==============================================================================
     * UTILS
     *============================================================================*/
    const formatName = (name: string) => name.trim().toUpperCase();

    /*==============================================================================
     * ACTIONS
     *============================================================================*/
    const handleSubmit = async () => { ... };

    /*==============================================================================
     * VALIDATION
     *============================================================================*/
    // Form validation logic here

    /*==============================================================================
     * ANIMATIONS
     *============================================================================*/
    useAnimation('.fade-up');

    /*==============================================================================
     * LIFECYCLE / WATCHERS
     *============================================================================*/
    onMounted(() => { ... });
    watch(() => props.data, () => { ... });
</script>
```

**Rules:**
- Use semicolons everywhere
- Use 4 spaces for indentation
- Omit any section that has no code (no empty comment blocks)
- No explicit `import { computed, ref } from 'vue'` — Nuxt 4 auto-imports Vue APIs

---

## Tailwind CSS v4 Rules

### Rule 1: No Arbitrary Values

**❌ Bad:**
```html
<div class="gap-[5px] text-[#71717A] leading-[22px] p-[13px]">
```

**✅ Good:**
```html
<div class="gap-1.5 text-neutral-secondary leading-relaxed p-3">
```

**When no standard value exists:**
1. Check if the design system has a custom utility in `design-system.css`
2. If not, add a new `@utility` rule in `design-system.css`
3. Never use arbitrary values in component code

### Rule 2: No Hex Codes in Classes

**❌ Bad:**
```html
<div class="text-[#0b202f] bg-[#f5f5f5]">
```

**✅ Good:**
```html
<div class="text-brand-primary bg-neutral-50">
```

**Process for new colors:**
1. Check if color exists in `app/assets/css/design-system.css`
2. If not, add to the `@theme` block as a CSS variable
3. Name it descriptively: `--color-brand-gray`, `--color-accent-blue`
4. Convert hex to OKLCH format before adding

### Rule 3: OKLCH Color Format (MANDATORY)

**❌ Bad:**
```css
--color-neutral-8: #8d8d8d;
```

**✅ Good:**
```css
--color-neutral-8: oklch(0.5517 0.0138 285.94);
```

**Conversion process:**
1. Take the hex value from Figma
2. Convert to OKLCH using a converter tool
3. Add to `design-system.css` in the `@theme` block
4. Use the semantic class name in components

### Rule 4: Use Existing Design System Utilities

The project has custom utilities defined in `design-system.css`. Check these first:

| Utility | Purpose | Example |
|---------|---------|---------|
| `text-6xl` / `text-5xl` | Large display/heading text | `<h1 class="text-6xl font-primary">` |
| `text-xl` | Medium heading | `<h2 class="text-xl font-primary">` |
| `text-base` | Body text | `<p class="text-base font-secondary">` |
| `text-eyebrow` | Small uppercase label | `<span class="text-eyebrow">` |
| `container-site` | Max-width container | `<div class="container-site">` |
| `card-base` | Base card styles | `<div class="card-base">` |
| `btn-primary` | Primary button | `<button class="btn-primary">` |
| `gradient-overlay-bottom` | Bottom fade overlay | `<div class="gradient-overlay-bottom">` |

### Rule 5: Responsive Breakpoints

Standard breakpoints: `sm:`, `md:`, `lg:`, `xl:`

Common patterns:
```html
<!-- Padding scaling -->
<div class="px-5 md:px-10 lg:px-20">

<!-- Typography scaling -->
<h2 class="text-xl md:text-5xl font-primary">

<!-- Layout stacking -->
<div class="flex flex-col lg:flex-row gap-6">

<!-- Grid responsive -->
<div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3">
```

### Rule 6: Dynamic Classes

Use `:class` with arrays for conditional classes:

```vue
<template>
    <div :class="[
        'base-class',
        'always-present-class',
        conditional ? 'conditional-class' : 'alternative-class',
        props.light ? 'text-white' : 'text-brand-primary',
    ]">
</template>
```

**❌ Don't use separate `:class` attributes:**
```html
<!-- DON'T DO THIS -->
<div class="base-class" :class="{ 'conditional': isTrue }">
```

**✅ Use single `:class` array:**
```html
<!-- DO THIS -->
<div :class="['base-class', isTrue && 'conditional']">
```

---

## Asset Handling Rules

### Rule 1: Use `nuxt-img` Component

**❌ Bad:**
```html
<img src="/path/to/image.jpg" alt="Description" />
```

**✅ Good:**
```html
<nuxt-img
    :src="getAssetUrl(data.image)"
    alt="Description"
    class="w-full h-full object-cover"
/>
```

### Rule 2: Use `useAssetUrl()` Composable

```vue
<script lang="ts" setup>
    const { getAssetUrl } = useAssetUrl();
</script>
```

The `getAssetUrl()` function:
- Prepends the storage URL prefix
- Strips query parameters (signed S3 tokens)
- Handles both string paths and object paths
- Returns empty string for null/undefined

### Rule 3: Image Placeholder Strategy

When Figma images are not available:

1. **For development/testing:** Use lorem picsum
   ```
   https://picsum.photos/seed/<unique-id>/<width>/<height>
   ```

2. **For sample pages:** Use static images from `public/images/`

3. **For production:** Use `getAssetUrl(data.image)` with CMS paths

---

## Icon System

Use `lucide-vue-next` icons:

```vue
<script lang="ts" setup>
    import { ArrowUpRight, ChevronLeft, Mail } from '@lucide/vue';
</script>

<template>
    <arrow-up-right class="w-5 h-5" />
    <chevron-left class="w-4 h-4" />
</template>
```

**Rules:**
- Import icons explicitly from `@lucide/vue`
- Use `w-*` and `h-*` classes for sizing
- Icons are components, not `<img>` tags

---

## Form Field Patterns

Use existing shared field components:

```vue
<template>
    <base-input
        id="email"
        v-model="formData.email"
        label="Email Address"
        placeholder="Enter email here"
        type="email"
        :error="formErrors.email"
        required
    />

    <base-select
        id="subject"
        v-model="formData.subject"
        label="Subject"
        :options="subjectOptions"
        :error="formErrors.subject"
    />

    <base-textarea
        id="message"
        v-model="formData.message"
        label="Message"
        placeholder="Enter your message..."
        rows="5"
    />

    <base-button variant="primary" color="dark" @click="handleSubmit">
        Submit
    </base-button>
</template>
```

**Available field components:**
- `BaseInput.vue` — Text input with validation, icons
- `BaseMobileInput.vue` — Phone number input
- `BaseSelect.vue` — Dropdown select
- `BaseTextarea.vue` — Multi-line text

---

## Animation Patterns

Use the `useAnimation()` composable for scroll-triggered animations:

```vue
<script lang="ts" setup>
    // Default: fade-up on scroll
    useAnimation('.fade-up');

    // With options
    useAnimation('.fade-up', { trigger: 'mount' });
</script>

<template>
    <section class="fade-up opacity-0 translate-y-[60px]">
        <!-- Content will fade up on scroll -->
    </section>
</template>
```

**Animation classes to use:**
- `.fade-up` — Fade in + translate Y
- Add `opacity-0 translate-y-[60px]` to animated elements
- The composable adds the transition classes automatically

---

## TypeScript Patterns

### Props Interface Pattern

```ts
// app/types/modular/category.ts
export interface FrameNameData {
    heading?: string;
    description?: string;
    image?: string;
    items?: FrameItemData[];
    hasBackground?: boolean;
}

export interface FrameItemData {
    title: string;
    description?: string;
    image: string;
    link?: string;
}
```

### Component Props with Defaults

```vue
<script lang="ts" setup>
    import type { FrameNameData } from '~/types/modular/category';

    const props = withDefaults(defineProps<{
        data: FrameNameData;
        light?: boolean;
    }>(), {
        light: false,
    });
</script>
```

**Important:** `withDefaults()` only applies when prop value is `undefined`, NOT `null`. If parent passes `null` (from `useFetch` before resolution), defaults will NOT apply.

### Event Typing

```vue
<script lang="ts" setup>
    defineEmits<{
        click: [event: MouseEvent];
        'update:modelValue': [value: string];
        submit: [formData: FormData];
    }>();
</script>
```

---

## Naming Conventions Summary

| Element | Convention | Example |
|---------|-----------|---------|
| Component files | PascalCase | `BlogCard.vue`, `ContactFormFrame.vue` |
| Template tags | Kebab-case | `<blog-card>`, `<contact-form-frame>` |
| Props interfaces | PascalCase + Data suffix | `BlogCardData`, `ContactFormFrameData` |
| Composables | camelCase + use prefix | `useAssetUrl`, `useAnimation` |
| Utility functions | camelCase | `formatDate`, `validateEmail` |
| Constants | UPPER_SNAKE_CASE | `API_BASE_URL` |
| CSS variables | kebab-case | `--color-brand-primary` |
| Tailwind utilities | kebab-case | `text-xl`, `text-6xl`, `bg-brand-primary` |

---

## Code Quality Checklist

Before marking a component complete, verify:

- [ ] Uses `<script lang="ts" setup>` (not Options API)
- [ ] Has proper section comment blocks
- [ ] Uses 4-space indentation
- [ ] Uses semicolons
- [ ] No arbitrary Tailwind values
- [ ] No hex codes in class attributes
- [ ] Uses existing design system utilities
- [ ] Uses `nuxt-img` for images
- [ ] Uses `getAssetUrl()` for asset paths
- [ ] Props are typed from `~/types/modular/`
- [ ] Single `data` prop for modular components
- [ ] Kebab-case in templates
- [ ] No scoped CSS blocks
- [ ] No inline styles (except dynamic runtime values)
- [ ] Proper responsive breakpoints
- [ ] Accessible attributes (alt, aria-label, etc.)
