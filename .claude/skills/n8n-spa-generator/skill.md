---
name: n8n-spa-generator
description: Generate interactive single-page applications for n8n webhook responses. Provides templates and patterns for forms, wizards, dashboards, and real-time interactions using HTMX + Alpine.js + Pico.css (53KB total).
---

# n8n SPA Generator

Generate production-ready, interactive single-page applications for n8n webhooks using HTMX + Alpine.js + Pico.css.

## Core Philosophy

**Server-Driven Architecture**: Each HTMX endpoint maps to one n8n workflow. The server (n8n) handles all logic and returns HTML fragments. HTMX handles DOM updates.

```
User Action → HTMX Request → n8n Workflow → Process Logic → Return HTML → HTMX Updates Page
```

## Tech Stack

- **HTMX 2.x** (14KB) - Hypermedia-driven AJAX via HTML attributes
- **Alpine.js 3.x** (39KB) - Lightweight reactive JavaScript framework
- **Pico.css 2.x** (10KB) - Semantic, classless CSS framework
- **Total**: 53KB (lightweight, no build tools required)

### Version Management

**IMPORTANT**: Always verify the latest versions before starting:

- HTMX: https://unpkg.com/browse/htmx.org@2/ or https://htmx.org
- Alpine.js: https://alpinejs.dev
- Pico.css: https://picocss.com

**Current versions (as of 2025-10-28)**:
- HTMX: `https://unpkg.com/htmx.org@2.0.7`
- Alpine.js: `https://cdn.jsdelivr.net/npm/alpinejs@3/dist/cdn.min.js`
- Pico.css: `https://cdn.jsdelivr.net/npm/@picocss/pico@2/css/pico.min.css`

### Responsibility Division

- **HTMX**: Server communication (HTTP requests, HTML swapping)
- **Alpine.js**: Client-side interactions (state, events, DOM manipulation)
- **Pico.css**: Styling and responsive layout

**Priority Rules**:
1. Alpine.js First for all client-side logic
2. HTMX for server communication only
3. Use Alpine.js methods instead of inline JavaScript
4. Keep HTMX attributes minimal and declarative

## When to Use This Skill

Invoke when:
- Creating HTML pages returned from n8n Webhook nodes
- Building multi-step forms and wizards
- Implementing real-time search and filtering
- Creating data collection and submission flows
- Displaying dynamic content from n8n workflows
- Building status dashboards and monitoring pages
- Implementing infinite scroll or load-more patterns
- Creating interactive surveys and questionnaires

## Quick Start

### 1. Use the Base Template

Copy `assets/base-template.html` as your starting point. It includes:
- All CDN imports (HTMX, Alpine.js, Pico.css)
- Mobile optimization CSS
- Smooth scrolling setup
- HTMX indicator styles
- Basic Alpine.js component structure

### 2. Build Your Page

**For client-side interactions**: Use Alpine.js
```html
<div x-data="{ count: 0 }">
    <button @click="count++">Clicked: <span x-text="count"></span></button>
</div>
```

**For server communication**: Use HTMX
```html
<input hx-get="/webhook/search"
       hx-trigger="keyup changed delay:500ms"
       hx-target="#results">
<div id="results"></div>
```

**Combining both** (Alpine.js listens to HTMX events):
```html
<div x-data="{ loading: false }"
     @htmx:before-request="loading = true"
     @htmx:after-request="loading = false">
    <button hx-get="/api/data">Load Data</button>
    <div x-show="loading">Loading...</div>
</div>
```

### 3. Configure n8n Workflow

**Display HTML (GET)**:
```
[Webhook Node] → [Code/Data Node] → [Respond to Webhook]
```

Respond to Webhook settings:
- Respond With: `Text`
- Response Body: `{{ $json.html }}` or paste full HTML
- Headers: `Content-Type: text/html; charset=utf-8`

**Handle Form/Data (POST)**:
```
[Webhook Node] → [Code: Process Data] → [Respond with HTML]
```

Access form data in Code node:
```javascript
const formData = $input.item.json.body; // { field1: "value1", ... }
const html = `<div>Received: ${formData.field1}</div>`;
return { json: { html } };
```

## Reference Documentation

For detailed syntax, patterns, and best practices, consult these reference files:

### API References
- **`references/htmx-guide.md`** - Complete HTMX documentation (~2,200 words)
  - Core attributes, triggers, swapping strategies
  - Event system, headers, configuration
  - Search with: `grep -i "hx-swap" references/htmx-guide.md`

- **`references/alpine-guide.md`** - Complete Alpine.js documentation (~3,000 words)
  - Reactive directives (x-data, x-model, x-show, x-if, etc.)
  - State management, lifecycle, events
  - Search with: `grep -i "x-data" references/alpine-guide.md`

