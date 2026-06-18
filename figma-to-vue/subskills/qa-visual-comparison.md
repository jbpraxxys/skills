# QA & Visual Comparison Subskill

## Purpose

Defines the workflow for quality assurance and visual comparison between Figma designs and implemented Vue/Nuxt components.

---

## QA Philosophy

Visual fidelity is critical. The coded output should match the Figma design within acceptable tolerance. QA is not optional — it's a required phase before marking any component as complete.

**Acceptable tolerance:**
- Layout alignment: ±2px
- Typography: Exact match for size, weight, line-height
- Colors: Exact match (OKLCH values)
- Spacing: ±4px for large gaps, ±2px for small gaps
- Responsive: Must match at all breakpoints (desktop, tablet, mobile)

---

## Phase 1: Build Verification

### Step 1: Start Dev Server

```bash
npm run dev
```

Ensure the dev server starts without errors.

### Step 2: Navigate to Component

Use `chrome-devtools_navigate_page` to navigate to:
- The sample page: `http://localhost:3000/sample/<component-name>`
- Or the actual page where the component is used

### Step 3: Wait for Full Render

Wait for:
- All images to load (or placeholders to show)
- Fonts to load
- Any async data to resolve
- Animations to settle

---

## Phase 2: Screenshot Capture

### Method 1: Chrome DevTools (Recommended)

Use `chrome-devtools_take_screenshot` with options:

```
chrome-devtools_take_screenshot(
    fullPage: true    # Capture entire scrollable page
    OR
    fullPage: false   # Capture current viewport only
)
```

**When to use fullPage:**
- For sections that are taller than viewport
- For comparing entire page layouts

**When to use viewport:**
- For components that fit in viewport
- For above-the-fold comparison

### Method 2: Playwright Browser

Alternative using Playwright:

```
playwright_browser_navigate(url: "http://localhost:3000/sample/component")
playwright_browser_take_screenshot(fullPage: true)
```

### Method 3: Multiple Viewport Sizes

Test responsive breakpoints by resizing:

```
chrome-devtools_resize_page(width: 1440, height: 900)   # Desktop
chrome-devtools_resize_page(width: 768, height: 1024)   # Tablet
chrome-devtools_resize_page(width: 375, height: 812)    # Mobile
```

Take screenshots at each breakpoint for comparison.

---

## Phase 3: Visual Comparison

### Side-by-Side Comparison

1. **Get Figma screenshot** (from Phase 1 of main workflow)
2. **Get coded screenshot** (from Phase 2 above)
3. **Compare visually** using your vision capabilities

### Comparison Checklist

#### Layout & Structure
- [ ] Overall layout matches (grid, flex, positioning)
- [ ] Container widths match (max-width, padding)
- [ ] Element alignment is correct (left, center, right)
- [ ] Grid columns match (1-col mobile, 2-col tablet, 3-col desktop)

#### Typography
- [ ] Font family matches (Styrene A for headings, Avenir for body)
- [ ] Font sizes match at each breakpoint
- [ ] Font weights match (300 light, 400 regular, 500 medium, bold)
- [ ] Line heights match
- [ ] Letter spacing matches (especially for eyebrow text)
- [ ] Text alignment matches
- [ ] Line clamping matches (truncate at same points)

#### Colors
- [ ] Background colors match exactly
- [ ] Text colors match exactly
- [ ] Border colors match
- [ ] Gradient overlays match (check opacity values)
- [ ] Hover states match (if visible in Figma)

#### Spacing
- [ ] Section padding matches (top/bottom, left/right)
- [ ] Gap between elements matches
- [ ] Margin/padding on individual elements matches
- [ ] Card internal padding matches

#### Images & Assets
- [ ] Image aspect ratios match
- [ ] Image positioning matches (object-fit, object-position)
- [ ] Border radius on images matches
- [ ] Overlay effects match (shadows, gradients)
- [ ] Placeholder images have appropriate sizing

#### Interactive Elements
- [ ] Button sizes match
- [ ] Button colors match (primary, outline, text variants)
- [ ] Icon sizes and colors match
- [ ] Link underlines/hover states match
- [ ] Form field heights and borders match

