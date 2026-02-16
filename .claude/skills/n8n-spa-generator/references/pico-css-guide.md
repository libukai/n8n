# Pico.css Complete Guide

## Core Concepts

### Philosophy: Semantic CSS

Pico.css is a **classless** CSS framework that styles semantic HTML automatically. Write HTML the way it was meant to be written, and Pico makes it beautiful.

**Key principles**:
1. **Semantic HTML first**: Use proper HTML elements
2. **Minimal classes**: Only when semantic HTML isn't enough
3. **Accessibility built-in**: ARIA attributes trigger styles
4. **Responsive by default**: Mobile-first, scales up
5. **Dark mode ready**: Theme switching with one attribute

**Comparison**:

```html
<!-- Tailwind CSS -->
<button class="px-6 py-3 bg-blue-600 text-white font-semibold rounded-lg shadow-md hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-blue-500">
    Click Me
</button>

<!-- Pico.css -->
<button>Click Me</button>
```

Both look professional. Pico has 95% less code.

### File Size

**Production builds**:
- Pico.css: ~10KB (gzipped)
- Tailwind CSS: ~100KB+ (with common utilities)
- Bootstrap: ~60KB

## Getting Started

### CDN Installation

```html
<head>
    <!-- Pico CSS -->
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/@picocss/pico@2/css/pico.min.css">
</head>
```

### Theme Selection

```html
<!-- Light theme (default) -->
<html data-theme="light">

<!-- Dark theme -->
<html data-theme="dark">

<!-- Auto (respects system preference) -->
<html>
```

### Basic Structure

```html
<!DOCTYPE html>
<html lang="en" data-theme="light">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>My Page</title>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/@picocss/pico@2/css/pico.min.css">
</head>
<body>
    <main class="container">
        <!-- Your content -->
    </main>
</body>
</html>
```

## Layout

### Container

Centers content with responsive max-width:

```html
<main class="container">
    <!-- Content auto-centered, responsive padding -->
</main>
```

**Breakpoints**:
- Mobile: 100% width, 1rem padding
- Tablet (576px+): 540px max-width
- Desktop (768px+): 720px max-width
- Large (992px+): 960px max-width
- XL (1200px+): 1140px max-width

### Grid

Auto-responsive grid system:

```html
<!-- 2-column grid (auto-responsive) -->
<div class="grid">
    <div>Column 1</div>
    <div>Column 2</div>
</div>

<!-- 3-column grid -->
<div class="grid">
    <div>Column 1</div>
    <div>Column 2</div>
    <div>Column 3</div>
</div>

<!-- 4-column grid -->
<div class="grid">
    <div>1</div>
    <div>2</div>
    <div>3</div>
    <div>4</div>
</div>
```

**Responsive behavior**:
- Mobile: Stacks vertically
- Tablet+: Shows columns side-by-side
- Equal width columns automatically

## Typography

### Headings

All headings styled automatically:

```html
<h1>Heading 1</h1>  <!-- 2rem, bold -->
<h2>Heading 2</h2>  <!-- 1.75rem, bold -->
<h3>Heading 3</h3>  <!-- 1.5rem, bold -->
<h4>Heading 4</h4>  <!-- 1.25rem, bold -->
<h5>Heading 5</h5>  <!-- 1.125rem, bold -->
<h6>Heading 6</h6>  <!-- 1rem, bold -->
```

### Paragraphs and Text

```html
<p>Regular paragraph text with optimal line-height and spacing.</p>

<p><strong>Bold text</strong> and <em>italic text</em> and <mark>highlighted text</mark>.</p>

<p><small>Small text for fine print</small></p>

<p><u>Underlined text</u> and <s>strikethrough</s> and <code>inline code</code>.</p>

<blockquote>
    "A quote from someone important."
    <footer>
        <cite>— Author Name</cite>
    </footer>
</blockquote>

<pre><code>function hello() {
    console.log('Code block');
}</code></pre>
```

### Links

```html
<p>Visit our <a href="#">website</a> for more information.</p>

<!-- Links automatically styled with color and hover effect -->
```

### Lists

