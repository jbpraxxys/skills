# API Readiness Subskill

## Purpose

Defines how to handle components when the backend API is not yet ready, including sample data patterns, documentation templates, and migration strategies.

---

## API Context

The Filinvest project is a **headless frontend** that consumes an external Laravel/CMS backend. There are no local Nitro API routes (`server/api/`). All data comes from:

```
GET {apiBaseUrl}/api/v1/dynamic-pages/{slug}
```

Response structure:
```json
{
    "success": true,
    "data": {
        "title": "Page Title",
        "slug": "page-slug",
        "sections": [
            {
                "section_type": "full_banner_image_slider",
                "section_name": "Hero Banner",
                "data": { ... },
                "sort_order": 1,
                "is_active": true
            }
        ]
    }
}
```

---

## Scenario 1: API Already Exists

### When to use this scenario
- Backend endpoint is deployed and documented
- CMS fields are configured
- Data shape is known

### Implementation

1. **Create the TypeScript interface** matching the API response:

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
    description?: string;
    location?: string;
    hours?: string;
}
```

2. **Create the component** using the interface:

```vue
<script lang="ts" setup>
    import type { RestaurantListFrameData } from '~/types/modular/restaurant';

    const props = defineProps<{ data: RestaurantListFrameData }>();
</script>
```

3. **Register in component map**:

```ts
// app/composables/useComponentMap.ts
const componentMap: Record<string, string> = {
    // ... existing mappings
    'restaurant_list': 'RestaurantListFrame',
};
```

4. **Test with real data** by navigating to a CMS page that includes the section.

---

## Scenario 2: API Does NOT Exist (MOST COMMON)

### When to use this scenario
- Backend endpoint is not ready
- CMS fields are not configured
- Data shape is not finalized
- You're building the frontend ahead of the backend

### Implementation Strategy

#### Step 1: Create Component with Static Sample Data

```vue
<!-- components/modular/cards/RestaurantListFrame.vue -->
<script lang="ts" setup>
    import type { RestaurantListFrameData } from '~/types/modular/restaurant';
    import RestaurantCard from '@/components/shared/cards/RestaurantCard.vue';

    const props = withDefaults(defineProps<{
        data?: RestaurantListFrameData;
    }>(), {
        data: () => sampleData,
    });

    // TODO: Remove sampleData when API is ready
    // Expected API shape documented below
    const sampleData: RestaurantListFrameData = {
        heading: 'Our Restaurants',
        description: 'Discover culinary excellence across our properties.',
        restaurants: [
            {
                id: 1,
                name: 'Enye by Chele Gonzalez',
                cuisine: 'Spanish Cuisine',
                image: '/images/restaurants/enye.jpg',
                rating: 4.8,
                link: '/culinary/enye',
                description: 'Michelin-Selected Restaurant 2026',
                location: 'Crimson Resort & Spa, Mactan',
                hours: 'Dinner: Tue-Sun, 6pm-10:30pm',
            },
            {
                id: 2,
                name: 'Azure Beach Club',
                cuisine: 'Modern Mediterranean',
                image: '/images/restaurants/azure.jpg',
                rating: 4.5,
                link: '/culinary/azure',
                description: 'Beachfront dining with stunning sunsets',
                location: 'Crimson Resort & Spa, Boracay',
                hours: 'Lunch & Dinner: Daily, 11am-11pm',
            },
            // Add more sample items as needed
        ],
    };
</script>

<template>
    <section class="py-10 md:py-16 lg:py-20 px-5 md:px-10 lg:px-20 bg-white">
        <div class="md:container-site flex flex-col items-center gap-10">
            <div v-if="data.heading" class="text-center">
                <h2 class="text-xl md:text-5xl font-primary text-brand-primary">
                    {{ data.heading }}
                </h2>
                <p v-if="data.description" class="text-base font-secondary text-neutral-secondary mt-4">
                    {{ data.description }}
                </p>
            </div>

            <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6 w-full">
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

#### Step 2: Create Sample Page for Testing

```vue
<!-- app/pages/sample/restaurant-list.vue -->
<script lang="ts" setup>
    // Sample page for testing the RestaurantListFrame component
    // TODO: Remove this page when component is integrated into CMS
    import RestaurantListFrame from '@/components/modular/cards/RestaurantListFrame.vue';
</script>

<template>
    <div>
        <restaurant-list-frame />
    </div>
</template>
```

#### Step 3: Document Expected API Shape

Add a comment block at the top of the component file documenting the expected API:

```vue
<script lang="ts" setup>
    /*==============================================================================
     * API DOCUMENTATION
     *============================================================================
     * Expected CMS section_type: "restaurant_list"
     *
     * Expected API Response Shape:
     * {
     *   "section_type": "restaurant_list",
     *   "section_name": "Our Restaurants",
     *   "data": {
     *     "heading": "Our Restaurants",
     *     "description": "Discover culinary excellence...",
     *     "restaurants": [
     *       {
     *         "id": 1,
     *         "name": "Restaurant Name",
     *         "cuisine": "Cuisine Type",
     *         "image": "path/to/image.jpg",
     *         "rating": 4.8,
     *         "link": "/culinary/restaurant-slug",
     *         "description": "Short description",
     *         "location": "Property Name",
     *         "hours": "Operating hours"
     *       }
     *     ]
     *   }
     * }
     *
     * TODO:
     * - [ ] Confirm API endpoint with backend team
     * - [ ] Verify field names match CMS configuration
     * - [ ] Update component map when section_type is confirmed
     * - [ ] Remove sampleData when API is integrated
     *============================================================================*/

    // ... component code
</script>
```

