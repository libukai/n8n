# Alpine.js Complete Guide

Alpine.js is a minimal JavaScript framework for composing behavior directly in your markup. It offers the reactive and declarative nature of big frameworks like Vue or React at a much lower cost.

**Version**: Alpine.js 3.x
**Size**: ~39KB (minified)
**CDN**: `https://cdn.jsdelivr.net/npm/alpinejs@3/dist/cdn.min.js`

## Philosophy

Alpine.js follows these core principles:

1. **Declarative**: Behavior is defined in HTML attributes
2. **Reactive**: Data changes automatically update the DOM
3. **Minimal**: Small API surface, easy to learn
4. **No Build Step**: Works directly in the browser
5. **Progressive**: Can be added to existing projects incrementally

## Installation

```html
<!-- Add to <head> with defer attribute -->
<script defer src="https://cdn.jsdelivr.net/npm/alpinejs@3/dist/cdn.min.js"></script>
```

The `defer` attribute ensures Alpine loads after the HTML is parsed.

## Core Directives

### x-data

Declares a new Alpine component scope and defines its reactive data.

**Syntax**:
```html
<div x-data="{ propertyName: value, ... }">
    <!-- Component scope -->
</div>
```

**Examples**:

```html
<!-- Simple data -->
<div x-data="{ open: false, count: 0 }">
    <!-- Access data here -->
</div>

<!-- With methods -->
<div x-data="{
    count: 0,
    increment() {
        this.count++
    },
    decrement() {
        this.count--
    }
}">
    <button @click="decrement()">-</button>
    <span x-text="count"></span>
    <button @click="increment()">+</button>
</div>

<!-- Component function (reusable) -->
<div x-data="dropdown()">
    <!-- Component content -->
</div>

<script>
function dropdown() {
    return {
        open: false,
        toggle() {
            this.open = !this.open
        }
    }
}
</script>
```

**Nested Components**:
```html
<div x-data="{ foo: 'bar' }">
    <span x-text="foo"></span> <!-- "bar" -->

    <div x-data="{ foo: 'baz' }">
        <span x-text="foo"></span> <!-- "baz" (shadowed) -->
    </div>
</div>
```

### x-bind

Dynamically bind attributes to JavaScript expressions.

**Syntax**:
```html
<element x-bind:attribute="expression"></element>
<!-- Shorthand -->
<element :attribute="expression"></element>
```

**Examples**:

```html
<!-- Class binding -->
<div :class="isActive ? 'bg-blue' : 'bg-gray'"></div>

<!-- Style binding -->
<div :style="{ color: textColor, fontSize: size + 'px' }"></div>

<!-- Boolean attributes -->
<button :disabled="isLoading">Submit</button>
<input :required="isRequired" :readonly="!isEditable">

<!-- Multiple attributes -->
<a :href="url" :title="linkTitle">Link</a>

<!-- Spread attributes -->
<div x-data="{ attrs: { class: 'foo', 'data-id': '123' } }">
    <span x-bind="attrs"></span>
    <!-- Results in: <span class="foo" data-id="123"></span> -->
</div>
```

**Class Object Syntax**:
```html
<div :class="{
    'active': isActive,
    'text-danger': hasError,
    'font-bold': isBold
}"></div>
```

**Class Array Syntax**:
```html
<div :class="[baseClass, isActive ? 'active' : '']"></div>
```

### x-on

Listen to DOM events and execute JavaScript.

**Syntax**:
```html
<element x-on:event="expression"></element>
<!-- Shorthand -->
<element @event="expression"></element>
```

**Examples**:

```html
<!-- Click event -->
<button @click="count++">Increment</button>
<button @click="handleClick()">Click Me</button>

<!-- Input events -->
<input @input="searchQuery = $event.target.value">
<input @keyup.enter="submitForm()">
<input @keyup.escape="clearInput()">

<!-- Form events -->
<form @submit.prevent="handleSubmit()">
    <button type="submit">Submit</button>
</form>

<!-- Mouse events -->
<div @mouseenter="isHovered = true" @mouseleave="isHovered = false">
    Hover me
</div>

<!-- Custom events -->
<div @custom-event="handleCustom($event.detail)"></div>
```

