# Best Practices for n8n Webhook HTML Development

## Code Style - Declarative First

### Avoid Custom Script Tags

**❌ Bad - Don't use custom JavaScript**
```html
<input type="date" id="date-picker">
<script>
document.addEventListener('DOMContentLoaded', function() {
    const dateInput = document.getElementById('date-picker');
    htmx.trigger(dateInput, 'change');
});

document.body.addEventListener('htmx:afterSwap', function() {
    window.scrollTo(0, 0);
});
</script>
```

**✅ Good - Use Alpine.js for client-side logic**
```html
<div x-data="{
    selectedDate: new Date().toISOString().split('T')[0],
    scrollToTop() {
        window.scrollTo({
            top: 0,
            behavior: 'smooth'
        });
    }
}">
    <input type="date"
           x-model="selectedDate"
           hx-get="/webhook/get-content"
           hx-trigger="change, load"
           hx-target="#content-display"
           @htmx:after-swap="scrollToTop()">

    <div id="content-display"></div>
</div>
```

### Key Rules

1. **Alpine.js First** for all client-side interactions
   - State management: `x-data`
   - Methods: Define in `x-data` object
   - Events: `@click`, `@htmx:after-swap`, etc.
   - DOM manipulation: Alpine.js methods

2. **HTMX for server communication only**
   - `hx-get`, `hx-post` for requests
   - `hx-target`, `hx-swap` for HTML updates
   - Keep HTMX attributes minimal

3. **Avoid inline JavaScript**
   - Use Alpine.js methods instead of inline code
   - Use `hx-trigger="load"` instead of `DOMContentLoaded`
   - Use Alpine.js directives instead of `addEventListener`

4. **Only use `<script>` tags for:**
   - Library CDN imports (HTMX, Alpine.js, Pico.css)
   - Global HTMX configuration (e.g., `htmx.config.timeout`)
   - Truly complex logic that cannot be expressed declaratively

### Responsibility Division Examples

```html
<!-- ✅ Alpine.js: State management -->
<div x-data="{ count: 0, isOpen: false }">
    <button @click="count++">Clicked: <span x-text="count"></span></button>
</div>

<!-- ✅ Alpine.js: Event handling -->
<div x-data="{ message: '' }">
    <button @click="message = 'Hello!'">Say Hello</button>
    <p x-text="message"></p>
</div>

<!-- ✅ Alpine.js: Listen to HTMX events -->
<div x-data="{ loading: false }">
    <button hx-get="/api/data"
            @htmx:before-request="loading = true"
            @htmx:after-request="loading = false">
        Load Data
    </button>
    <div x-show="loading">Loading...</div>
</div>

<!-- ✅ HTMX: Server communication only -->
<input hx-get="/search"
       hx-trigger="keyup changed delay:500ms, load"
       hx-target="#results">
```

## Security

### Escape User Content

```javascript
function escapeHtml(unsafe) {
    return unsafe
        .replace(/&/g, "&amp;")
        .replace(/</g, "&lt;")
        .replace(/>/g, "&gt;")
        .replace(/"/g, "&quot;")
        .replace(/'/g, "&#039;");
}

const html = `<p>${escapeHtml(userName)}</p>`;
```

### CSRF Protection

```html
<body hx-headers='{"X-CSRF-Token": "token"}'>
```

### Validate in n8n

```javascript
const { email } = $input.item.json.body;
if (!email || !email.match(/^[^\s@]+@[^\s@]+\.[^\s@]+$/)) {
    return { html: '<div>Invalid input</div>' };
}
```

## Performance

### Prevent Duplicate Requests

```html
<input hx-get="/search"
       hx-trigger="keyup changed delay:500ms"
       hx-sync="this:abort">
```

### Return Fragments, Not Full Pages

```javascript
// ✅ Good
return { html: '<div>Updated content</div>' };

// ❌ Bad
return { html: '<html><body>...</body></html>' };
```

## User Experience

### Loading Indicators

```html
<button hx-post="/endpoint" hx-indicator="#spinner">Submit</button>
<span id="spinner" class="htmx-indicator">Processing...</span>
```

### Disable During Requests

