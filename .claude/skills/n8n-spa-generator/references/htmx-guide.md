# HTMX Complete Guide

## Core Concepts

### Philosophy: Hypermedia-Driven Applications

HTMX extends HTML with attributes that enable modern browser features directly in markup, without writing JavaScript. The core philosophy is **Hypermedia as the Engine of Application State** (HATEOAS).

**Traditional SPA approach**:
```
Server → JSON → JavaScript → DOM
```

**HTMX approach**:
```
Server → HTML → DOM (HTMX handles the plumbing)
```

### Key Benefits for n8n

1. **Server-side logic**: All business logic stays in n8n workflows
2. **No build tools**: Drop CDN links into HTML and start
3. **Progressive enhancement**: Works without JavaScript (graceful degradation)
4. **Simplicity**: HTML attributes instead of JavaScript frameworks
5. **Small footprint**: 14KB total

### Core Attributes Reference

#### HTTP Methods

```html
hx-get="/path"      <!-- GET request -->
hx-post="/path"     <!-- POST request -->
hx-put="/path"      <!-- PUT request -->
hx-patch="/path"    <!-- PATCH request -->
hx-delete="/path"   <!-- DELETE request -->
```

**When to use each**:
- `hx-get`: Fetching data, search, filters, pagination
- `hx-post`: Creating resources, form submissions
- `hx-put`: Full updates of resources
- `hx-patch`: Partial updates
- `hx-delete`: Removing resources

#### Targeting: Where to Put the Response

```html
<!-- CSS Selectors -->
hx-target="#result"              <!-- Element with id="result" -->
hx-target=".results"             <!-- First element with class="results" -->
hx-target="closest tr"           <!-- Nearest ancestor <tr> -->
hx-target="next .item"           <!-- Next sibling with class="item" -->
hx-target="previous .item"       <!-- Previous sibling -->
hx-target="find .child"          <!-- First descendant -->

<!-- Special Keywords -->
hx-target="this"                 <!-- Replace triggering element -->
hx-target="body"                 <!-- Replace entire body -->
```

**Best practices**:
- Use IDs for unique targets: `#result`
- Use `closest` for table rows: `closest tr`
- Use `this` for self-updating elements
- Avoid deeply nested selectors

#### Swapping: How to Insert the Response

```html
<!-- Content Replacement -->
hx-swap="innerHTML"      <!-- Replace inner content (default) -->
hx-swap="outerHTML"      <!-- Replace entire element including tag -->
hx-swap="textContent"    <!-- Replace text only (no HTML parsing) -->

<!-- Insertion -->
hx-swap="beforebegin"    <!-- Insert before element -->
hx-swap="afterbegin"     <!-- Insert as first child -->
hx-swap="beforeend"      <!-- Insert as last child -->
hx-swap="afterend"       <!-- Insert after element -->

<!-- Special -->
hx-swap="delete"         <!-- Remove target element -->
hx-swap="none"           <!-- Don't swap (useful with OOB swaps) -->
```

**Swap Modifiers**:
```html
hx-swap="innerHTML swap:200ms"           <!-- Delay swap 200ms -->
hx-swap="innerHTML settle:500ms"         <!-- Delay settle 500ms -->
hx-swap="innerHTML show:top"             <!-- Scroll to top on swap -->
hx-swap="innerHTML show:bottom"          <!-- Scroll to bottom -->
hx-swap="innerHTML show:#element:top"    <!-- Scroll element to top of viewport -->
hx-swap="innerHTML focus-scroll:true"    <!-- Maintain scroll on focused element -->
```

**Use cases**:
- `innerHTML`: Most common, update content area
- `outerHTML`: Replace entire component (wizards, cards)
- `beforeend`: Infinite scroll, load more
- `afterend`: Insert new rows in tables
- `delete`: Remove items from lists

#### Triggers: When to Send the Request

```html
<!-- Standard Events -->
hx-trigger="click"       <!-- Button/link clicks (default for non-inputs) -->
hx-trigger="change"      <!-- Input value changes (default for inputs) -->
hx-trigger="submit"      <!-- Form submission -->
hx-trigger="keyup"       <!-- Key released -->
hx-trigger="keydown"     <!-- Key pressed -->
hx-trigger="mouseenter"  <!-- Mouse enters element -->
hx-trigger="mouseleave"  <!-- Mouse leaves element -->

<!-- Special Triggers -->
hx-trigger="load"        <!-- When element loads (DOMContentLoaded) -->
hx-trigger="revealed"    <!-- When element scrolls into viewport (Intersection Observer) -->
hx-trigger="intersect"   <!-- Same as revealed, with more options -->
hx-trigger="every 5s"    <!-- Poll every 5 seconds -->
```