**Event Modifiers**:

```html
<!-- Prevent default -->
<form @submit.prevent="handleSubmit()"></form>

<!-- Stop propagation -->
<button @click.stop="handleClick()"></button>

<!-- Run only once -->
<button @click.once="runOnce()"></button>

<!-- Passive listener (better scroll performance) -->
<div @scroll.passive="handleScroll()"></div>

<!-- Capture phase -->
<div @click.capture="handleClick()"></div>

<!-- Self (only if event.target is element itself) -->
<div @click.self="handleClick()"></div>

<!-- Debounce (wait 500ms after last event) -->
<input @input.debounce.500ms="search()">

<!-- Throttle (run at most once per 1000ms) -->
<div @scroll.throttle.1000ms="handleScroll()"></div>
```

**Key Modifiers**:

```html
<input @keyup.enter="submit()">
<input @keyup.escape="cancel()">
<input @keyup.tab="nextField()">
<input @keyup.delete="removeItem()">
<input @keyup.backspace="goBack()">
<input @keyup.arrow-up="moveUp()">
<input @keyup.arrow-down="moveDown()">
<input @keyup.page-up="pageUp()">
<input @keyup.page-down="pageDown()">

<!-- Modifier keys -->
<button @click.ctrl="handleCtrlClick()"></button>
<button @click.shift="handleShiftClick()"></button>
<button @click.alt="handleAltClick()"></button>
<button @click.meta="handleMetaClick()"></button>
```

**$event Magic Property**:

```html
<button @click="handleClick($event)">Click</button>

<script>
function handleClick(event) {
    console.log(event.target)
    console.log(event.type)
}
</script>
```

### x-model

Two-way data binding for form inputs.

**Syntax**:
```html
<input x-model="propertyName">
```

**Examples**:

```html
<!-- Text input -->
<div x-data="{ message: '' }">
    <input type="text" x-model="message">
    <p>You typed: <span x-text="message"></span></p>
</div>

<!-- Number input -->
<div x-data="{ age: 0 }">
    <input type="number" x-model.number="age">
    <p>Age: <span x-text="age"></span> (type: <span x-text="typeof age"></span>)</p>
</div>

<!-- Checkbox -->
<div x-data="{ checked: false }">
    <input type="checkbox" x-model="checked">
    <p>Checked: <span x-text="checked"></span></p>
</div>

<!-- Multiple checkboxes -->
<div x-data="{ colors: [] }">
    <input type="checkbox" value="red" x-model="colors">Red
    <input type="checkbox" value="blue" x-model="colors">Blue
    <input type="checkbox" value="green" x-model="colors">Green
    <p>Selected: <span x-text="colors.join(', ')"></span></p>
</div>

<!-- Radio buttons -->
<div x-data="{ choice: '' }">
    <input type="radio" value="yes" x-model="choice">Yes
    <input type="radio" value="no" x-model="choice">No
    <p>Choice: <span x-text="choice"></span></p>
</div>

<!-- Select dropdown -->
<div x-data="{ selected: '' }">
    <select x-model="selected">
        <option value="">Choose...</option>
        <option value="a">Option A</option>
        <option value="b">Option B</option>
    </select>
    <p>Selected: <span x-text="selected"></span></p>
</div>

<!-- Textarea -->
<div x-data="{ content: '' }">
    <textarea x-model="content"></textarea>
    <p>Length: <span x-text="content.length"></span></p>
</div>
```

**Modifiers**:

```html
<!-- .lazy - Update on "change" instead of "input" -->
<input x-model.lazy="message">

<!-- .number - Convert to number -->
<input x-model.number="age">

<!-- .debounce - Debounce updates (500ms default) -->
<input x-model.debounce="search">
<input x-model.debounce.1000ms="search">

<!-- .throttle - Throttle updates -->
<input x-model.throttle.500ms="search">

<!-- .fill - Pre-fill from server-rendered value -->
<input x-model.fill="username" value="john_doe">
```

