# Lessons Learned from Real Projects

This document captures practical insights from actual n8n webhook HTML development projects.

## 1. Event Listener Placement Matters

### Problem
Events not firing when listening on the triggering element.

### Solution
Place listener on parent container with `.window` modifier.

```html
<!-- ❌ Doesn't always work -->
<input hx-get="/api" @htmx:after-swap="doSomething()">

<!-- ✅ Reliable -->
<div @htmx:after-swap.window="doSomething()">
    <input hx-get="/api">
</div>
```

### Why This Matters
HTMX events bubble up from the triggering element. Listening at the parent level with `.window` ensures you catch the event regardless of where it originates.

## 2. Scroll Behavior with Sticky Headers

### Problem
`scrollIntoView()` causes layout jumps with sticky headers.

### Solution
Use `window.scrollTo({top: 0})` instead.

```javascript
// ❌ Can cause issues with sticky headers
scrollToTop() {
    document.getElementById('top').scrollIntoView({behavior: 'smooth'});
}

// ✅ More reliable
scrollToTop() {
    window.scrollTo({top: 0, behavior: 'smooth'});
}
```

### Why This Matters
When you have a sticky header (position: sticky), `scrollIntoView()` tries to scroll the element into view but doesn't account for the sticky header's offset. This can cause the content to appear behind the header or create unexpected jumps.

`window.scrollTo({top: 0})` simply scrolls to the page top, which is always predictable and works perfectly with sticky headers.

### Real-World Example

```html
<!-- Sticky header -->
<div class="date-selector" style="position: sticky; top: 0;">
    <input type="date" hx-get="/content" hx-trigger="change">
</div>

<!-- Content area -->
<div x-data="{
    scrollToTop() {
        // This works perfectly with the sticky header above
        window.scrollTo({ top: 0, behavior: 'smooth' });
    }
}"
@htmx:after-swap.window="scrollToTop()">
    <div id="content"></div>
</div>
```

## 3. Mobile Input Zoom Prevention

### Problem
iOS Safari zooms in when focusing on inputs with font-size < 16px.

### Solution
Set all form inputs to `font-size: 16px`.

```css
/* Critical for iOS */
input[type="date"],
input[type="text"],
input[type="email"],
textarea {
    font-size: 16px;
}
```

### Why This Matters
This is a specific iOS Safari behavior designed to help users see small text. However, it creates a jarring experience where the page suddenly zooms in when tapping an input field.

Setting `font-size: 16px` (or larger) tells iOS that the text is already readable, preventing the automatic zoom.

### Additional Mobile Optimization

```css
:root {
    --pico-font-size: 16px; /* Set globally for Pico.css */
}

/* Also prevent zoom on viewport meta tag */
```

```html
<meta name="viewport" content="width=device-width, initial-scale=1.0">
```

## 4. Version Management

### Problem
Using outdated library versions leads to missing features and bugs.

### Solution
Check for latest versions before each project.

**Resources:**
- HTMX: https://unpkg.com/browse/htmx.org@2/
- Alpine.js: https://alpinejs.dev
- Pico.css: https://picocss.com

### Why This Matters
Web frameworks evolve quickly. For example:
- HTMX 2.x has different event syntax than 1.x
- Alpine.js 3.x has significant improvements over 2.x
- Pico.css 2.x has better mobile support than 1.x

### Recommended Practice

Create a checklist at the start of each project:

```markdown
- [ ] Check HTMX latest version
- [ ] Check Alpine.js latest version
- [ ] Check Pico.css latest version
- [ ] Update CDN links in template
```

## 5. Debugging Strategy

### Problem
Unclear where the issue is when something doesn't work.

### Solution
Use systematic debugging with console logs.

**Pattern**: Add logs → Test → Verify → Clean up

```javascript
// Debug version
scrollToTop() {
    console.log('Called');  // Add this first
    window.scrollTo({top: 0, behavior: 'smooth'});
    console.log('Done');    // Verify execution
}

// Production version (remove logs)
scrollToTop() {
    window.scrollTo({top: 0, behavior: 'smooth'});
}
```

### Debugging Checklist

When something doesn't work:

1. **Is the library loaded?**
   ```javascript
   console.log(htmx.version);  // Should show version
   console.log(typeof Alpine);  // Should show "object"
   ```

2. **Is the event firing?**
   ```html
   @htmx:after-swap.window="console.log('Event fired!'); myMethod()"
   ```

3. **Is the method called?**
   ```javascript
   myMethod() {
       console.log('Method called!');
       // ... actual logic
   }
   ```

4. **Is the selector correct?**
   ```javascript
   const element = document.getElementById('my-id');
   console.log('Element found:', element);  // Should not be null
   ```

## 6. Alpine.js First, HTMX Second