```html
<!-- Unordered list -->
<ul>
    <li>First item</li>
    <li>Second item</li>
    <li>Third item</li>
</ul>

<!-- Ordered list -->
<ol>
    <li>Step one</li>
    <li>Step two</li>
    <li>Step three</li>
</ol>

<!-- Description list -->
<dl>
    <dt>Term 1</dt>
    <dd>Definition 1</dd>
    <dt>Term 2</dt>
    <dd>Definition 2</dd>
</dl>
```

## Buttons

### Basic Buttons

```html
<!-- Primary button (default) -->
<button>Primary Button</button>

<!-- Secondary button -->
<button class="secondary">Secondary</button>

<!-- Contrast button -->
<button class="contrast">Contrast</button>
```

### Button Variants

```html
<!-- Outline buttons -->
<button class="outline">Outline Primary</button>
<button class="outline secondary">Outline Secondary</button>
<button class="outline contrast">Outline Contrast</button>

<!-- Disabled button -->
<button disabled>Disabled</button>

<!-- Loading button -->
<button aria-busy="true">Loading...</button>
```

### Links as Buttons

```html
<a href="#" role="button">Link Button</a>
<a href="#" role="button" class="secondary">Secondary Link</a>
<a href="#" role="button" class="outline">Outline Link</a>
```

### Button Groups

```html
<div role="group">
    <button>Button 1</button>
    <button>Button 2</button>
    <button>Button 3</button>
</div>
```

## Forms

### Basic Form

```html
<form>
    <!-- Text input -->
    <label>
        Full Name
        <input type="text" name="name" placeholder="John Doe" required>
    </label>

    <!-- Email input -->
    <label>
        Email
        <input type="email" name="email" placeholder="john@example.com" required>
        <small>We'll never share your email.</small>
    </label>

    <!-- Password -->
    <label>
        Password
        <input type="password" name="password" required>
    </label>

    <button type="submit">Submit</button>
</form>
```

### Input Types

```html
<!-- Text input -->
<input type="text" placeholder="Text">

<!-- Email -->
<input type="email" placeholder="email@example.com">

<!-- Password -->
<input type="password" placeholder="Password">

<!-- Number -->
<input type="number" placeholder="42">

<!-- Tel -->
<input type="tel" placeholder="(123) 456-7890">

<!-- URL -->
<input type="url" placeholder="https://example.com">

<!-- Search -->
<input type="search" placeholder="Search...">

<!-- Date -->
<input type="date">

<!-- Time -->
<input type="time">

<!-- Color -->
<input type="color">

<!-- File -->
<input type="file">
```

### Textarea

```html
<label>
    Message
    <textarea name="message" placeholder="Your message..." rows="4"></textarea>
</label>
```

### Select

```html
<label>
    Choose option
    <select name="option">
        <option value="">Select...</option>
        <option value="1">Option 1</option>
        <option value="2">Option 2</option>
        <option value="3">Option 3</option>
    </select>
</label>

<!-- Multiple select -->
<label>
    Choose multiple
    <select name="options[]" multiple>
        <option>Option 1</option>
        <option>Option 2</option>
        <option>Option 3</option>
    </select>
</label>
```

### Checkboxes

```html
<!-- Single checkbox -->
<label>
    <input type="checkbox" name="agree">
    I agree to the terms
</label>

<!-- Multiple checkboxes -->
<fieldset>
    <legend>Select your interests</legend>
    <label>
        <input type="checkbox" name="interest[]" value="tech">
        Technology
    </label>
    <label>
        <input type="checkbox" name="interest[]" value="design">
        Design
    </label>
    <label>
        <input type="checkbox" name="interest[]" value="business">
        Business
    </label>
</fieldset>
```

### Radio Buttons

```html
<fieldset>
    <legend>Choose one</legend>
    <label>
        <input type="radio" name="size" value="small">
        Small
    </label>
    <label>
        <input type="radio" name="size" value="medium" checked>
        Medium
    </label>
    <label>
        <input type="radio" name="size" value="large">
        Large
    </label>
</fieldset>
```

### Switch

```html
<label>
    <input type="checkbox" name="notifications" role="switch">
    Enable notifications
</label>

<!-- Switch with description -->
<label>
    <input type="checkbox" name="dark-mode" role="switch">
    Dark mode
    <small>Use dark color scheme</small>
</label>
```

### Range

