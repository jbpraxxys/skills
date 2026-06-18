# Component Architecture Subskill

## Purpose

Defines how to structure, name, and organize Vue components when translating Figma designs to code.

---

## Component Directory Structure

The project uses a 3-tier component hierarchy:

```
app/components/
├── layout/          # Global shell components (always visible)
│   ├── TheHeader.vue
│   ├── TheFooter.vue
│   └── NavDropdown.vue
│
├── modular/         # CMS-driven page sections (dynamic content)
│   ├── banners/     # Hero sections, image sliders
│   ├── cards/       # Card grid sections
│   ├── description/ # Text/image split layouts
│   ├── sliders/     # Carousel sections
│   └── unique/      # One-off sections (forms, FAQs, etc.)
│
└── shared/          # Reusable atomic components
    ├── buttons/
    ├── cards/
    ├── fields/
    ├── skeletons/
    └── [other utility components]
```

---

## Component Classification Rules

### Rule 1: Layout Components
**Location:** `components/layout/`
**Purpose:** Global shell UI that persists across pages
**Examples:**
- `TheHeader.vue` — Site header with navigation
- `TheFooter.vue` — Site footer
- `NavDropdown.vue` — Navigation dropdown wrapper

**Naming:** Prefix with `The` for singletons (TheHeader, TheFooter).

### Rule 2: Modular Components (Sections/Frames)
**Location:** `components/modular/<category>/`
**Purpose:** CMS-configurable page sections that receive data via props
**Pattern:** Named with `Frame` suffix for sections, or descriptive name for sub-types

| Category | Use Case | Examples |
|----------|----------|----------|
| `banners/` | Hero/top-of-page sections | `FullBannerImageSlider.vue`, `BlogBanner.vue` |
| `cards/` | Card grid/list sections | `CardGalleryFrame.vue`, `CardServiceFeatureFrame.vue` |
| `description/` | Text/image split layouts | `DescriptionSplitFrame.vue`, `DescriptionAppPreviewFrame.vue` |
| `sliders/` | Carousel sections | `BlogSlider.vue`, `ExperienceSlider.vue` |
| `unique/` | One-off complex sections | `ContactFormFrame.vue`, `FaqFrame.vue`, `CareersFrame.vue` |

**Naming Convention:**
- Section components: `<Purpose><Type>Frame.vue` (e.g., `DescriptionSplitFrame.vue`)
- Sub-items within sections: `<Purpose>Item.vue` (e.g., `FaqItem.vue`)

### Rule 3: Shared Components
**Location:** `components/shared/<category>/`
**Purpose:** Reusable atomic components used by multiple modular components

| Category | Examples | Reuse Pattern |
|----------|----------|---------------|
| `buttons/` | `BaseButton.vue`, `NavigationButton.vue`, `RedirectionButton.vue` | Used by forms, sliders, CTAs |
| `cards/` | `BlogCard.vue`, `ExperienceCard.vue`, `GalleryCard.vue` | Used inside slider/carousel components |
| `fields/` | `BaseInput.vue`, `BaseSelect.vue`, `BaseTextarea.vue` | Used by form frames |
| `skeletons/` | `BlogPostSkeleton.vue`, `BlogSliderSkeleton.vue` | Loading states for async data |

**Naming Convention:**
- Base components: `Base<Name>.vue` (e.g., `BaseButton.vue`, `BaseInput.vue`)
- Specialized components: `<Purpose><Type>.vue` (e.g., `BlogCard.vue`, `NavigationButton.vue`)

---

## Component Boundary Decision Framework

When analyzing a Figma design, use this flowchart to decide what becomes a component:

```
Visual Element
    |
    v
Is it a global UI element (header, footer, nav)?
    |---YES--> components/layout/
    | NO
    v
Is it a complete page section with distinct data?
    |---YES--> components/modular/<category>/
    | NO
    v
Does it repeat 3+ times within a section?
    |---YES--> Extract to components/shared/<category>/
    | NO
    v
Is it a simple atomic element (button, input, icon)?
    |---YES--> components/shared/<category>/
    | NO
    v
Is it complex and section-specific?
    |---YES--> Inline within the modular component
```

---

## Sub-component Extraction Criteria

Extract a sub-component when ANY of these are true:

1. **Repetition** — Element repeats 3+ times (cards, list items, slides)
2. **Variability** — Element has props that vary per instance
3. **Reusability** — Element could be used in other sections
4. **Complexity** — Element's template exceeds ~50 lines
5. **Independence** — Element can render independently with its own data

**Example:**