**Trigger Modifiers**:

```html
<!-- Timing -->
hx-trigger="keyup delay:500ms"              <!-- Debounce: wait 500ms after last event -->
hx-trigger="click throttle:1s"              <!-- Throttle: max once per second -->
hx-trigger="click once"                     <!-- Fire only once -->

<!-- Conditionals -->
hx-trigger="keyup changed"                  <!-- Only if value changed -->
hx-trigger="click[ctrlKey]"                 <!-- Only if Ctrl key pressed -->
hx-trigger="mousedown[button==1]"           <!-- Only middle mouse button -->
hx-trigger="keyup[key=='Enter']"            <!-- Only Enter key -->

<!-- Event Source -->
hx-trigger="click from:body"                <!-- Listen on body instead -->
hx-trigger="customEvent from:window"        <!-- Custom event from window -->

<!-- Multiple Triggers -->
hx-trigger="click, keyup delay:1s"          <!-- Either click or keyup -->
hx-trigger="mouseenter once, mouseleave"    <!-- Mix modifiers -->

<!-- Consumption -->
hx-trigger="click consume"                  <!-- Prevent event bubbling -->
```

**Common Patterns**:

1. **Search with debouncing**:
```html
<input hx-get="/search"
       hx-trigger="keyup changed delay:500ms"
       hx-target="#results">
```

2. **Infinite scroll**:
```html
<div hx-get="/page/2"
     hx-trigger="revealed"
     hx-swap="afterend">
```

3. **Polling**:
```html
<div hx-get="/status"
     hx-trigger="every 5s"
     hx-swap="innerHTML">
```

4. **Ctrl+Click for details**:
```html
<button hx-get="/details"
        hx-trigger="click[ctrlKey]"
        hx-target="#modal">
```

#### Request Synchronization

```html
<!-- Abort Strategy -->
hx-sync="this:abort"           <!-- Abort in-flight request from this element -->
hx-sync="closest form:abort"   <!-- Abort requests from parent form -->

<!-- Queue Strategy -->
hx-sync="this:queue"           <!-- Queue requests (default) -->
hx-sync="this:queue first"     <!-- Queue, process first only -->
hx-sync="this:queue last"      <!-- Queue, process last only -->
hx-sync="this:queue all"       <!-- Queue, process all -->

<!-- Drop Strategy -->
hx-sync="this:drop"            <!-- Drop new requests while one is in flight -->

<!-- Replace Strategy -->
hx-sync="this:replace"         <!-- Replace queued request with new one -->
```

**When to use**:
- `abort`: Search boxes (abort old searches)
- `drop`: Prevent double-clicks on submit buttons
- `queue last`: Only care about most recent state
- `queue all`: All requests matter (analytics)

#### Loading Indicators

```html
<!-- Show spinner -->
<button hx-get="/slow" hx-indicator="#spinner">Load</button>
<div id="spinner" class="htmx-indicator">Loading...</div>

<!-- CSS (required) -->
<style>
.htmx-indicator {
    display: none;
}
.htmx-request .htmx-indicator {
    display: inline-block;
}
.htmx-request.htmx-indicator {
    display: inline-block;
}
</style>
```

**Advanced indicators**:
```html
<!-- Multiple indicators -->
<button hx-get="/data" hx-indicator="#spinner1, #spinner2">

<!-- Indicator on different element -->
<button hx-get="/data" hx-indicator="closest tr .spinner">
```

#### Disabling Elements During Requests

```html
<!-- Disable self -->
<button hx-post="/submit" hx-disabled-elt="this">Submit</button>

<!-- Disable other elements -->
<button hx-post="/submit" hx-disabled-elt="#form-inputs">Submit</button>

<!-- Disable multiple -->
<button hx-post="/submit" hx-disabled-elt="this, #other-button">
```

#### Including Additional Data

```html
<!-- Include other form fields -->
<button hx-post="/submit" hx-include="[name='csrf']">

<!-- Include closest form -->
<button hx-post="/submit" hx-include="closest form">

<!-- Add JSON values -->
<button hx-post="/submit" hx-vals='{"extra": "value"}'>

<!-- Add computed values -->
<button hx-post="/submit"
        hx-vars="timestamp:Date.now()">
```

