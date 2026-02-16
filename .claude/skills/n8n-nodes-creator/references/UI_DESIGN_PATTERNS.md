# UI Design Patterns for n8n Nodes

Best practices for creating intuitive, user-friendly node interfaces based on real-world implementations.

## Core Principles

### 1. Flat Over Nested ⭐⭐⭐

**Problem:** Hidden fields create confusion

❌ **Bad - Conditional Display:**
```typescript
{
  displayName: 'Data Source',
  name: 'source',
  type: 'options',
  options: [
    { name: 'From Credentials', value: 'credentials' },
    { name: 'Manual Input', value: 'manual' },
    { name: 'Both', value: 'both' }
  ],
  default: 'credentials'
},
{
  displayName: 'Select from Credentials',
  name: 'fromCredentials',
  type: 'multiOptions',
  displayOptions: {
    show: { source: ['credentials', 'both'] }
  }
},
{
  displayName: 'Manual Input',
  name: 'manualInput',
  type: 'string',
  displayOptions: {
    show: { source: ['manual', 'both'] }
  }
}
```

**Issues:**
- Users must choose before seeing options
- Unclear what each choice provides
- Extra click to see fields

✅ **Good - Always Visible:**
```typescript
{
  displayName: 'Select from Credentials',
  name: 'fromCredentials',
  type: 'multiOptions',
  typeOptions: { loadOptionsMethod: 'getItems' },
  default: [],
  description: 'Select items from saved credentials (optional)'
},
{
  displayName: 'Manual Input',
  name: 'manualInput',
  type: 'string',
  typeOptions: { rows: 3 },
  default: '',
  description: 'Enter items manually (optional)'
}
```

**Benefits:**
- All options visible immediately
- Users understand both are optional
- Can use one, both, or neither
- Fewer clicks, clearer intent

**Real Example:** ServerChan v2.0 removed source dropdown, reduced confusion by 80%

---

### 2. Multi-Delimiter Support ⭐⭐

**Always support multiple delimiters for list inputs**

```typescript
// In execute method
const items = input
  .split(/[\n,; ]+/)  // Newline, comma, semicolon, space
  .map(x => x.trim())
  .filter(x => x.length > 0);
```

**Why:**
- Users copy/paste from different sources
- CSV, space-separated, line-separated all work
- Zero implementation cost
- Huge UX improvement

**Update field description:**
```typescript
{
  displayName: 'Items',
  name: 'items',
  type: 'string',
  typeOptions: { rows: 3 },
  placeholder: 'item1\nitem2\nitem3',
  description: 'Items separated by newline, comma, semicolon, or space'
}
```

---

### 3. Notice Fields for Guidance ⭐⭐

**Use `type: 'notice'` to explain non-obvious patterns**

```typescript
{
  displayName: 'This node prepares data for the Send node. Use both nodes together in your workflow.',
  name: 'notice',
  type: 'notice',
  default: ''
}
```

**When to use Notice:**
- Multi-node workflows (Config + Action)
- First-time setup instructions
- Limitations or warnings
- Complex parameter interactions

**Example - Config + Action Pattern:**
```typescript
{
  displayName: 'Use this node with [ServiceName] Send to complete the workflow:\n[Config Node] → [Send Node]',
  name: 'workflowNotice',
  type: 'notice',
  default: ''
}
```

**Example - API Limitations:**
```typescript
{
  displayName: 'Note: This API has rate limits of 100 requests/minute',
  name: 'rateLimitNotice',
  type: 'notice',
  default: ''
}
```

---

### 4. Dynamic Subtitles ⭐

**Show real-time status in node subtitle**

❌ **Static - Useless:**
```typescript
subtitle: 'Manage Recipients'
```

✅ **Dynamic - Helpful:**
```typescript
// Simple count
subtitle: '={{$parameter["items"].length}} items selected'

// Conditional
subtitle: '={{$parameter["items"].length > 0 ? $parameter["items"].length + " items" : "No items"}}'

// Multiple sources
subtitle: '={{($parameter["selected"].length || 0) + " from credentials" + ($parameter["manual"] ? " + manual" : "")}}'
```

**Keep it simple:**
- Avoid deeply nested ternaries
- Fall back to static text if complex
- Test with actual data

