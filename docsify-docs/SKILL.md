# Skill: docsify-docs

Use when the user wants to serve markdown documentation in a browser, mentions "docsify", "documentation site", "markdown viewer", or asks to convert docs into a browsable format with search, navigation, or diagram support.

## Leading Words

- **docsify** — lightweight docs site generator, zero build step
- **diagram-zoom** — click-to-expand modal with pan/zoom controls
- **dark-mode** — persistent theme toggle with localStorage
- **mermaid** — markdown-native diagram rendering
- **sidebar-gen** — automatic navigation from file structure

## Steps

### 1. Inventory markdown files

List all `.md` files in the docs directory. Identify:
- Homepage candidate (README, INDEX, or 00- prefixed file)
- File naming pattern (numbered prefix? semantic names?)
- Presence of `mermaid` code blocks

**Completion:** File list with count; homepage identified.

### 2. Generate `_sidebar.md`

Group files into sections based on filename prefixes or directory structure. Create navigation with:
- Section headers (`**Section Name**`)
- Page links (`[Title](filename.md)`)

For numbered files (01-, 02-), extract titles from first H1 or use descriptive names.

**Completion:** `_sidebar.md` written to docs root.

### 3. Create `index.html`

Write a single HTML file with:
- Docsify 4.x from CDN
- Vue theme (or dark-theme-ready base)
- Search plugin
- Mermaid 10.x with dark-mode theme switching
- Diagram-zoom modal (click, pan, wheel-zoom, +/- buttons, reset)
- Dark-mode toggle (sun/moon) with localStorage persistence
- Copy-code plugin
- Pagination plugin

**Completion:** `index.html` exists in docs root.

### 4. Configure mermaid rendering