#### Responsive Behavior
- [ ] Mobile layout stacks correctly
- [ ] Tablet layout adjusts properly
- [ ] Desktop layout matches design
- [ ] Font sizes scale appropriately
- [ ] Images resize correctly
- [ ] Navigation/menu works on mobile

---

## Phase 4: Functional Testing

### Console Check

Check browser console for:
- JavaScript errors
- Vue warnings
- Failed network requests
- TypeScript errors (check terminal)

Use:
```
chrome-devtools_list_console_messages
```

### Interaction Testing

Test all interactive elements:
- [ ] Buttons are clickable
- [ ] Links navigate correctly
- [ ] Forms validate properly
- [ ] Carousels/sliders work
- [ ] Hover states activate
- [ ] Mobile menu opens/closes

### Accessibility Check

- [ ] Images have alt text
- [ ] Form inputs have labels
- [ ] Color contrast meets WCAG standards
- [ ] Focus states are visible
- [ ] Semantic HTML elements used (section, article, nav)

---

## Phase 5: Iteration & Fixes

### When Discrepancies Are Found

1. **Document the discrepancy**
   - What element is different
   - Expected value (from Figma)
   - Actual value (from code)
   - Severity: Critical / Major / Minor

2. **Fix the code**
   - Adjust Tailwind classes
   - Update component props
   - Fix responsive breakpoints
   - Adjust spacing values

3. **Re-build and re-screenshot**
   - Make changes
   - Rebuild dev server (if needed)
   - Take new screenshot
   - Re-compare

4. **Repeat until match**

### Common Fixes

| Issue | Likely Cause | Fix |
|-------|-------------|-----|
| Text too large/small | Wrong text utility class | Check `text-xl` vs `text-6xl` (or the mapped size) |
| Wrong color | Using hex instead of token | Use `text-brand-primary` not `text-[#0b202f]` |
| Spacing off | Wrong gap/padding class | Check `gap-6` vs `gap-8`, `py-10` vs `py-16` |
| Image stretched | Wrong aspect ratio | Add `aspect-[4/3]` or `aspect-square` |
| Layout broken | Missing responsive class | Add `md:` or `lg:` prefixes |
| Font wrong | Not using design token | Use `font-primary` or `font-secondary` |

---

## QA Report Template

After QA, generate a brief report:

```markdown
# QA Report: <ComponentName>

## Screenshots
- Figma: [attached]
- Coded (Desktop): [attached]
- Coded (Mobile): [attached]

## Visual Comparison Results

### Layout
- [x] Container alignment matches
- [x] Grid columns correct
- [ ] Padding slightly off (expected 80px, got 72px) — FIXED

### Typography
- [x] Font sizes match
- [x] Font weights match
- [x] Line heights match

### Colors
- [x] Background colors match
- [x] Text colors match
- [x] Gradients match

### Spacing
- [x] Section padding matches
- [x] Gap between elements matches

### Responsive
- [x] Mobile layout stacks correctly
- [x] Tablet layout adjusts properly
- [x] Desktop layout matches

## Issues Found
1. [Issue description] — [Status: Fixed / Pending]
2. [Issue description] — [Status: Fixed / Pending]

## Console Errors
- None / [List any errors]

## Approval
- [ ] QA passed — Ready for merge
- [ ] QA failed — Needs fixes
```

---

## Tools for QA

### Screenshot Tools
- `chrome-devtools_take_screenshot` — Primary tool
- `playwright_browser_take_screenshot` — Alternative
- `chrome-devtools_resize_page` — Test responsive breakpoints

### Console Tools
- `chrome-devtools_list_console_messages` — Check for errors
- `chrome-devtools_list_network_requests` — Check asset loading

### Browser Tools
- `chrome-devtools_navigate_page` — Navigate to test pages
- `chrome-devtools_evaluate_script` — Test JavaScript

---

## Subagent Usage for QA

For complex QA tasks, use subagents:

1. **Exploration subagent** — To investigate specific visual issues
2. **General subagent** — To run tests and gather console output
3. **prx-nuxt subagent** — To fix identified issues

**Example subagent prompt:**
```
Compare the Figma screenshot and the coded screenshot for the RestaurantCard component.
Identify all visual differences and categorize them by severity (Critical/Major/Minor).
Return a detailed list of discrepancies with specific element names and expected vs actual values.
```