```html
<label>
    Volume
    <input type="range" name="volume" min="0" max="100" value="50">
</label>

<!-- With labels -->
<label>
    <input type="range" min="0" max="100">
    <div style="display: flex; justify-content: space-between;">
        <small>0</small>
        <small>100</small>
    </div>
</label>
```

### Validation States

```html
<!-- Valid input -->
<input type="email" aria-invalid="false" value="valid@email.com">

<!-- Invalid input -->
<input type="email" aria-invalid="true" value="invalid">

<!-- With helper text -->
<label>
    Email
    <input type="email" name="email" aria-invalid="true">
    <small>Please enter a valid email address</small>
</label>
```

### Fieldset

```html
<fieldset>
    <legend>Personal Information</legend>

    <label>
        First Name
        <input type="text" name="first-name">
    </label>

    <label>
        Last Name
        <input type="text" name="last-name">
    </label>
</fieldset>
```

## Tables

### Basic Table

```html
<table>
    <thead>
        <tr>
            <th>Name</th>
            <th>Email</th>
            <th>Role</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Alice Johnson</td>
            <td>alice@example.com</td>
            <td>Admin</td>
        </tr>
        <tr>
            <td>Bob Smith</td>
            <td>bob@example.com</td>
            <td>User</td>
        </tr>
    </tbody>
</table>
```

### Striped Table

```html
<table role="grid">
    <!-- Adds zebra striping -->
    <thead>
        <tr>
            <th>Product</th>
            <th>Price</th>
            <th>Stock</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Item 1</td>
            <td>$10</td>
            <td>50</td>
        </tr>
        <tr>
            <td>Item 2</td>
            <td>$20</td>
            <td>30</td>
        </tr>
    </tbody>
</table>
```

## Components

### Cards (Articles)

```html
<!-- Basic card -->
<article>
    <h2>Card Title</h2>
    <p>Card content goes here.</p>
</article>

<!-- Card with header and footer -->
<article>
    <header>
        <h2>Card Header</h2>
    </header>
    <p>Main content of the card.</p>
    <footer>
        <button>Action</button>
    </footer>
</article>
```

### Navigation

```html
<nav>
    <ul>
        <li><strong>Brand</strong></li>
    </ul>
    <ul>
        <li><a href="#">Home</a></li>
        <li><a href="#">About</a></li>
        <li><a href="#">Contact</a></li>
    </ul>
</nav>
```

### Accordion (Details)

```html
<details>
    <summary>What is Pico.css?</summary>
    <p>Pico.css is a minimal CSS framework for semantic HTML.</p>
</details>

<details>
    <summary>How to install?</summary>
    <p>Just add the CDN link to your HTML.</p>
</details>

<!-- Open by default -->
<details open>
    <summary>Already open</summary>
    <p>This accordion starts expanded.</p>
</details>
```

### Modal (Dialog)

```html
<dialog open>
    <article>
        <header>
            <button aria-label="Close" rel="prev"></button>
            <h2>Modal Title</h2>
        </header>
        <p>Modal content goes here.</p>
        <footer>
            <button class="secondary">Cancel</button>
            <button>Confirm</button>
        </footer>
    </article>
</dialog>
```

### Dropdown

```html
<details class="dropdown">
    <summary>Dropdown Menu</summary>
    <ul>
        <li><a href="#">Option 1</a></li>
        <li><a href="#">Option 2</a></li>
        <li><a href="#">Option 3</a></li>
    </ul>
</details>
```

### Progress

```html
<!-- Determinate progress -->
<progress value="50" max="100"></progress>

<!-- Indeterminate progress (loading) -->
<progress></progress>
```

### Loading State

```html
<!-- Loading button -->
<button aria-busy="true">Loading...</button>

<!-- Loading card -->
<article aria-busy="true">
    Loading content...
</article>

<!-- Loading div -->
<div aria-busy="true">
    Loading...
</div>
```

## Utility Classes

Pico has minimal classes, but these are available:

### Container

```html
<main class="container">
    <!-- Centered, responsive content -->
</main>
```

### Grid

```html
<div class="grid">
    <div>Col 1</div>
    <div>Col 2</div>
</div>
```

### Button Variants

```html
<button class="secondary">Secondary</button>
<button class="contrast">Contrast</button>
<button class="outline">Outline</button>
```

### Role Attributes

```html
<button role="button">Link as Button</button>
<table role="grid">Striped Table</table>
<details class="dropdown">Dropdown</details>
```