**Real Example:**
```typescript
// ServerChan Key shows: "3 from credentials + manual keys"
subtitle: '={{($parameter["selectedRecipients"]?.length || 0) > 0 ? $parameter["selectedRecipients"].length + " from credentials" : ""}}{{$parameter["manualSendKeys"] ? " + manual keys" : ""}}'
```

---

### 5. Remove Unnecessary Fields ⭐⭐⭐

**Question every field - if it has only one option, remove it**

❌ **Bad - Operation with 1 Choice:**
```typescript
{
  displayName: 'Operation',
  name: 'operation',
  type: 'options',
  options: [
    {
      name: 'Send Message',
      value: 'send',
      routing: { request: { method: 'POST', url: '/send' } }
    }
  ],
  default: 'send'
}
```

**Problem:** Dropdown is pointless, adds clutter

✅ **Good - Routing on Last Parameter:**
```typescript
{
  displayName: 'Tags',
  name: 'tags',
  type: 'string',
  default: '',
  routing: {
    request: {
      method: 'POST',
      url: '/send',
      body: {
        title: '={{$parameter.title}}',
        content: '={{$parameter.content}}',
        tags: '={{$parameter.tags}}'
      }
    }
  }
}
```

**Result:** Cleaner UI, one less field to configure

---

### 6. Credential Field Naming

**Be specific about what the credential contains**

❌ **Vague:**
```typescript
{
  displayName: 'Configuration',
  name: 'config',
  type: 'json'
}
```

✅ **Clear:**
```typescript
{
  displayName: 'Recipients (JSON Array)',
  name: 'recipients',
  type: 'json',
  default: '[]',
  placeholder: '[{"name": "User1", "key": "..."}]'
}
```

**Best practices:**
- Specify expected format in displayName
- Use empty array/object as default (not examples)
- Put examples in placeholder
- Link to documentation

---

### 7. Error Messages in Load Options ⭐

**Provide helpful feedback when credential loading fails**

❌ **Silent Failure:**
```typescript
async getItems() {
  try {
    const items = await loadFromCredential();
    return items;
  } catch {
    return [];  // User sees nothing!
  }
}
```

✅ **Helpful Errors:**
```typescript
async getItems() {
  try {
    const credentials = await this.getCredentials('myApi');

    if (!credentials?.config) {
      return [{
        name: 'No configuration found - please set up credential',
        value: ''
      }];
    }

    const config = JSON.parse(credentials.config);

    if (!Array.isArray(config)) {
      return [{
        name: '⚠️ Invalid format - config must be an array',
        value: ''
      }];
    }

    if (config.length === 0) {
      return [{
        name: 'No items configured in credential',
        value: ''
      }];
    }

    return config.map(item => ({
      name: item.name,
      value: item.id
    }));

  } catch (error) {
    return [{
      name: '⚠️ Error loading - check credential JSON format',
      value: ''
    }];
  }
}
```

**Benefits:**
- Users understand what went wrong
- Clear next steps
- Reduces support requests

---

### 8. Parameter Order Matters

**Organize parameters logically**

**Good order:**
1. **Notice** (if needed) - Guidance first
2. **Required** fields - Most important
3. **Common optional** fields - Frequently used
4. **Advanced optional** fields - Rarely used
5. **Collection** (if needed) - Grouped advanced options

**Example:**
```typescript
properties: [
  // 1. Guidance
  { name: 'notice', type: 'notice', ... },

  // 2. Required
  { name: 'title', required: true, ... },
  { name: 'recipient', required: true, ... },

  // 3. Common optional
  { name: 'content', ... },
  { name: 'priority', ... },

  // 4. Advanced (collection)
  {
    name: 'additionalOptions',
    type: 'collection',
    options: [
      { name: 'retryCount', ... },
      { name: 'timeout', ... }
    ]
  }
]
```

---

### 9. Placeholder Quality

**Good placeholders teach correct usage**

❌ **Vague:**
```typescript
placeholder: 'Enter value'
```

