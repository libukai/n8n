# Zero Dependencies Strategy

Guide to building n8n nodes without runtime dependencies for optimal performance and maintainability.

## Why Zero Dependencies?

### 1. n8n Verified Nodes Requirement ⭐⭐⭐

**Official requirement:** n8n verified nodes CANNOT have runtime dependencies

**Source:** [n8n Node Development Guidelines](https://docs.n8n.io/integrations/creating-nodes/)

If you want your node to be eligible for verification:
- ✅ `"dependencies": {}`
- ✅ Only `devDependencies` allowed

### 2. Performance Benefits

| Metric | With SDK | Zero Deps | Improvement |
|--------|----------|-----------|-------------|
| Package size | ~500KB | ~50KB | **90% smaller** |
| Install time | ~15s | ~2s | **86% faster** |
| Security surface | High | Minimal | **Low risk** |
| Maintenance | Ongoing | Minimal | **Less work** |

**Real Example: ServerChan Node**
- Before (with serverchan-sdk): 500KB
- After (Declarative mode): 50KB
- Result: 10x smaller, same functionality

### 3. Maintenance Advantages

**With Dependencies:**
- ❌ SDK breaking changes affect your node
- ❌ Security vulnerabilities in dependencies
- ❌ Dependency conflicts with other nodes
- ❌ Regular updates required

**Zero Dependencies:**
- ✅ No breaking changes from third parties
- ✅ No security vulnerabilities to patch
- ✅ No conflicts possible
- ✅ Set and forget

---

## When You Can Avoid Dependencies

### Scenario 1: HTTP APIs ⭐⭐⭐

**✅ Use n8n's Declarative Routing (Zero Dependencies)**

```typescript
// NO SDK NEEDED!
{
  displayName: 'Message',
  name: 'message',
  type: 'string',
  routing: {
    request: {
      method: 'POST',
      url: '=/send/{{$parameter.recipient}}',
      body: {
        message: '={{$parameter.message}}'
      }
    }
  }
}
```

**Benefits:**
- Built into n8n
- Zero code to maintain
- Handles authentication automatically
- Supports expressions

**Covers 80% of HTTP APIs:**
- RESTful CRUD operations
- Simple authentication (API key, Basic Auth)
- JSON/form data requests
- Standard HTTP methods

---

### Scenario 2: Data Processing

**✅ Use Native JavaScript**

Instead of libraries like lodash, use native JavaScript:

```typescript
// ❌ With dependency
import _ from 'lodash';
const unique = _.uniq(items);

// ✅ Native JavaScript
const unique = [...new Set(items)];
```

```typescript
// ❌ With dependency
import moment from 'moment';
const date = moment().format('YYYY-MM-DD');

// ✅ Native JavaScript
const date = new Date().toISOString().split('T')[0];
```

```typescript
// ❌ With dependency
import validator from 'validator';
const isEmail = validator.isEmail(input);

// ✅ Native JavaScript
const isEmail = /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(input);
```

**Common tasks without dependencies:**
- Array operations: `map`, `filter`, `reduce`, `Set`
- String manipulation: `split`, `trim`, `replace`, regex
- JSON: `JSON.parse`, `JSON.stringify`
- Date/time: `Date` object, `Intl` API
- Validation: Regular expressions

---

### Scenario 3: Simple Authentication

**✅ Use n8n's Built-in Auth**

n8n supports:
- API Key (header, query param)
- Basic Auth
- OAuth1
- OAuth2
- Custom headers

```typescript
// credentials/MyApi.credentials.ts
export class MyApi implements ICredentialType {
  name = 'myApi';
  displayName = 'My API';
  properties: INodeProperties[] = [
    {
      displayName: 'API Key',
      name: 'apiKey',
      type: 'string',
      typeOptions: { password: true },
      default: ''
    }
  ];

  authenticate: IAuthenticateGeneric = {
    type: 'generic',
    properties: {
      headers: {
        'Authorization': '=Bearer {{$credentials.apiKey}}'
      }
    }
  };
}
```

**No SDK needed for:**
- Token-based auth
- API key auth
- Basic authentication
- Custom headers

---

## When You NEED Dependencies

### Use Dependencies If:

#### 1. Complex Cryptographic Operations

```typescript
// Signing requests (AWS, OAuth1)
"dependencies": {
  "crypto-js": "^4.0.0"  // OK - Crypto is hard
}
```

**Example:** AWS signature v4, HMAC-SHA256

#### 2. Official SDK with OAuth Flow

```typescript
// OAuth with refresh tokens, PKCE, etc.
"dependencies": {
  "google-auth-library": "^8.0.0"  // OK - Complex OAuth
}
```

**Example:** Google OAuth, Microsoft Graph

#### 3. Binary Data Processing

```typescript
// Image manipulation, PDF generation
"dependencies": {
  "sharp": "^0.31.0",  // OK - Binary processing
  "pdfkit": "^0.13.0"
}
```

**Example:** Image resizing, PDF creation

#### 4. WebSocket/Streaming APIs

```typescript
// Real-time connections
"dependencies": {
  "ws": "^8.0.0"  // OK - WebSocket not in Declarative
}
```

**Example:** Chat applications, live updates

---

## Migration Guide: SDK → Zero Dependencies

### Step 1: Analyze SDK Usage

```typescript
// With SDK
import ServiceSDK from 'service-sdk';

async execute() {
  const sdk = new ServiceSDK(apiKey);
  const result = await sdk.sendMessage(params);
  return result;
}
```

**Questions:**
1. What HTTP requests does the SDK make?
2. What authentication does it use?
3. What data transformations does it do?

### Step 2: Replace with Declarative Routing

```typescript
// Without SDK - Declarative
{
  displayName: 'Message',
  name: 'message',
  type: 'string',
  routing: {
    request: {
      method: 'POST',
      url: '/api/send',
      body: {
        message: '={{$parameter.message}}',
        recipient: '={{$parameter.recipient}}'
      },
      headers: {
        'Authorization': '=Bearer {{$credentials.apiKey}}'
      }
    }
  }
}
```

### Step 3: Handle Authentication

```typescript
// credentials/ServiceApi.credentials.ts
authenticate: IAuthenticateGeneric = {
  type: 'generic',
  properties: {
    headers: {
      'Authorization': '=Bearer {{$credentials.apiKey}}'
    }
  }
}
```

### Step 4: Test Thoroughly

- [ ] All operations work
- [ ] Authentication successful
- [ ] Error handling correct
- [ ] Package size reduced

---

## Real-World Examples

### Example 1: ServerChan

**Before (v1.0 with SDK):**
```json
{
  "dependencies": {
    "serverchan-sdk": "^1.0.6"
  }
}
```
- Package size: 500KB
- Installation: 15s
- Maintenance: Regular SDK updates needed

**After (v2.0 Declarative):**
```json
{
  "dependencies": {}
}
```
- Package size: 50KB (90% reduction)
- Installation: 2s (86% faster)
- Maintenance: None

**Migration:**
```typescript
// Before - SDK
import ServiceChan from 'serverchan-sdk';
const sc = new ServiceChan(sendKey);
await sc.send(title, content);

// After - Declarative
routing: {
  request: {
    method: 'POST',
    url: '=/{{$parameter.sendKey}}.send',
    body: {
      title: '={{$parameter.title}}',
      desp: '={{$parameter.content}}'
    }
  }
}
```

---

### Example 2: Weather API

**Avoid:**
```json
{
  "dependencies": {
    "weather-api-client": "^2.0.0"
  }
}
```

**Use Declarative:**
```typescript
{
  displayName: 'City',
  name: 'city',
  type: 'string',
  routing: {
    request: {
      method: 'GET',
      url: '=/weather/{{$parameter.city}}',
      qs: {
        units: 'metric',
        appid: '={{$credentials.apiKey}}'
      }
    }
  }
}
```

**Result:**
- Zero dependencies
- Same functionality
- Faster installation
- Less maintenance

---

### Example 3: GitHub API

**Avoid:**
```json
{
  "dependencies": {
    "@octokit/rest": "^19.0.0"
  }
}
```

**Use Declarative:**
```typescript
{
  displayName: 'Operation',
  name: 'operation',
  type: 'options',
  options: [
    {
      name: 'Get Issue',
      value: 'getIssue',
      routing: {
        request: {
          method: 'GET',
          url: '=/repos/{{$parameter.owner}}/{{$parameter.repo}}/issues/{{$parameter.issueNumber}}'
        }
      }
    },
    {
      name: 'Create Issue',
      value: 'createIssue',
      routing: {
        request: {
          method: 'POST',
          url: '=/repos/{{$parameter.owner}}/{{$parameter.repo}}/issues',
          body: {
            title: '={{$parameter.title}}',
            body: '={{$parameter.body}}'
          }
        }
      }
    }
  ]
}
```

---

## Decision Tree

```
Do you need to integrate with an HTTP API?
├─ YES → Can you use Declarative routing?
│   ├─ YES → ✅ Use Zero Dependencies (Declarative)
│   └─ NO → Why not?
│       ├─ Complex auth (OAuth flow) → ⚠️ Use OAuth library
│       ├─ Binary data → ⚠️ Use processing library
│       ├─ WebSockets → ⚠️ Use ws library
│       └─ Other → 🤔 Reconsider, probably can avoid
└─ NO → Are you processing data?
    ├─ YES → Can you use native JavaScript?
    │   ├─ YES → ✅ Use Zero Dependencies (Native JS)
    │   └─ NO → ⚠️ Use specialized library (crypto, image processing)
    └─ NO → What are you doing?
        └─ 🤔 Reconsider approach
```

---

## Checklist

Before adding a dependency:

- [ ] Checked if Declarative routing can handle it
- [ ] Checked if native JavaScript can do it
- [ ] Checked if n8n provides built-in support
- [ ] Verified dependency is absolutely necessary
- [ ] Considered maintenance burden
- [ ] Considered package size impact
- [ ] Checked if it disqualifies node from verification

**If ALL checks pass, only then add dependency.**

---

## Best Practices

### 1. Start with Zero

Always start with zero dependencies. Only add when truly needed.

### 2. Use DevDependencies

Tools are OK in devDependencies:
```json
{
  "devDependencies": {
    "n8n-workflow": "^1.x",    // OK
    "typescript": "^5.x",       // OK
    "@types/node": "^18.x"      // OK
  }
}
```

### 3. Document Why

If you must add a runtime dependency, document why:
```json
{
  "dependencies": {
    // Required for OAuth2 PKCE flow - no Declarative alternative
    "oauth-library": "^1.0.0"
  }
}
```

### 4. Regular Audits

Periodically check if dependencies are still needed:
```bash
# Check dependency tree
npm ls

# Check for alternatives
npm outdated

# Analyze bundle size
npm pack --dry-run
```

---

## Performance Comparison

| Dependency Count | Package Size | Install Time | Security Risk |
|------------------|--------------|--------------|---------------|
| 0 (Zero deps) | 50KB | 2s | Minimal |
| 1-3 (Few deps) | 200KB | 5s | Low |
| 4-10 (Some deps) | 500KB | 10s | Medium |
| 10+ (Many deps) | 1MB+ | 20s+ | High |

**Recommendation:** Aim for 0, accept 1-3 if necessary, avoid 4+

---

## Summary

**Default to zero dependencies. Add only when absolutely necessary.**

**90% of n8n nodes can be zero-dependency using:**
- Declarative routing (HTTP APIs)
- Native JavaScript (data processing)
- Built-in n8n auth (API keys, Basic Auth)

**Add dependencies only for:**
- Complex cryptography (AWS signatures, etc.)
- OAuth flows (with refresh, PKCE)
- Binary processing (images, PDFs)
- WebSockets/streaming

**Benefits:**
- ✅ Smaller package size (90% reduction)
- ✅ Faster installation (86% faster)
- ✅ Less maintenance
- ✅ No security vulnerabilities
- ✅ Eligible for n8n verification

---

**Version:** 1.0.0
**Based on:** ServerChan v2.0 migration experience