## Customization with CSS Variables

Pico uses CSS custom properties for theming:

### Color Variables

```css
:root {
    /* Primary colors */
    --pico-primary: #4F46E5;
    --pico-primary-hover: #4338CA;
    --pico-primary-focus: rgba(79, 70, 229, 0.125);
    --pico-primary-inverse: #FFF;

    /* Secondary colors */
    --pico-secondary: #64748B;
    --pico-secondary-hover: #475569;
    --pico-secondary-focus: rgba(100, 116, 139, 0.125);
    --pico-secondary-inverse: #FFF;

    /* Contrast colors */
    --pico-contrast: #1E293B;
    --pico-contrast-hover: #0F172A;
    --pico-contrast-focus: rgba(30, 41, 59, 0.125);
    --pico-contrast-inverse: #FFF;

    /* Background */
    --pico-background-color: #FFF;

    /* Text */
    --pico-color: #374151;
    --pico-h1-color: #1F2937;
    --pico-h2-color: #1F2937;
    --pico-h3-color: #1F2937;
    --pico-muted-color: #6B7280;

    /* Links */
    --pico-text-selection-color: rgba(79, 70, 229, 0.25);

    /* Form elements */
    --pico-form-element-background-color: #FFF;
    --pico-form-element-border-color: #D1D5DB;
    --pico-form-element-color: #374151;
    --pico-form-element-placeholder-color: #9CA3AF;

    /* Validation */
    --pico-form-element-valid-border-color: #10B981;
    --pico-form-element-invalid-border-color: #EF4444;

    /* Buttons */
    --pico-button-box-shadow: 0 1px 2px 0 rgba(0, 0, 0, 0.05);
    --pico-button-hover-box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1);
}
```

### Typography Variables

```css
:root {
    /* Font family */
    --pico-font-family: system-ui, -apple-system, "Segoe UI", "Roboto", sans-serif;

    /* Font sizes */
    --pico-font-size: 16px;
    --pico-line-height: 1.5;
    --pico-font-weight: 400;

    /* Headings */
    --pico-h1-font-size: 2rem;
    --pico-h2-font-size: 1.75rem;
    --pico-h3-font-size: 1.5rem;
    --pico-h4-font-size: 1.25rem;
    --pico-h5-font-size: 1.125rem;
    --pico-h6-font-size: 1rem;
}
```

### Spacing Variables

```css
:root {
    /* Block spacing */
    --pico-spacing: 1rem;
    --pico-typography-spacing-vertical: 1.5rem;
    --pico-block-spacing-vertical: calc(var(--pico-spacing) * 2);
    --pico-block-spacing-horizontal: var(--pico-spacing);

    /* Form elements */
    --pico-form-element-spacing-vertical: 0.75rem;
    --pico-form-element-spacing-horizontal: 1rem;
}
```

### Border Variables

```css
:root {
    /* Border radius */
    --pico-border-radius: 0.25rem;

    /* Border width */
    --pico-border-width: 1px;

    /* Outline width */
    --pico-outline-width: 2px;
}
```

### Example Customization

```html
<style>
:root {
    /* Custom brand colors */
    --pico-primary: #FF6B6B;
    --pico-primary-hover: #FF5252;

    /* Custom font */
    --pico-font-family: 'Inter', sans-serif;

    /* Custom spacing */
    --pico-spacing: 1.25rem;

    /* Custom border radius */
    --pico-border-radius: 0.5rem;
}

/* Dark theme customization */
[data-theme="dark"] {
    --pico-background-color: #1A1A1A;
    --pico-color: #E5E5E5;
}
</style>
```

## Dark Mode

### Automatic Dark Mode

```html
<!-- Auto (respects system preference) -->
<html>
```

### Manual Dark Mode

```html
<!-- Force light -->
<html data-theme="light">

<!-- Force dark -->
<html data-theme="dark">
```

### Toggle Dark Mode

```html
<button onclick="toggleTheme()">Toggle Theme</button>

<script>
function toggleTheme() {
    const html = document.documentElement;
    const currentTheme = html.getAttribute('data-theme');
    const newTheme = currentTheme === 'dark' ? 'light' : 'dark';
    html.setAttribute('data-theme', newTheme);
    localStorage.setItem('theme', newTheme);
}

// Load saved theme
const savedTheme = localStorage.getItem('theme');
if (savedTheme) {
    document.documentElement.setAttribute('data-theme', savedTheme);
}
</script>
```