```html
<button hx-post="/submit" hx-disabled-elt="this">Submit</button>
```

### Auto-Scroll After Content Update

```html
<div x-data="{
    scrollToTop() {
        window.scrollTo({ top: 0, behavior: 'smooth' });
    }
}"
@htmx:after-swap.window="scrollToTop()">
    <!-- Content that changes -->
</div>
```

**Why `window.scrollTo()` over `scrollIntoView()`?**
- More reliable with sticky/fixed headers
- Simpler API, less edge cases
- Better cross-browser consistency

## Error Handling

### Client-Side

```html
<div hx-get="/data"
     @htmx:response-error="$el.textContent = 'Error loading data'">
</div>
```

### Server-Side (n8n)

```javascript
try {
    const data = await fetchData();
    return { html: renderData(data) };
} catch (error) {
    return {
        html: '<div role="alert">⚠️ Something went wrong</div>'
    };
}
```

## Accessibility

### Use Semantic HTML

```html
<!-- ✅ Good -->
<button hx-post="/action">Click me</button>

<!-- ❌ Bad -->
<div onclick="...">Click me</div>
```

### ARIA Labels

```html
<button hx-delete="/item/1" aria-label="Delete item">🗑️</button>
```

### Focus Management

```html
<div hx-get="/modal"
     @htmx:after-swap="$el.querySelector('input')?.focus()">
</div>
```

## Mobile Optimization

### Prevent iOS Auto-Zoom on Input Focus

**Critical for iOS Safari**

```css
:root {
    --pico-font-size: 16px; /* iOS won't zoom if font-size >= 16px */
}

input[type="date"],
input[type="text"],
input[type="email"],
textarea {
    font-size: 16px; /* Critical for iOS */
}
```

### Sticky Headers for Mobile

```css
.header {
    position: sticky;
    top: 0;
    background: var(--pico-background-color);
    z-index: 10;
}
```

### Touch-Friendly Spacing

```css
/* Pico.css handles this automatically, but if you need custom buttons: */
button, .touchable {
    min-height: 44px; /* Apple's minimum touch target */
    padding: 12px 20px;
}
```

### Responsive Containers

```css
@media (max-width: 768px) {
    .container {
        padding: 0; /* Full width on mobile */
    }
}
```

## Debugging Techniques

### Step 1: Add Console Logs to Verify Event Flow

```html
<div x-data="{
    scrollToTop() {
        console.log('scrollToTop called');
        window.scrollTo({ top: 0, behavior: 'smooth' });
        console.log('scroll executed');
    }
}"
@htmx:after-swap.window="console.log('HTMX after-swap detected'); scrollToTop()">
```

### Step 2: Check Browser Console

- If you see "HTMX after-swap detected" → event is firing
- If you see "scrollToTop called" → method is executing
- If neither appears → check event listener placement

### Step 3: Verify HTMX is Working

```javascript
// In browser console:
htmx.version  // Should show "2.0.7" or similar
```

### Step 4: Clean Up After Debugging

```html
<!-- Remove all console.log statements once working -->
<div x-data="{
    scrollToTop() {
        window.scrollTo({ top: 0, behavior: 'smooth' });
    }
}"
@htmx:after-swap.window="scrollToTop()">
```

### Common Debugging Tips

- Alpine.js not working? Check `defer` attribute on script tag
- HTMX events not firing? Use `.window` modifier for global events
- Scroll not working? Check if `scroll-behavior: smooth` is set on `html`
- Mobile issues? Test with real device, not just browser DevTools

## Version Management

### Always Check for Latest Versions

Before starting any project, verify the latest versions:

- **HTMX**: https://unpkg.com/browse/htmx.org@2/ or https://htmx.org
- **Alpine.js**: https://alpinejs.dev
- **Pico.css**: https://picocss.com

### Current Versions (as of 2025-10-28)

- HTMX: `https://unpkg.com/htmx.org@2.0.7`
- Alpine.js: `https://cdn.jsdelivr.net/npm/alpinejs@3/dist/cdn.min.js`
- Pico.css: `https://cdn.jsdelivr.net/npm/@picocss/pico@2/css/pico.min.css`