#### Step 4: Use Placeholder Images

When real images are not available, use lorem picsum:

```ts
// For sample data, use picsum with a seed for consistency
const sampleData: RestaurantListFrameData = {
    restaurants: [
        {
            id: 1,
            name: 'Enye',
            image: 'https://picsum.photos/seed/enye/800/600',
            // ...
        },
        {
            id: 2,
            name: 'Azure',
            image: 'https://picsum.photos/seed/azure/800/600',
            // ...
        },
    ],
};
```

---

## Scenario 3: API Partially Ready

### When to use this scenario
- Some fields are available from API
- Other fields need to be mocked
- Component needs to handle both real and fallback data

### Implementation

```vue
<script lang="ts" setup>
    import type { RestaurantListFrameData } from '~/types/modular/restaurant';

    const props = defineProps<{ data: RestaurantListFrameData }>();

    // Merge API data with defaults for missing fields
    const mergedData = computed<RestaurantListFrameData>(() => ({
        heading: props.data.heading ?? 'Our Restaurants',
        description: props.data.description ?? '',
        restaurants: props.data.restaurants?.map(r => ({
            ...r,
            // Fallback for fields not yet in API
            rating: r.rating ?? 0,
            description: r.description ?? 'Description coming soon',
            image: r.image ?? '/images/placeholder-restaurant.jpg',
        })) ?? [],
    }));
</script>
```

---

## Documentation Template

Create a markdown file alongside the component:

```markdown
# Component: RestaurantListFrame

## Purpose
Display a grid of restaurant cards with details, ratings, and links.

## CMS Configuration

### section_type
`restaurant_list`

### Required Fields
| Field | Type | Description |
|-------|------|-------------|
| heading | string | Section title |
| description | string | Section subtitle/description |
| restaurants | array | List of restaurant objects |

### Restaurant Object Fields
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| id | number | Yes | Unique identifier |
| name | string | Yes | Restaurant name |
| cuisine | string | Yes | Cuisine type |
| image | string | Yes | Image path (resolved via getAssetUrl) |
| rating | number | No | Rating out of 5 |
| link | string | No | Detail page URL |
| description | string | No | Short description |
| location | string | No | Property/location name |
| hours | string | No | Operating hours |

## API Endpoint
- **Method:** GET
- **URL:** `/api/v1/dynamic-pages/{slug}`
- **Section Type:** `restaurant_list`

## Sub-components
- `RestaurantCard.vue` (components/shared/cards/)

## Assets Required
- [ ] Restaurant thumbnail images (variable count)
- [ ] Default placeholder image for restaurants without images

## Responsive Behavior
| Breakpoint | Layout | Cards per row |
|------------|--------|---------------|
| Mobile (< 768px) | Single column | 1 |
| Tablet (768px - 1024px) | Two columns | 2 |
| Desktop (> 1024px) | Three columns | 3 |

## Animation
- Cards fade up on scroll (`useAnimation('.fade-up')`)
- Hover: Image scale 1.05x, subtle shadow

## Dependencies
- `useAssetUrl()` composable
- `RestaurantCard.vue` shared component
- Tailwind grid utilities

## Status
- [x] Component created
- [x] TypeScript interfaces defined
- [x] Sample data created
- [x] Sample page created
- [ ] API endpoint confirmed
- [ ] CMS fields configured
- [ ] Component map updated
- [ ] QA passed

## Notes
- Created: 2026-06-17
- Backend contact: [Name]
- Ticket/PR: [Link]
```

---

## Migration Checklist

When the API becomes ready, follow this checklist to migrate from static to dynamic:

1. [ ] Confirm API endpoint returns expected data shape
2. [ ] Remove `sampleData` from component
3. [ ] Update `withDefaults` to remove fallback
4. [ ] Update component to use `props.data` directly (no fallback)
5. [ ] Add component to `useComponentMap.ts`
6. [ ] Verify CMS section_type matches component map key
7. [ ] Test with real API data
8. [ ] Remove sample page (`app/pages/sample/*.vue`)
9. [ ] Update documentation status
10. [ ] Run QA against real data

---

## Common API Patterns

### Data Fetching in Pages

```vue
<!-- app/pages/[...slug].vue -->
<script lang="ts" setup>
    const route = useRoute();
    const slug = computed(() => (route.params.slug as string[])?.[0] ?? 'home');

    const { data: pageData } = await useFetch(
        () => `${apiBaseUrl}/api/v1/dynamic-pages/${slug.value}`,
        {
            transform: transformDynamicPage,
            getCachedData: (key) => {
                const nuxtApp = useNuxtApp();
                return nuxtApp.payload.data[key] || nuxtApp.static.data[key];
            },
        }
    );
</script>
```

### Section Rendering

```vue
<template>
    <div v-if="pageData?.sections?.length">
        <section-adapter
            v-for="section in pageData.sections"
            :key="section.id"
            :component="getAdapterComponent(section.section_type)"
            :data="section.data"
        />
    </div>
</template>
```

---

## Tips for Working Without APIs

1. **Use realistic sample data** — Names, descriptions, and images that match the real content type
2. **Document everything** — Every field assumption should be documented
3. **Make it easy to swap** — Structure code so removing sample data is a single-line change
4. **Test responsive** — Sample data should include enough items to test all breakpoints
5. **Use seeded picsum** — `picsum.photos/seed/<name>` gives consistent images
6. **Keep sample pages** — They serve as documentation and quick tests
7. **Communicate with backend** — Share the expected API shape early