```
Figma: Restaurant Listing Section
├── Section Title: "Our Restaurants"
├── Section Description: "Discover culinary excellence..."
└── Restaurant Cards (x6)
    ├── Image
    ├── Name
    ├── Cuisine Type
    ├── Rating
    └── CTA Button

Decision:
- Restaurant Listing Section → components/modular/cards/RestaurantListFrame.vue
- Restaurant Card (repeats 6x) → components/shared/cards/RestaurantCard.vue
- CTA Button (reusable) → Use existing components/shared/buttons/BaseButton.vue
```

---

## Props Architecture

### Modular Frame Props Pattern

Every modular component receives exactly ONE prop:

```ts
// app/types/modular/restaurant.ts
export interface RestaurantListFrameData {
    heading?: string;
    description?: string;
    restaurants?: RestaurantCardData[];
}

export interface RestaurantCardData {
    id: number;
    name: string;
    cuisine: string;
    image: string;
    rating?: number;
    link?: string;
}
```

```vue
<!-- components/modular/cards/RestaurantListFrame.vue -->
<script lang="ts" setup>
    import type { RestaurantListFrameData } from '~/types/modular/restaurant';
    import RestaurantCard from '@/components/shared/cards/RestaurantCard.vue';

    const props = defineProps<{ data: RestaurantListFrameData }>();
</script>

<template>
    <section class="py-10 md:py-16 lg:py-20 px-5 md:px-10 lg:px-20 bg-white">
        <div class="md:container-site flex flex-col items-center gap-10">
            <h2 v-if="data.heading" class="text-xl md:text-5xl font-primary text-brand-primary">
                {{ data.heading }}
            </h2>
            <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
                <restaurant-card
                    v-for="restaurant in data.restaurants"
                    :key="restaurant.id"
                    :data="restaurant"
                />
            </div>
        </div>
    </section>
</template>
```

### Shared Component Props Pattern

```vue
<!-- components/shared/cards/RestaurantCard.vue -->
<script lang="ts" setup>
    import type { RestaurantCardData } from '~/types/modular/restaurant';

    const props = defineProps<{ data: RestaurantCardData }>();
</script>

<template>
    <div class="card-base flex flex-col gap-4">
        <div class="aspect-[4/3] overflow-hidden rounded-xl">
            <nuxt-img :src="getAssetUrl(data.image)" :alt="data.name" class="w-full h-full object-cover" />
        </div>
        <div class="flex flex-col gap-2 p-4">
            <h3 class="text-xl font-primary text-brand-primary">{{ data.name }}</h3>
            <p class="text-base font-secondary text-neutral-secondary">{{ data.cuisine }}</p>
        </div>
    </div>
</template>
```

---

## Component Mapping for CMS Integration

When a component is CMS-driven, it must be registered in the component map:

```ts
// app/composables/useComponentMap.ts
export function useComponentMap() {
    const componentMap: Record<string, string> = {
        'full_banner_image_slider': 'FullBannerImageSlider',
        'blog_slider': 'BlogSlider',
        'card_service_feature': 'CardServiceFeatureFrame',
        'restaurant_list': 'RestaurantListFrame', // <-- Add new component here
    };

    function getAdapterComponent(sectionType: string): string {
        return componentMap[sectionType] ?? 'GenericFrame';
    }

    return { componentMap, getAdapterComponent };
}
```

**Important:** The `section_type` from CMS must match the key in `componentMap`. The value is the PascalCase component name (auto-resolved by Nuxt).

---

## File Creation Checklist

When creating a new modular component, create these files:

- [ ] `app/components/modular/<category>/<ComponentName>Frame.vue`
- [ ] `app/types/modular/<category>.ts` (if new category)
- [ ] `app/components/shared/<category>/<SubComponent>.vue` (if sub-components needed)
- [ ] `app/pages/sample/<component-name>.vue` (sample page for testing)
- [ ] Update `app/composables/useComponentMap.ts` (if CMS-driven)

---

## Common Anti-Patterns to Avoid

### ❌ Flat Props
```vue
<!-- DON'T DO THIS -->
<script setup>
const props = defineProps<{
    title: string;
    description: string;
    image: string;
    items: Array<{ ... }>;
}>();
</script>
```

### ✅ Single Data Prop
```vue
<!-- DO THIS -->
<script setup>
const props = defineProps<{ data: FrameNameData }>();
</script>
```

### ❌ Scoped CSS
```vue
<!-- DON'T DO THIS -->
<style scoped>
.custom-class { color: red; }
</style>
```

### ✅ Tailwind Utilities
```vue
<!-- DO THIS -->
<template>
    <div class="text-brand-primary">...</div>
</template>
```

### ❌ Individual Image Tags
```vue
<!-- DON'T DO THIS -->
<img src="/path/to/image.jpg" />
```

### ✅ Nuxt Image with Asset URL
```vue
<!-- DO THIS -->
<nuxt-img :src="getAssetUrl(data.image)" alt="Description" />
```