#### Custom Headers

```html
<!-- Static headers -->
<div hx-headers='{"X-CSRF-Token": "abc123"}'>
    <button hx-post="/submit">Submit</button>
</div>

<!-- Apply to all requests in subtree -->
<body hx-headers='{"X-Custom": "value"}'>
```

#### Confirming Actions

```html
<!-- Simple confirm -->
<button hx-delete="/user/1"
        hx-confirm="Are you sure?">
    Delete
</button>

<!-- Custom confirm (via event) -->
<button hx-delete="/user/1"
        hx-on:htmx:confirm="if(!customConfirm()) event.preventDefault()">
    Delete
</button>
```

#### History Support

```html
<!-- Push to browser history -->
<a hx-get="/page2" hx-push-url="true">Page 2</a>

<!-- Custom URL in history -->
<a hx-get="/page2" hx-push-url="/pages/two">Page 2</a>

<!-- Replace current history entry -->
<a hx-get="/page2" hx-replace-url="true">Page 2</a>
```

#### Boosting Links and Forms

Convert all links and forms to AJAX:

```html
<!-- Boost entire app -->
<body hx-boost="true">
    <a href="/page2">Page 2</a>  <!-- AJAX request -->
    <form action="/submit">      <!-- AJAX submit -->
        <button>Submit</button>
    </form>
</body>
```

**Best practice**: Use with history support for SPA-like experience.

### Request/Response Headers

#### Request Headers (Sent by HTMX)

```
HX-Request: true                 <!-- Identifies HTMX request -->
HX-Trigger: element-id           <!-- ID of triggering element -->
HX-Trigger-Name: input-name      <!-- Name attribute of trigger -->
HX-Target: target-id             <!-- ID of target element -->
HX-Current-URL: /current/path    <!-- Current page URL -->
HX-Boosted: true                 <!-- Request via hx-boost -->
HX-Prompt: user-input            <!-- Response to hx-prompt -->
```

**n8n usage**:
```javascript
// Check if HTMX request
const isHtmx = $input.item.json.headers['hx-request'] === 'true';

if (isHtmx) {
    // Return HTML fragment
    return { html: '<div>Updated</div>' };
} else {
    // Return full page
    return { html: '<!DOCTYPE html>...' };
}
```

#### Response Headers (From n8n)

```
HX-Location: /new/url                    <!-- Client-side redirect -->
HX-Push-Url: /new/url                    <!-- Push to history -->
HX-Redirect: /new/url                    <!-- Full page redirect -->
HX-Refresh: true                         <!-- Refresh entire page -->

HX-Trigger: eventName                    <!-- Trigger client event -->
HX-Trigger: {"event1": null, "event2": null}  <!-- Multiple events -->
HX-Trigger: {"showMessage": {"text": "Hello"}}  <!-- Event with detail -->

HX-Retarget: #new-target                 <!-- Change target selector -->
HX-Reswap: beforeend                     <!-- Change swap method -->
HX-Reselect: #specific-part              <!-- Select part of response -->
```

**n8n example**:
```javascript
// Trigger client-side event after update
return {
    html: '<div>Updated</div>',
    headers: {
        'HX-Trigger': JSON.stringify({
            'showNotification': { message: 'Success!' }
        })
    }
};
```

### Out-of-Band (OOB) Swaps

Update multiple parts of the page from a single response:

```html
<!-- Main target -->
<button hx-post="/update" hx-target="#main">Update</button>

<div id="main"></div>
<div id="notifications"></div>
<div id="counter"></div>
```

**n8n response**:
```javascript
return {
    html: `
        <!-- This goes to hx-target="#main" -->
        <div id="main">
            Main content updated
        </div>

        <!-- These go to their matching IDs -->
        <div id="notifications" hx-swap-oob="true">
            <div role="alert">Update successful!</div>
        </div>

        <div id="counter" hx-swap-oob="innerHTML">
            42 items
        </div>

        <!-- Custom swap strategy -->
        <div id="list" hx-swap-oob="beforeend">
            <li>New item</li>
        </div>
    `
};
```

**OOB Swap Strategies**:
```html
hx-swap-oob="true"           <!-- Default: innerHTML, match by ID -->
hx-swap-oob="outerHTML"      <!-- Replace entire element -->
hx-swap-oob="beforeend"      <!-- Append -->
hx-swap-oob="afterbegin"     <!-- Prepend -->
hx-swap-oob="beforebegin"    <!-- Insert before -->
hx-swap-oob="afterend"       <!-- Insert after -->
hx-swap-oob="delete"         <!-- Remove element -->
hx-swap-oob="innerHTML:#id"  <!-- Explicit target -->
```