- **`references/pico-css-guide.md`** - Complete Pico.css documentation (~2,000 words)
  - Semantic HTML components, forms, layout
  - CSS variables, theming, customization
  - Search with: `grep -i "CSS variables" references/pico-css-guide.md`

### Practical Guides
- **`references/best-practices.md`** - Code style, security, mobile optimization
  - Declarative-first coding patterns
  - Security (XSS prevention, CSRF protection)
  - Mobile optimization (iOS zoom prevention, touch targets)
  - Debugging techniques and version management

- **`references/lessons-learned.md`** - Real-world insights from actual projects
  - Event listener placement (use `.window` modifier)
  - Scroll behavior with sticky headers
  - Mobile input zoom prevention
  - Systematic debugging strategies
  - Common pitfalls and solutions

## Common Patterns

### Pattern: Date Selector with Auto-Scroll

```html
<div x-data="{
    selectedDate: new Date().toISOString().split('T')[0],
    scrollToTop() {
        window.scrollTo({ top: 0, behavior: 'smooth' });
    }
}"
@htmx:after-swap.window="scrollToTop()">
    <!-- Date selector -->
    <input type="date"
           name="date"
           x-model="selectedDate"
           hx-get="/webhook/get-content"
           hx-trigger="change, load"
           hx-target="#content-display"
           hx-indicator="#loading"
           :value="selectedDate">

    <!-- Loading indicator -->
    <div id="loading" class="htmx-indicator">
        <progress></progress>
    </div>

    <!-- Content display -->
    <div id="content-display"></div>
</div>
```

**n8n Code Node**:
```javascript
const { date } = $input.item.json.query;
const content = `<article><h2>${date}</h2><p>Content for ${date}</p></article>`;
return { json: { html: content } };
```

### Pattern: Search with Live Results

```html
<input type="search"
       name="q"
       hx-get="/webhook/search"
       hx-trigger="keyup changed delay:500ms"
       hx-target="#results"
       hx-indicator="#search-loading">

<div id="search-loading" class="htmx-indicator">Searching...</div>
<div id="results"></div>
```

### Pattern: Form Submission

```html
<form hx-post="/webhook/submit"
      hx-target="#response"
      hx-indicator="#submit-loading">
    <input type="email" name="email" required>
    <button type="submit">
        Submit
        <span id="submit-loading" class="htmx-indicator">...</span>
    </button>
</form>
<div id="response"></div>
```

## Key Principles

1. **Zero Build Tools** - All dependencies from CDN, no compilation needed
2. **Alpine.js First** - All client-side logic uses Alpine.js, not inline JavaScript
3. **Server-Driven** - n8n handles all business logic and returns HTML
4. **Progressive Enhancement** - Start simple, add features incrementally
5. **Mobile Optimized** - 16px font sizes, touch-friendly spacing
6. **Declarative** - Use HTMX/Alpine.js attributes, avoid custom `<script>` tags

## Troubleshooting

If something doesn't work:

1. **Check versions**: Ensure using latest HTMX 2.x, Alpine.js 3.x, Pico.css 2.x
2. **Check event placement**: Use `@htmx:event.window` on parent container, not triggering element
3. **Check console**: Look for errors, use `console.log()` for debugging
4. **Check Alpine.js**: Ensure script tag has `defer` attribute
5. **Check mobile**: Test on real device, not just browser DevTools

For detailed debugging strategies, see `references/best-practices.md` and `references/lessons-learned.md`.

## Workflow Design

### CRUD Operations
```
GET    /webhook/items           → List all items
GET    /webhook/items/new       → Show create form
POST   /webhook/items           → Create item
GET    /webhook/items/:id       → Show item details
GET    /webhook/items/:id/edit  → Show edit form
PUT    /webhook/items/:id       → Update item
DELETE /webhook/items/:id       → Delete item
```

### Multi-Step Forms
```
GET  /webhook/form         → Show step 1
POST /webhook/form/step2   → Process step 1, show step 2
POST /webhook/form/step3   → Process step 2, show step 3
POST /webhook/form/submit  → Final submission
```

## Delivery Checklist

When providing a solution, ensure:

- [ ] Uses latest library versions (HTMX 2.x, Alpine.js 3.x, Pico.css 2.x)
- [ ] Alpine.js handles all client-side logic
- [ ] HTMX only for server communication
- [ ] Mobile optimized (16px fonts, responsive CSS)
- [ ] Loading indicators for async operations
- [ ] Error handling (client and server)
- [ ] Semantic HTML and accessibility
- [ ] Security (input escaping, CSRF if needed)
- [ ] n8n workflow configuration instructions
- [ ] No custom `<script>` tags (except CDN imports)
- [ ] Comments explaining key logic