### x-text

Set the text content of an element.

**Syntax**:
```html
<element x-text="expression"></element>
```

**Examples**:

```html
<div x-data="{ username: 'Alice' }">
    <p>Hello, <span x-text="username"></span>!</p>
</div>

<!-- With expressions -->
<div x-data="{ count: 5 }">
    <p x-text="count * 2"></p> <!-- 10 -->
    <p x-text="'Count: ' + count"></p> <!-- Count: 5 -->
</div>

<!-- Escapes HTML automatically (safe from XSS) -->
<div x-data="{ html: '<script>alert(1)</script>' }">
    <p x-text="html"></p> <!-- Shows as text, not executed -->
</div>
```

### x-html

Set the innerHTML of an element.

**Syntax**:
```html
<element x-html="expression"></element>
```

**Examples**:

```html
<div x-data="{ content: '<strong>Bold text</strong>' }">
    <div x-html="content"></div> <!-- Renders: <strong>Bold text</strong> -->
</div>
```

**Warning**: Only use with trusted content. Never use with user-generated content (XSS risk).

### x-show

Toggle visibility using `display: none`.

**Syntax**:
```html
<element x-show="expression"></element>
```

**Examples**:

```html
<!-- Basic toggle -->
<div x-data="{ open: false }">
    <button @click="open = !open">Toggle</button>
    <div x-show="open">
        This content can be hidden
    </div>
</div>

<!-- With transition -->
<div x-data="{ show: false }">
    <button @click="show = !show">Toggle</button>
    <div x-show="show" x-transition>
        Fades in and out
    </div>
</div>

<!-- Conditional display -->
<div x-data="{ user: null }">
    <div x-show="user !== null">
        Welcome back!
    </div>
    <div x-show="user === null">
        Please log in
    </div>
</div>
```

**Modifiers**:

```html
<!-- .important - Use !important on display style -->
<div x-show.important="isVisible"></div>
```

### x-if

Conditionally add/remove elements from DOM (unlike x-show which uses CSS).

**Syntax**:
```html
<template x-if="expression">
    <element>...</element>
</template>
```

**Examples**:

```html
<!-- Basic conditional -->
<div x-data="{ loggedIn: false }">
    <template x-if="loggedIn">
        <div>Welcome back!</div>
    </template>

    <template x-if="!loggedIn">
        <div>Please log in</div>
    </template>
</div>

<!-- With complex conditions -->
<div x-data="{ userType: 'guest' }">
    <template x-if="userType === 'admin'">
        <div>Admin Panel</div>
    </template>

    <template x-if="userType === 'user'">
        <div>User Dashboard</div>
    </template>

    <template x-if="userType === 'guest'">
        <div>Guest View</div>
    </template>
</div>
```

**x-if vs x-show**:

- `x-if`: Removes element from DOM. Better for heavy components that are rarely shown.
- `x-show`: Hides with CSS. Better for frequently toggled elements.

### x-for

Loop through arrays and objects.

**Syntax**:
```html
<template x-for="item in items" :key="item.id">
    <element>...</element>
</template>
```

**Examples**:

```html
<!-- Array of primitives -->
<div x-data="{ colors: ['red', 'blue', 'green'] }">
    <ul>
        <template x-for="color in colors" :key="color">
            <li x-text="color"></li>
        </template>
    </ul>
</div>

<!-- Array of objects -->
<div x-data="{
    users: [
        { id: 1, name: 'Alice' },
        { id: 2, name: 'Bob' }
    ]
}">
    <ul>
        <template x-for="user in users" :key="user.id">
            <li x-text="user.name"></li>
        </template>
    </ul>
</div>

<!-- With index -->
<div x-data="{ items: ['a', 'b', 'c'] }">
    <template x-for="(item, index) in items" :key="index">
        <p><span x-text="index"></span>: <span x-text="item"></span></p>
    </template>
</div>

<!-- Range (numbers 1-5) -->
<template x-for="i in 5" :key="i">
    <div x-text="i"></div>
</template>

<!-- Objects -->
<div x-data="{ person: { name: 'Alice', age: 30 } }">
    <template x-for="(value, key) in person" :key="key">
        <p><span x-text="key"></span>: <span x-text="value"></span></p>
    </template>
</div>
```