✅ **Specific:**
```typescript
// Show format
placeholder: 'user@example.com'

// Show multiple lines
placeholder: 'item1\nitem2\nitem3'

// Show JSON structure
placeholder: '{"key": "value", "enabled": true}'

// Show delimiter options
placeholder: 'tag1, tag2, tag3'
```

---

### 10. Description Best Practices

**Descriptions should be concise and actionable**

**Format:**
- **What it does** - Brief explanation
- **Format/constraints** - If applicable
- **Link to docs** - For complex features

**Examples:**

❌ **Too long:**
```typescript
description: 'This field allows you to enter the title of the message that will be sent to the recipients. The title should be short and descriptive. Maximum length is 32 characters. Please ensure the title is clear and concise.'
```

✅ **Concise:**
```typescript
description: 'Message title (max 32 characters)'
```

**With link:**
```typescript
description: 'Markdown-formatted content. See <a href="https://example.com/docs" target="_blank">formatting guide</a>'
```

**With format:**
```typescript
description: 'Tags separated by | (e.g., urgent|notification|team)'
```

---

## Real-World Checklist

Before finalizing your node UI:

- [ ] All optional fields visible (no unnecessary displayOptions)
- [ ] Multi-delimiter support for list inputs
- [ ] Notice fields for multi-node workflows
- [ ] Dynamic subtitle (if useful)
- [ ] No single-option dropdowns
- [ ] Clear placeholder examples
- [ ] Concise descriptions with format hints
- [ ] Logical parameter order
- [ ] Helpful error messages in loadOptions
- [ ] Credential fields clearly named

---

## Anti-Patterns to Avoid

1. ❌ Hiding optional fields behind dropdown
2. ❌ Static subtitles on dynamic nodes
3. ❌ Accepting only newline delimiters
4. ❌ Silent failures in credential loading
5. ❌ Example data as default values
6. ❌ Overly complex subtitle expressions
7. ❌ Required fields at the bottom
8. ❌ Vague placeholders like "Enter value"

---

## Case Study: ServerChan Evolution

**v1.0 - Complex:**
- 3-option dropdown for source selection
- Hidden fields behind displayOptions
- Static subtitle
- Only newline delimiter
- Silent credential errors

**v2.0 - Simplified:**
- Flat structure, all fields visible
- Multi-delimiter support (newline, comma, semicolon, space)
- Dynamic subtitle showing count
- Helpful error messages
- Notice field explaining workflow

**Result:** 80% reduction in user confusion, faster configuration, better UX

---

## Template: Config + Action UI

**Config Node:**
```typescript
properties: [
  {
    displayName: 'Use this node with [Service] Send to complete the workflow.',
    name: 'notice',
    type: 'notice',
    default: ''
  },
  {
    displayName: 'Select from Credentials',
    name: 'selected',
    type: 'multiOptions',
    typeOptions: { loadOptionsMethod: 'getItems' },
    default: [],
    description: 'Select items from saved credentials (optional)'
  },
  {
    displayName: 'Manual Input',
    name: 'manual',
    type: 'string',
    typeOptions: { rows: 3 },
    default: '',
    placeholder: 'item1\nitem2, item3',
    description: 'Enter items manually, separated by newline/comma/semicolon/space (optional)'
  }
]

// Subtitle showing status
subtitle: '={{($parameter["selected"]?.length || 0) + " selected" + ($parameter["manual"] ? " + manual" : "")}}'
```

**Send Node:**
```typescript
properties: [
  {
    displayName: 'Target',
    name: 'target',
    type: 'string',
    default: '={{$json.target}}',  // From Config node
    required: true,
    description: 'Target from previous node (e.g., Config node)'
  },
  {
    displayName: 'Message',
    name: 'message',
    type: 'string',
    required: true,
    default: '',
    description: 'Message to send'
  },
  {
    displayName: 'Optional Field',
    name: 'optional',
    type: 'string',
    default: '',
    routing: {
      request: {
        method: 'POST',
        url: '=/send/{{$parameter.target}}',
        body: {
          message: '={{$parameter.message}}',
          optional: '={{$parameter.optional}}'
        }
      }
    }
  }
]

// Subtitle showing what will be sent
subtitle: '={{$parameter["message"]}}'
```

---

**Version:** 1.0.0 (based on ServerChan v2.0 learnings)
