# Troubleshooting n8n Node Development

Common issues and solutions based on real-world development experience.

## Build & Development Issues

### Issue 1: Icon Not Showing

**Symptom:** Node appears with default n8n icon instead of custom icon

**Causes:**
1. Missing `as const` type assertion
2. Icon file not copied to dist/
3. Wrong icon path

**Solutions:**

✅ **Correct Icon Declaration:**
```typescript
export class MyNode implements INodeType {
  description: INodeTypeDescription = {
    icon: 'file:icon.svg' as const,  // ⭐ 'as const' is required
    // NOT: icon: 'file:icon.svg' (TypeScript error)
  }
}
```

✅ **Ensure Icon Copying:**
```json
// package.json
{
  "scripts": {
    "build": "tsc && npm run copy:icons",
    "copy:icons": "cp nodes/*/icon.svg dist/nodes/*/"
  }
}
```

✅ **Verify Icon Location:**
```
nodes/MyNode/
├── MyNode.node.ts
└── icon.svg           # ← Must be here

dist/nodes/MyNode/     # After build
├── MyNode.node.js
└── icon.svg           # ← Must be copied here
```

---

### Issue 2: Node Not Appearing After Changes

**Symptom:** Changes to node code don't appear in n8n UI

**Cause:** n8n caches node definitions

**Solution (Complete Restart):**

```bash
# 1. Stop n8n completely (Ctrl+C)

# 2. Rebuild the node
npm run build

# 3. Restart n8n
npm run dev  # or restart your n8n instance

# 4. Hard refresh browser (Cmd+Shift+R / Ctrl+F5)

# 5. If still not working, clear browser cache completely
```

**For Development:**
Use watch mode to auto-rebuild:
```bash
npm run dev  # Watches for changes and auto-rebuilds
```

---

### Issue 3: Credential Not Found

**Symptom:** "Credential type 'myApi' not found" error

**Causes:**
1. Wrong path in package.json
2. Credential file not built
3. Typo in credential name

**Solutions:**

✅ **Check package.json paths:**
```json
{
  "n8n": {
    "credentials": [
      "dist/credentials/MyApi.credentials.js"  // ⭐ Must be .js, not .ts
    ],
    "nodes": [
      "dist/nodes/MyNode/MyNode.node.js"
    ]
  }
}
```

✅ **Verify credential name matches:**
```typescript
// credentials/MyApi.credentials.ts
export class MyApi implements ICredentialType {
  name = 'myApi';  // ⭐ Must match exactly
}

// nodes/MyNode/MyNode.node.ts
credentials: [
  {
    name: 'myApi',  // ⭐ Must match credential name
    required: true
  }
]
```

✅ **Check file exists:**
```bash
ls dist/credentials/MyApi.credentials.js  # Should exist after build
```

---

### Issue 4: TypeScript Build Errors

**Symptom:** `tsc` fails with type errors

**Common Causes & Fixes:**

**Missing Type Imports:**
```typescript
// ❌ Error: Cannot find name 'INodeType'
export class MyNode implements INodeType { }

// ✅ Fix: Import the type
import type { INodeType, INodeTypeDescription } from 'n8n-workflow';
```

**Icon Type Error:**
```typescript
// ❌ Type 'string' is not assignable to type 'Icon'
icon: 'file:icon.svg'

// ✅ Use type assertion
icon: 'file:icon.svg' as const
```

**Missing Required Fields:**
```typescript
// ❌ Property 'description' is missing
export class MyNode implements INodeType { }

// ✅ Add all required fields
export class MyNode implements INodeType {
  description: INodeTypeDescription = {
    displayName: 'My Node',
    name: 'myNode',
    group: ['transform'],
    version: 1,
    description: 'My node description',
    defaults: { name: 'My Node' },
    inputs: [NodeConnectionTypes.Main],
    outputs: [NodeConnectionTypes.Main],
    properties: []
  };
}
```

---

## Runtime Issues

### Issue 5: Subtitle Not Updating

**Symptom:** Dynamic subtitle doesn't change or shows error

**Causes:**
1. Expression syntax error
2. Accessing undefined properties
3. Expression too complex

**Solutions:**