### Dark Mode with HTMX

```html
<button hx-post="/set-theme/dark"
        hx-target="body"
        _="on click set @data-theme to 'dark' on <html/>">
    Dark Mode
</button>

<button hx-post="/set-theme/light"
        hx-target="body"
        _="on click set @data-theme to 'light' on <html/>">
    Light Mode
</button>
```

## Best Practices

### 1. Use Semantic HTML

```html
<!-- ✅ Good -->
<nav>
    <ul>
        <li><a href="/">Home</a></li>
    </ul>
</nav>

<!-- ❌ Bad -->
<div class="navigation">
    <div class="nav-list">
        <div class="nav-item"><a href="/">Home</a></div>
    </div>
</div>
```

### 2. Leverage ARIA Attributes

```html
<!-- Invalid input -->
<input type="email" aria-invalid="true">

<!-- Loading state -->
<button aria-busy="true">Loading...</button>

<!-- Switch -->
<input type="checkbox" role="switch">
```

### 3. Minimal Custom CSS

Pico handles most styling. Only add custom CSS when needed:

```html
<style>
/* Only override when absolutely necessary */
.custom-spacing {
    margin-top: calc(var(--pico-spacing) * 2);
}
</style>
```

### 4. Mobile-First Responsive

Pico is mobile-first. Test on mobile devices:

```html
<meta name="viewport" content="width=device-width, initial-scale=1.0">
```

### 5. Container for Centering

```html
<body>
    <main class="container">
        <!-- All content here auto-centered -->
    </main>
</body>
```

## Common Patterns

### Pattern: Form with Validation

```html
<form>
    <label>
        Email
        <input type="email"
               name="email"
               aria-invalid="false"
               required>
    </label>

    <label>
        Password
        <input type="password"
               name="password"
               aria-invalid="false"
               required>
        <small>At least 8 characters</small>
    </label>

    <button type="submit">Sign In</button>
</form>
```

### Pattern: Card Grid

```html
<div class="grid">
    <article>
        <header>Card 1</header>
        <p>Content 1</p>
    </article>

    <article>
        <header>Card 2</header>
        <p>Content 2</p>
    </article>

    <article>
        <header>Card 3</header>
        <p>Content 3</p>
    </article>
</div>
```

### Pattern: Hero Section

```html
<header class="container">
    <hgroup>
        <h1>Welcome to Our Site</h1>
        <p>A brief description of what we do</p>
    </hgroup>
    <p>
        <button>Get Started</button>
        <button class="secondary">Learn More</button>
    </p>
</header>
```

### Pattern: Alert/Notification

```html
<!-- Success -->
<div role="alert">
    ✅ Operation completed successfully!
</div>

<!-- Error -->
<div role="alert" style="--pico-background-color: var(--pico-form-element-invalid-border-color);">
    ❌ An error occurred. Please try again.
</div>

<!-- Info -->
<div role="alert" style="--pico-background-color: var(--pico-primary);">
    ℹ️ New features available!
</div>
```

## Integration with HTMX

Pico works seamlessly with HTMX:

```html
<!-- Form with HTMX -->
<form hx-post="/submit" hx-target="#result">
    <label>
        Name
        <input type="text" name="name" required>
    </label>

    <button type="submit" aria-busy="false"
            hx-indicator="this"
            hx-disabled-elt="this">
        Submit
    </button>
</form>

<div id="result"></div>

<!-- n8n returns Pico-styled HTML -->
<!-- Response:
<div role="alert">
    ✅ Form submitted successfully!
</div>
-->
```

## Accessibility

Pico is built with accessibility in mind:

- Semantic HTML elements
- Proper color contrast ratios
- ARIA attributes trigger styles
- Keyboard navigation support
- Screen reader friendly
- Focus indicators on interactive elements

**Always include**:
```html
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<html lang="en">
<title>Descriptive page title</title>
```

## Browser Support

Pico supports all modern browsers:
- Chrome/Edge (latest)
- Firefox (latest)
- Safari (latest)
- Mobile browsers

No IE11 support (uses modern CSS features).