**Important**: Always provide a `:key` attribute for efficient DOM updates.

### x-transition

Add CSS transitions when toggling visibility.

**Syntax**:
```html
<div x-show="open" x-transition>...</div>
```

**Examples**:

```html
<!-- Default transition (fade + scale) -->
<div x-show="open" x-transition>
    Fades and scales
</div>

<!-- Customized duration -->
<div x-show="open" x-transition.duration.500ms>
    500ms transition
</div>

<!-- Opacity only -->
<div x-show="open" x-transition.opacity>
    Fades
</div>

<!-- Scale only -->
<div x-show="open" x-transition.scale>
    Scales
</div>

<!-- Combined -->
<div x-show="open" x-transition.opacity.scale.duration.300ms>
    Fades and scales in 300ms
</div>

<!-- Custom classes (full control) -->
<div x-show="open"
     x-transition:enter="transition ease-out duration-300"
     x-transition:enter-start="opacity-0 transform scale-90"
     x-transition:enter-end="opacity-100 transform scale-100"
     x-transition:leave="transition ease-in duration-300"
     x-transition:leave-start="opacity-100 transform scale-100"
     x-transition:leave-end="opacity-0 transform scale-90">
    Custom transition
</div>
```

### x-cloak

Hide elements until Alpine is loaded.

**Syntax**:
```html
<div x-data="..." x-cloak>...</div>
```

**CSS**:
```css
[x-cloak] {
    display: none !important;
}
```

**Example**:

```html
<style>
[x-cloak] { display: none !important; }
</style>

<div x-data="{ message: 'Hello' }" x-cloak>
    <p x-text="message"></p>
    <!-- Won't show until Alpine loads and processes -->
</div>
```

### x-init

Run code when an element is initialized.

**Syntax**:
```html
<div x-init="expression"></div>
```

**Examples**:

```html
<!-- Set initial value -->
<div x-data="{ message: '' }" x-init="message = 'Hello'">
    <p x-text="message"></p>
</div>

<!-- Fetch data on init -->
<div x-data="{ posts: [] }"
     x-init="posts = await (await fetch('/api/posts')).json()">
    <template x-for="post in posts" :key="post.id">
        <div x-text="post.title"></div>
    </template>
</div>

<!-- Multiple statements -->
<div x-data="{ count: 0 }"
     x-init="count = 10; console.log('Initialized')">
</div>

<!-- Access Alpine's $el -->
<div x-init="$el.classList.add('initialized')"></div>
```

### x-effect

Automatically re-run expression when dependencies change.

**Syntax**:
```html
<div x-effect="expression"></div>
```

**Examples**:

```html
<!-- Console log when count changes -->
<div x-data="{ count: 0 }" x-effect="console.log('Count is:', count)">
    <button @click="count++">Increment</button>
</div>

<!-- Update document title -->
<div x-data="{ title: 'Home' }" x-effect="document.title = title">
    <input x-model="title">
</div>

<!-- Multiple dependencies -->
<div x-data="{ firstName: 'John', lastName: 'Doe' }"
     x-effect="console.log(`Full name: ${firstName} ${lastName}`)">
</div>
```

### x-ref

Reference DOM elements directly.

**Syntax**:
```html
<div x-ref="name"></div>
<!-- Access with $refs.name -->
```

**Examples**:

```html
<!-- Focus input -->
<div x-data>
    <input x-ref="myInput" type="text">
    <button @click="$refs.myInput.focus()">Focus Input</button>
</div>

<!-- Get value -->
<div x-data>
    <input x-ref="email" type="email">
    <button @click="alert($refs.email.value)">Show Email</button>
</div>

<!-- Manipulate element -->
<div x-data>
    <div x-ref="box" style="width: 100px; height: 100px; background: red;"></div>
    <button @click="$refs.box.style.background = 'blue'">Change Color</button>
</div>
```