❌ **Bad - No Safety Checks:**
```typescript
subtitle: '={{$parameter["items"].length}} items'
// Error if items is undefined!
```

✅ **Good - Safe Access:**
```typescript
subtitle: '={{($parameter["items"]?.length || 0)}} items'
```

❌ **Bad - Too Complex:**
```typescript
subtitle: '={{$parameter["source"] === "credentials" ? ($parameter["selected"] && $parameter["selected"].length > 0 ? $parameter["selected"].length + " from creds" : "None from creds") : ($parameter["manual"] ? "Manual input" : "No input")}}'
// Unreadable!
```

✅ **Good - Simplified:**
```typescript
subtitle: '={{($parameter["selected"]?.length || 0)}} selected'
// OR use static text if too complex
subtitle: 'Configure Recipients'
```

---

### Issue 6: Credential Validation Fails

**Symptom:** Test button fails with error

**Causes:**
1. Wrong test URL
2. Missing authentication headers
3. API endpoint changed

**Debug Steps:**

```typescript
test: ICredentialTestRequest = {
  request: {
    method: 'GET',
    baseURL: 'https://api.example.com',  // ⭐ Check this URL
    url: '/validate',                     // ⭐ And this endpoint
    headers: {
      'Authorization': '=Bearer {{$credentials.apiKey}}'  // ⭐ Correct header format
    }
  }
}
```

**Common Fixes:**

✅ **Use correct expression syntax:**
```typescript
// ❌ Wrong
url: 'https://api.example.com/{{$credentials.key}}'

// ✅ Right
baseURL: 'https://api.example.com',
url: '=/{{$credentials.key}}'  // ⭐ Note the =/ prefix
```

✅ **Test manually first:**
```bash
# Test the API endpoint manually
curl -X GET https://api.example.com/validate \
  -H "Authorization: Bearer YOUR_KEY"

# If this fails, fix the test configuration
```

---

### Issue 7: Operation Not Executing

**Symptom:** Node execution completes but nothing happens

**Causes (Declarative Mode):**
1. Missing `routing` configuration
2. Wrong HTTP method
3. Incorrect parameter mapping

**Solutions:**

✅ **Verify routing exists:**
```typescript
{
  displayName: 'Message',
  name: 'message',
  type: 'string',
  routing: {  // ⭐ Must have routing!
    request: {
      method: 'POST',
      url: '/send',
      body: {
        message: '={{$parameter.message}}'
      }
    }
  }
}
```

✅ **Check HTTP method matches API:**
```typescript
// If API expects POST
routing: {
  request: {
    method: 'POST',  // ⭐ Not GET
    url: '/create'
  }
}
```

✅ **Verify parameter names match:**
```typescript
// Parameter name
{ name: 'title', ... }

// Must match in routing
body: {
  title: '={{$parameter.title}}'  // ⭐ Same name
}
```

---

### Issue 8: Items Not Looping

**Symptom:** Node only processes first item, ignores others

**Cause:** Not using n8n's item loop in Programmatic mode

**Solution:**

❌ **Wrong - Processes only first item:**
```typescript
async execute(this: IExecuteFunctions) {
  const item = this.getInputData()[0];  // ❌ Only first item!
  const result = await processItem(item);
  return [[{ json: result }]];
}
```

✅ **Right - Loops all items:**
```typescript
async execute(this: IExecuteFunctions) {
  const items = this.getInputData();
  const returnData: INodeExecutionData[] = [];

  for (let i = 0; i < items.length; i++) {  // ⭐ Loop all items
    const item = items[i];
    const result = await processItem(item);

    returnData.push({
      json: result,
      pairedItem: { item: i }  // ⭐ Link to input item
    });
  }

  return [returnData];
}
```

---

## Credential Issues

### Issue 9: JSON Parsing Errors

**Symptom:** "Unexpected token" or "Invalid JSON" errors

**Cause:** Invalid JSON in credential field

**Solutions:**

✅ **Provide clear error handling:**
```typescript
async loadOptions() {
  try {
    const credentials = await this.getCredentials('myApi');
    const config = JSON.parse(credentials.config as string);

    if (!Array.isArray(config)) {
      return [{
        name: '⚠️ Config must be an array',
        value: ''
      }];
    }

    return config.map(item => ({
      name: item.name,
      value: item.id
    }));

  } catch (error) {
    return [{
      name: '⚠️ Invalid JSON format - check credential',
      value: ''
    }];
  }
}
```