### HTMX Events

Listen to HTMX lifecycle events:

```html
<!-- Inline event handlers -->
<div hx-get="/data"
     hx-on:htmx:before-request="console.log('Starting...')"
     hx-on:htmx:after-swap="console.log('Done!')">
</div>

<!-- JavaScript event listeners -->
<script>
document.body.addEventListener('htmx:configRequest', function(evt) {
    // Modify request before sending
    evt.detail.parameters.extra = 'value';
    evt.detail.headers['X-Custom'] = 'header';
});

document.body.addEventListener('htmx:beforeSwap', function(evt) {
    // Modify swap behavior
    if (evt.detail.xhr.status === 404) {
        evt.detail.shouldSwap = true;
        evt.detail.target = document.getElementById('error');
    }
});

document.body.addEventListener('htmx:afterSwap', function(evt) {
    // Run after swap
    console.log('Swapped into:', evt.detail.target);
});
</script>
```

**Complete Event Reference**:

```javascript
// Request Lifecycle
htmx:configRequest       // Before request sent (modify request)
htmx:beforeRequest       // Just before request sent
htmx:beforeSend          // Right before XHR sent
htmx:xhr:loadstart       // XHR loadstart
htmx:xhr:loadend         // XHR loadend
htmx:xhr:progress        // XHR progress
htmx:xhr:abort           // XHR aborted

// Response Handling
htmx:beforeSwap          // Before swap (modify swap)
htmx:afterSwap           // After swap, before settle
htmx:afterSettle         // After CSS transitions complete
htmx:load                // After new content loaded

// Errors
htmx:responseError       // 4xx or 5xx response
htmx:sendError           // Network error
htmx:timeout             // Request timeout
htmx:validationValidate  // Before validation
htmx:validationFailed    // Validation failed

// History
htmx:pushedIntoHistory   // URL pushed to history
htmx:replacedInHistory   // URL replaced in history
htmx:historyRestore      // Restoring from history

// Other
htmx:trigger             // Custom trigger fired
htmx:confirm             // Confirmation requested
```

### Configuration

Global HTMX configuration:

```html
<script>
// Set before loading HTMX
htmx.config.defaultSwapStyle = 'outerHTML';
htmx.config.defaultSwapDelay = 0;
htmx.config.defaultSettleDelay = 20;
htmx.config.timeout = 120000;  // 2 minutes
htmx.config.historyCacheSize = 10;
htmx.config.refreshOnHistoryMiss = false;
htmx.config.includeIndicatorStyles = true;
htmx.config.indicatorClass = 'htmx-indicator';
htmx.config.requestClass = 'htmx-request';
htmx.config.addedClass = 'htmx-added';
htmx.config.settlingClass = 'htmx-settling';
htmx.config.swappingClass = 'htmx-swapping';
htmx.config.allowEval = true;
htmx.config.allowScriptTags = true;
htmx.config.inlineScriptNonce = '';
htmx.config.useTemplateFragments = false;
htmx.config.wsReconnectDelay = 'full-jitter';
htmx.config.disableSelector = '[hx-disable], [data-hx-disable]';
htmx.config.scrollBehavior = 'smooth';
htmx.config.defaultFocusScroll = false;
htmx.config.getCacheBusterParam = false;
htmx.config.globalViewTransitions = false;
htmx.config.methodsThatUseUrlParams = ["get"];
</script>
```

### Best Practices

#### 1. Security

**Always escape user content** in n8n:
```javascript
function escapeHtml(unsafe) {
    return unsafe
        .replace(/&/g, "&amp;")
        .replace(/</g, "&lt;")
        .replace(/>/g, "&gt;")
        .replace(/"/g, "&quot;")
        .replace(/'/g, "&#039;");
}

const html = `<p>${escapeHtml(userInput)}</p>`;
```

**CSRF protection**:
```html
<body hx-headers='{"X-CSRF-Token": "{{ csrfToken }}"}'>
```

#### 2. Performance

**Use hx-sync to prevent race conditions**:
```html
<input hx-get="/search"
       hx-trigger="keyup changed delay:500ms"
       hx-sync="this:abort">  <!-- Abort old searches -->
```