### x-ignore

Prevent Alpine from processing an element and its children.

**Syntax**:
```html
<div x-ignore>...</div>
```

**Example**:

```html
<div x-data="{ message: 'Hello' }">
    <p x-text="message"></p> <!-- Works -->

    <div x-ignore>
        <p x-text="message"></p> <!-- Doesn't work, Alpine ignored -->
    </div>
</div>
```

## Magic Properties

Magic properties are special variables available in Alpine expressions.

### $el

Reference to current DOM element.

```html
<div @click="console.log($el)">Click me</div>

<button @click="$el.style.background = 'red'">Red Background</button>

<input @input="console.log($el.value)">
```

### $refs

Access elements marked with `x-ref`.

```html
<div x-data>
    <input x-ref="username">
    <button @click="alert($refs.username.value)">Show</button>
</div>
```

### $store

Access global stores (covered in Advanced section).

```html
<div x-data @click="$store.app.count++">
    Count: <span x-text="$store.app.count"></span>
</div>
```

### $watch

Watch a property and run callback when it changes.

```html
<div x-data="{ count: 0 }" x-init="$watch('count', value => console.log(value))">
    <button @click="count++">Increment</button>
</div>
```

### $dispatch

Dispatch custom events.

```html
<!-- Dispatch event -->
<button @click="$dispatch('custom-event', { detail: 'data' })">
    Dispatch
</button>

<!-- Listen to event -->
<div @custom-event="console.log($event.detail)"></div>

<!-- Dispatch to window -->
<button @click="$dispatch('notify', {}, { bubbles: true })">
    Notify App
</button>
```

### $nextTick

Wait for next DOM update before running code.

```html
<div x-data="{ items: [] }">
    <button @click="items.push(Date.now()); $nextTick(() => {
        $el.scrollTop = $el.scrollHeight
    })">Add Item</button>

    <div style="max-height: 200px; overflow: auto;">
        <template x-for="item in items" :key="item">
            <div x-text="item"></div>
        </template>
    </div>
</div>
```

### $root

Access root element of component.

```html
<div x-data="{ message: 'Hello' }">
    <div>
        <button @click="console.log($root)">Log Root</button>
    </div>
</div>
```

### $data

Access entire data object.

```html
<div x-data="{ foo: 'bar', baz: 'qux' }">
    <button @click="console.log($data)">Log Data</button>
</div>
```

### $id

Generate unique IDs for accessibility.

```html
<div x-data>
    <label :for="$id('input')">Name</label>
    <input type="text" :id="$id('input')">
</div>
```

## Global Helpers

### Alpine.data()

Register reusable component data.

```javascript
Alpine.data('dropdown', () => ({
    open: false,
    toggle() {
        this.open = !this.open
    },
    close() {
        this.open = false
    }
}))
```

```html
<div x-data="dropdown">
    <button @click="toggle()">Toggle</button>
    <div x-show="open" @click.outside="close()">
        Dropdown content
    </div>
</div>
```

### Alpine.store()

Create global reactive stores.

```javascript
Alpine.store('app', {
    count: 0,
    increment() {
        this.count++
    }
})
```

```html
<div x-data>
    <button @click="$store.app.increment()">
        Count: <span x-text="$store.app.count"></span>
    </button>
</div>
```

## Best Practices

### 1. Keep Components Small

```html
<!-- ✅ Good: Small, focused component -->
<div x-data="{ open: false }">
    <button @click="open = !open">Toggle</button>
    <div x-show="open">Content</div>
</div>

<!-- ❌ Bad: Too much logic in one component -->
<div x-data="{ /* 50 properties */ }">
    <!-- Complex UI -->
</div>
```

### 2. Extract Reusable Logic

```javascript
// ✅ Good: Reusable component
Alpine.data('tabs', () => ({
    active: 0,
    setActive(index) {
        this.active = index
    }
}))
```

### 3. Use x-if for Heavy Content