✅ **Use placeholder for example:**
```typescript
{
  displayName: 'Config (JSON Array)',
  name: 'config',
  type: 'json',
  default: '[]',  // ⭐ Empty array, not example
  placeholder: '[{"name": "Item1", "id": "123"}]',  // ⭐ Example here
  description: 'JSON array of items'
}
```

---

### Issue 10: fixedCollection in Credentials

**Symptom:** fixedCollection doesn't work in credential definition

**Cause:** Known n8n limitation (Issue #6693)

**Workaround - Use JSON Instead:**

❌ **Doesn't Work:**
```typescript
{
  displayName: 'Items',
  name: 'items',
  type: 'fixedCollection',  // ❌ Not supported in credentials
  typeOptions: {
    multipleValues: true
  },
  options: [...]
}
```

✅ **Use JSON:**
```typescript
{
  displayName: 'Items (JSON Array)',
  name: 'items',
  type: 'json',  // ✅ Works in credentials
  default: '[]',
  placeholder: '[{"name": "Item1", "value": "val1"}]',
  description: 'Array of items in JSON format'
}
```

---

## Publishing Issues

### Issue 11: Package Too Large

**Symptom:** npm publish fails or package > 1MB

**Causes:**
1. Including node_modules
2. Including source files
3. Including docs/images

**Solutions:**

✅ **Configure files array in package.json:**
```json
{
  "files": [
    "dist",
    "credentials"
  ]
  // ⭐ Only include what's needed
  // Excludes: src/, docs/, tests/, node_modules/
}
```

✅ **Create .npmignore:**
```
# .npmignore
src/
docs/
tests/
*.log
.DS_Store
tsconfig.json
```

✅ **Check package size before publishing:**
```bash
npm pack --dry-run
# Shows what will be included and total size
```

---

### Issue 12: Dependency Conflicts

**Symptom:** Users report installation errors

**Cause:** Conflicting peer dependencies

**Solutions:**

✅ **Use correct dependency types:**
```json
{
  "dependencies": {
    // ⚠️ Avoid if possible (increases package size)
  },
  "devDependencies": {
    "n8n-workflow": "^1.x",  // ⭐ Dev only, not bundled
    "typescript": "^5.x"
  },
  "peerDependencies": {
    // Only if truly required
  }
}
```

✅ **Prefer zero dependencies:**
- Use n8n's built-in routing (Declarative mode)
- Use native JavaScript instead of libraries
- Only add dependencies for complex auth (OAuth, crypto)

---

## Debug Checklist

When something doesn't work:

1. **Build Issues:**
   - [ ] Run `npm run build` successfully
   - [ ] Check `dist/` folder contains all files
   - [ ] Icons copied to dist/

2. **Code Issues:**
   - [ ] All imports present
   - [ ] TypeScript compiles without errors
   - [ ] Correct type assertions (`as const`)

3. **Configuration:**
   - [ ] package.json paths correct (.js not .ts)
   - [ ] Credential names match exactly
   - [ ] Icon paths correct

4. **Runtime:**
   - [ ] n8n completely restarted
   - [ ] Browser cache cleared
   - [ ] Check n8n console for errors

5. **Execution:**
   - [ ] Routing configuration present (Declarative)
   - [ ] Item looping implemented (Programmatic)
   - [ ] Error handling with continueOnFail

---

## Getting Help

If issues persist:

1. **Check n8n logs:**
   ```bash
   # Look for error messages
   n8n start --log-level debug
   ```

2. **Test in isolation:**
   - Create minimal test workflow
   - Use only your node
   - Test with simple data

3. **Compare with working examples:**
   - Check official n8n nodes
   - Look at community nodes
   - Use n8n-nodes-starter template

4. **Community resources:**
   - [n8n Community Forum](https://community.n8n.io/)
   - [GitHub Discussions](https://github.com/n8n-io/n8n/discussions)
   - [Official Documentation](https://docs.n8n.io/integrations/creating-nodes/)

---

**Version:** 1.0.0
**Last Updated:** 2025-10-21