**Return HTML fragments, not full pages**:
```javascript
// ❌ Bad - wasteful
return { html: '<!DOCTYPE html><html>...</html>' };

// ✅ Good - minimal
return { html: '<div>Updated content</div>' };
```

**Use hx-select to extract part of response**:
```html
<div hx-get="/full-page"
     hx-select="#content"
     hx-target="#here">
```

#### 3. User Experience

**Always show loading state**:
```html
<button hx-post="/slow" hx-indicator="#spinner">
    Submit
    <span id="spinner" class="htmx-indicator">⏳</span>
</button>
```

**Disable buttons during requests**:
```html
<button hx-post="/submit" hx-disabled-elt="this">
    Submit
</button>
```

**Smooth transitions**:
```css
.htmx-swapping {
    opacity: 0;
    transition: opacity 200ms ease-out;
}
```

#### 4. Error Handling

**Handle errors gracefully**:
```html
<div hx-get="/data"
     hx-on:htmx:response-error="this.innerHTML='<p>Error loading data</p>'">
</div>
```

**Or in n8n, return error HTML**:
```javascript
try {
    const data = await fetchData();
    return { html: renderData(data) };
} catch (error) {
    return {
        html: '<div role="alert">⚠️ Something went wrong. Please try again.</div>',
        statusCode: 500
    };
}
```

#### 5. Accessibility

**Use semantic HTML**:
```html
<!-- ✅ Good -->
<button hx-post="/action">Click me</button>

<!-- ❌ Bad -->
<div onclick="..." hx-post="/action">Click me</div>
```

**ARIA labels**:
```html
<button hx-delete="/item/1"
        aria-label="Delete item"
        hx-confirm="Are you sure?">
    🗑️
</button>
```

**Announce dynamic changes**:
```html
<div role="status" aria-live="polite" id="results">
    <!-- HTMX updates here will be announced to screen readers -->
</div>
```

### Common Patterns for n8n

#### Pattern: Pagination

```html
<div id="results">
    <!-- Page 1 content -->
</div>

<button hx-get="/items?page=2"
        hx-target="#results"
        hx-swap="beforeend">
    Load More
</button>
```

#### Pattern: Master-Detail

```html
<div class="grid">
    <!-- Master list -->
    <ul id="items">
        <li hx-get="/item/1" hx-target="#detail">Item 1</li>
        <li hx-get="/item/2" hx-target="#detail">Item 2</li>
    </ul>

    <!-- Detail view -->
    <div id="detail">
        Select an item
    </div>
</div>
```

#### Pattern: Inline Editing

```html
<div id="contact-1">
    <span>John Doe</span>
    <button hx-get="/edit/1" hx-target="#contact-1" hx-swap="outerHTML">
        Edit
    </button>
</div>

<!-- n8n returns -->
<div id="contact-1">
    <input type="text" value="John Doe">
    <button hx-put="/save/1"
            hx-target="#contact-1"
            hx-swap="outerHTML"
            hx-include="previous input">
        Save
    </button>
</div>
```

#### Pattern: Dependent Dropdowns

```html
<select name="country"
        hx-get="/states"
        hx-trigger="change"
        hx-target="#state-select"
        hx-include="this">
    <option value="US">United States</option>
    <option value="CA">Canada</option>
</select>

<select id="state-select" name="state">
    <option>Select country first...</option>
</select>
```

#### Pattern: Active Search

```html
<input type="search"
       name="q"
       placeholder="Search..."
       hx-get="/search"
       hx-trigger="keyup changed delay:500ms"
       hx-target="#results"
       hx-indicator="#spinner">

<span id="spinner" class="htmx-indicator">Searching...</span>
<div id="results"></div>
```

### Debugging

#### Enable Logging

```javascript
// In browser console
htmx.logAll();
```

#### Common Issues

**1. Response not updating**:
- Check `hx-target` selector is correct
- Verify n8n returns HTML (not JSON)
- Check `Content-Type: text/html` header

**2. CORS errors**:
```javascript
// In n8n response headers
{
    'Access-Control-Allow-Origin': '*',
    'Access-Control-Allow-Methods': 'GET, POST, PUT, DELETE'
}
```

**3. Form not submitting**:
- Ensure button type="submit"
- Check `hx-post` is on `<form>` tag
- Verify n8n webhook path matches

**4. History not working**:
- Add `hx-push-url="true"`
- Ensure full page HTML on history navigation
- Check browser console for errors