Add a Docsify plugin hook that:
- Detects ` ```mermaid ` blocks during markdown rendering
- Wraps them in `<div class="mermaid">`
- Re-runs `mermaid.run()` after each page navigation
- Attaches click handlers to open diagram-zoom modal
- Switches mermaid theme (default/dark) when dark-mode toggles

**Completion:** Mermaid diagrams render and are clickable.

### 5. Start server and verify

Run `docsify serve <docs-dir> --port 3000`. Verify in browser:
- Sidebar navigation loads
- Search works
- Dark mode toggles
- Mermaid diagrams render
- Diagram zoom modal opens and controls work
- Pagination appears between pages

**Completion:** Screenshot or curl confirms 200 OK on localhost:3000.

## Reference

### index.html template

Use this structure, adapting paths and colors to project needs:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Documentation</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <link rel="stylesheet" href="//cdn.jsdelivr.net/npm/docsify@4/lib/themes/vue.css">
  <style>
    /* Mermaid diagram styling */
    .mermaid { text-align: center; margin: 20px 0; cursor: zoom-in; position: relative; }
    .mermaid svg { max-width: 100%; height: auto; }
    
    /* Diagram zoom modal */
    .diagram-modal { display: none; position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.9); z-index: 10000; justify-content: center; align-items: center; overflow: hidden; }
    .diagram-modal.active { display: flex; }
    .diagram-modal-content { position: relative; width: 95%; height: 95%; overflow: hidden; cursor: grab; display: flex; justify-content: center; align-items: center; }
    .diagram-modal-content:active { cursor: grabbing; }
    .diagram-modal-content svg { max-width: none; max-height: none; transition: transform 0.1s ease-out; }
    .diagram-modal-close { position: absolute; top: 20px; right: 20px; background: white; border: none; width: 40px; height: 40px; border-radius: 50%; font-size: 24px; cursor: pointer; z-index: 10001; display: flex; align-items: center; justify-content: center; }
    .diagram-modal-controls { position: absolute; bottom: 20px; left: 50%; transform: translateX(-50%); background: rgba(255,255,255,0.9); border-radius: 8px; padding: 10px 20px; display: flex; gap: 10px; align-items: center; z-index: 10001; }
    .diagram-modal-controls button { background: #42b983; color: white; border: none; padding: 8px 16px; border-radius: 4px; cursor: pointer; font-size: 14px; }
    .diagram-modal-controls .zoom-level { font-size: 14px; min-width: 60px; text-align: center; }
    
    /* Dark mode toggle */
    .dark-mode-toggle { position: fixed; top: 15px; right: 15px; z-index: 100; background: transparent; border: 2px solid var(--theme-color); color: var(--theme-color); width: 40px; height: 40px; border-radius: 50%; cursor: pointer; font-size: 18px; display: flex; align-items: center; justify-content: center; transition: all 0.3s ease; }
    .dark-mode-toggle:hover { background: var(--theme-color); color: white; }
    
    /* Dark mode styles (add full dark theme overrides here) */
    body.dark-mode { background: #1a1a2e; color: #e0e0e0; }
    body.dark-mode .sidebar { background: #16213e; border-right: 1px solid #0f3460; }
    /* ... (extend as needed) */
  </style>
</head>
<body>
  <div id="app"></div>
  <button class="dark-mode-toggle" id="darkModeToggle" title="Toggle Dark Mode">🌙</button>
  <div id="diagramModal" class="diagram-modal">
    <button class="diagram-modal-close" onclick="closeDiagramModal()">&times;</button>
    <div class="diagram-modal-content" id="diagramModalContent"></div>
    <div class="diagram-modal-controls">
      <button onclick="zoomDiagram(0.8)">-</button>
      <span class="zoom-level" id="zoomLevel">100%</span>
      <button onclick="zoomDiagram(1.25)">+</button>
      <button onclick="resetZoom()">Reset</button>
    </div>
  </div>
  <script>
    // Dark mode logic (check localStorage + system preference)
    let isDarkMode = localStorage.getItem('docsify-dark-mode') === 'true' || 
      (localStorage.getItem('docsify-dark-mode') === null && window.matchMedia('(prefers-color-scheme: dark)').matches);
    
    function applyDarkMode() {
      document.body.classList.toggle('dark-mode', isDarkMode);
      document.getElementById('darkModeToggle').textContent = isDarkMode ? '☀️' : '🌙';
      if (typeof mermaid !== 'undefined') mermaid.initialize({ theme: isDarkMode ? 'dark' : 'default' });
    }
    applyDarkMode();
    
    window.$docsify = {
      name: 'Documentation',
      loadSidebar: true,
      subMaxLevel: 3,
      auto2top: true,
      search: { paths: 'auto', placeholder: 'Search...', depth: 6 }
    };
  </script>
  <script src="//cdn.jsdelivr.net/npm/docsify@4"></script>
  <script src="//cdn.jsdelivr.net/npm/docsify/lib/plugins/search.min.js"></script>
  <script src="//cdn.jsdelivr.net/npm/docsify-pagination/dist/docsify-pagination.min.js"></script>
  <script src="//cdn.jsdelivr.net/npm/docsify-copy-code@2"></script>
  <script src="//cdn.jsdelivr.net/npm/mermaid@10/dist/mermaid.min.js"></script>
  <script>
    // Mermaid + diagram zoom integration
    window.$docsify.markdown = {
      renderer: {
        code: function(code, lang) {
          if (lang === 'mermaid') return '<div class="mermaid">' + code + '</div>';
          return this.origin.code.apply(this, arguments);
        }
      }
    };
    
    window.$docsify.plugins = [].concat(window.$docsify.plugins || [], function(hook) {
      hook.doneEach(function() {
        if (typeof mermaid !== 'undefined') {
          mermaid.run({ querySelector: '.mermaid' }).then(() => {
            document.querySelectorAll('.mermaid').forEach(el => {
              el.addEventListener('click', () => openDiagramModal(el));
            });
          });
        }
      });
    });
    
    // Zoom modal functions (openDiagramModal, zoomDiagram, resetZoom, pan logic)
    // ... (implement full zoom/pan/controls as in working example)
  </script>
</body>
</html>
```

### Sidebar generation rules

- **Numbered files** (01-, 02-): Group by decade into sections (01-09 Architecture, 10-19 Operational, etc.)
- **Named files** (auth.md, api.md): Group by semantic category
- **README/INDEX**: Always first, under "Getting Started"
- **Missing files** in sequence: Omit from sidebar, don't create placeholder

### Common issues

| Symptom | Cause | Fix |
|---------|-------|-----|
| 404 on homepage | No README.md | Set `homepage: 'actual-file.md'` in $docsify config |
| Sidebar shows TOC not nav | `_sidebar.md` not loading | Ensure `loadSidebar: true`; check file is at docs root |
| Mermaid not rendering | Called `mermaid.initialize()` before script loaded | Guard with `if (typeof mermaid !== 'undefined')` |
| Dark mode flashes light | Class applied after render | Set class in `<script>` immediately after `<body>` opens |

## Dependencies

- `docsify-cli` (global npm package) for `docsify serve`
- Or any static file server (`npx serve`, `python -m http.server`)