### Principle
Use Alpine.js for ALL client-side logic, HTMX only for server communication.

### Why This Matters
Clear separation of concerns makes code more maintainable:

- **Alpine.js** = Client-side state, methods, event handling
- **HTMX** = Server HTTP requests, HTML swapping

### Example: Correct Responsibility Division

```html
<div x-data="{
    // State
    items: [],
    loading: false,
    error: null,

    // Methods
    refresh() {
        htmx.trigger('#list', 'refresh');
    },

    handleError(message) {
        this.error = message;
        setTimeout(() => this.error = null, 3000);
    }
}"
@htmx:before-request="loading = true; error = null"
@htmx:after-request="loading = false"
@htmx:response-error="handleError('Failed to load data')">

    <!-- HTMX only handles HTTP -->
    <div id="list" hx-get="/api/items" hx-trigger="refresh"></div>

    <!-- Alpine.js handles UI -->
    <div x-show="loading">Loading...</div>
    <div x-show="error" x-text="error" role="alert"></div>

    <button @click="refresh()">Refresh</button>
</div>
```

### Benefits

1. **Testability**: Alpine.js methods can be tested independently
2. **Reusability**: Methods can be called from multiple places
3. **Readability**: Clear where each piece of logic lives
4. **Maintainability**: Easy to update client-side logic without touching HTMX attributes

## 7. Progressive Enhancement Patterns

### Start Simple, Add Complexity

When building a feature, start with the simplest possible implementation:

**Phase 1: Basic functionality**
```html
<input hx-get="/search" hx-trigger="change" hx-target="#results">
<div id="results"></div>
```

**Phase 2: Add loading state**
```html
<input hx-get="/search"
       hx-trigger="change"
       hx-target="#results"
       hx-indicator="#spinner">
<div id="spinner" class="htmx-indicator">Loading...</div>
<div id="results"></div>
```

**Phase 3: Add debouncing**
```html
<input hx-get="/search"
       hx-trigger="keyup changed delay:500ms"
       hx-target="#results"
       hx-indicator="#spinner">
```

**Phase 4: Add error handling**
```html
<div x-data="{ hasError: false }"
     @htmx:response-error="hasError = true">
    <input hx-get="/search"
           hx-trigger="keyup changed delay:500ms"
           hx-target="#results">
    <div x-show="hasError" role="alert">Search failed</div>
    <div id="results"></div>
</div>
```

### Why This Matters

Building features incrementally:
- Makes debugging easier (you know what changed)
- Allows for user feedback at each stage
- Prevents over-engineering
- Creates natural testing checkpoints

## 8. Common Pitfalls and How to Avoid Them

### Pitfall 1: Forgetting `defer` on Alpine.js

```html
<!-- ❌ Alpine.js won't work -->
<script src="https://cdn.jsdelivr.net/npm/alpinejs@3/dist/cdn.min.js"></script>

<!-- ✅ Correct -->
<script defer src="https://cdn.jsdelivr.net/npm/alpinejs@3/dist/cdn.min.js"></script>
```

**Why**: Alpine.js needs the DOM to be ready before initializing.

### Pitfall 2: Not Using `hx-include` with Forms

```html
<!-- ❌ Only sends the button, not the date -->
<input type="date" name="date" value="2025-10-28">
<button hx-get="/search">Search</button>

<!-- ✅ Includes the date input -->
<input type="date" name="date" id="date-input" value="2025-10-28">
<button hx-get="/search" hx-include="#date-input">Search</button>
```

### Pitfall 3: Forgetting `.window` Modifier

```html
<!-- ❌ May not work reliably -->
<div x-data="{...}" @htmx:after-swap="doSomething()">
    <input hx-get="/api">
</div>

<!-- ✅ Reliable -->
<div x-data="{...}" @htmx:after-swap.window="doSomething()">
    <input hx-get="/api">
</div>
```

### Pitfall 4: Not Setting `scroll-behavior`

```css
/* ❌ Scroll will be instant, not smooth */
/* (no CSS) */

/* ✅ Enable smooth scrolling */
html {
    scroll-behavior: smooth;
}
```

## Summary

The key lessons from real projects:

1. ✅ **Event listeners**: Use `.window` modifier on parent container
2. ✅ **Scrolling**: Use `window.scrollTo()` not `scrollIntoView()` with sticky headers
3. ✅ **Mobile**: Set `font-size: 16px` on all inputs for iOS
4. ✅ **Versions**: Check for latest before each project
5. ✅ **Debugging**: Systematic approach with console logs
6. ✅ **Architecture**: Alpine.js for client, HTMX for server
7. ✅ **Development**: Build incrementally, test at each stage
8. ✅ **Common fixes**: Use `defer`, `hx-include`, `.window`, `scroll-behavior`