```html
<!-- ✅ Good: x-if removes from DOM -->
<template x-if="showChart">
    <div><!-- Heavy chart component --></div>
</template>

<!-- ❌ Bad: x-show keeps in DOM (if rarely shown) -->
<div x-show="showChart">
    <!-- Heavy chart always in DOM -->
</div>
```

### 4. Provide Keys in x-for

```html
<!-- ✅ Good: Unique keys -->
<template x-for="user in users" :key="user.id">
    <div x-text="user.name"></div>
</template>

<!-- ❌ Bad: No key -->
<template x-for="user in users">
    <div x-text="user.name"></div>
</template>
```

### 5. Use x-cloak to Prevent Flash

```html
<style>[x-cloak] { display: none !important; }</style>

<div x-data="{ message: 'Hello' }" x-cloak>
    <p x-text="message"></p>
</div>
```

## Integration with HTMX

Alpine.js and HTMX work beautifully together. HTMX handles server communication, Alpine handles client-side interactivity.

```html
<!-- Search with HTMX + Alpine for local filtering -->
<div x-data="{ filter: '' }">
    <!-- HTMX loads initial data -->
    <div hx-get="/api/items" hx-trigger="load" hx-swap="innerHTML">
        Loading...
    </div>

    <!-- Alpine filters displayed items -->
    <input x-model="filter" placeholder="Filter items">

    <template x-for="item in items.filter(i => i.name.includes(filter))" :key="item.id">
        <div x-text="item.name"></div>
    </template>
</div>
```

```html
<!-- Modal with HTMX content + Alpine state -->
<div x-data="{ open: false }">
    <button @click="open = true">Open Modal</button>

    <div x-show="open" @click.outside="open = false">
        <div hx-get="/modal-content" hx-trigger="load">
            Loading...
        </div>
    </div>
</div>
```

## Common Patterns

### Accordion

```html
<div x-data="{ active: null }">
    <template x-for="(item, index) in items" :key="index">
        <div>
            <button @click="active = active === index ? null : index"
                    x-text="item.title">
            </button>
            <div x-show="active === index" x-collapse>
                <p x-text="item.content"></p>
            </div>
        </div>
    </template>
</div>
```

### Modal

```html
<div x-data="{ open: false }">
    <button @click="open = true">Open Modal</button>

    <div x-show="open"
         @click.self="open = false"
         x-transition.opacity
         style="position: fixed; inset: 0; background: rgba(0,0,0,0.5);">
        <div style="background: white; padding: 2rem; margin: 2rem auto; max-width: 500px;">
            <h2>Modal Title</h2>
            <p>Modal content</p>
            <button @click="open = false">Close</button>
        </div>
    </div>
</div>
```

### Autocomplete

```html
<div x-data="{
    search: '',
    items: ['Apple', 'Banana', 'Cherry'],
    get filtered() {
        return this.items.filter(i =>
            i.toLowerCase().includes(this.search.toLowerCase())
        )
    }
}">
    <input x-model="search" placeholder="Search...">
    <ul x-show="search.length > 0">
        <template x-for="item in filtered" :key="item">
            <li x-text="item" @click="search = item"></li>
        </template>
    </ul>
</div>
```

## Debugging

### Enable Development Mode

```javascript
Alpine.start()
Alpine.devtools = true // Enable in console
```

### Log Data Changes

```html
<div x-data="{ count: 0 }" x-effect="console.log('count:', count)">
    <button @click="count++">Increment</button>
</div>
```

### Inspect Component Data

Use browser console:
```javascript
// Access Alpine from console
Alpine

// Get component data
Alpine.$data(document.querySelector('[x-data]'))
```

## Performance Tips

1. **Use x-if for rarely-shown content** (removes from DOM)
2. **Debounce expensive operations** (`@input.debounce.500ms`)
3. **Use :key in x-for** for efficient updates
4. **Avoid deep nesting** of x-data components
5. **Extract complex computed values** to methods/getters

## Resources

- Official Docs: https://alpinejs.dev
- Examples: https://alpinejs.dev/examples
- Plugins: https://alpinejs.dev/plugins
- Community: https://github.com/alpinejs/alpine/discussions
