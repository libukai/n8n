---
name: n8n-nodes-creator
description: Create n8n community nodes (Node.js/TypeScript backend, not Vue frontend). Provides architecture pattern selection (Declarative, Programmatic, Config+Action, Dynamic Modular), JSON-based UI configuration guidance, credential testing patterns, and migration to @n8n/node-cli build system.
---

# n8n Nodes Creator

Create n8n community nodes (backend execution engine) following official best practices with clear architecture guidance.

## ⚠️ Important: What This Skill Covers

**This skill is for building n8n Community Nodes - the backend execution logic, NOT Vue frontend development.**

### Understanding n8n's Architecture

```
┌─────────────────────────────────────────┐
│  n8n Frontend (Vue 3 + TypeScript)      │  ← NOT covered by this skill
│  - User interface, workflow canvas      │     (See n8n source code for this)
│  - Automatically renders node UI        │
└─────────────────┬───────────────────────┘
                  │
                  │ Reads node definitions
                  ↓
┌─────────────────────────────────────────┐
│  n8n Nodes (Node.js + TypeScript)       │  ← THIS SKILL COVERS THIS
│  - Backend execution logic               │
│  - JSON-based UI configuration          │
│  - Data processing and API calls        │
└─────────────────────────────────────────┘
```

### What This Skill Covers

✅ **Node.js + TypeScript development** - Write backend execution logic
✅ **JSON-based UI configuration** - Define node parameters via `properties` array
✅ **n8n execution patterns** - How nodes process data in workflows
✅ **API integration** - Connect to external services

❌ **NOT covered**: Writing Vue components (n8n's Vue frontend auto-renders JSON config)

### Relationship with Vue Development

- **n8n Community Nodes**: Backend only, UI via JSON config (this skill)
- **n8n Frontend Apps**: Separate Vue apps that call n8n webhooks (different skill/use case)
- **n8n Core UI**: Contributing to n8n's Vue frontend (requires n8n source code knowledge)

## Purpose

Create production-ready n8n community nodes with:
- Official @n8n/node-cli tooling
- Architecture pattern selection (Declarative, Programmatic, Config+Action, Dynamic Modular)
- JSON-based UI configuration that n8n renders automatically
- Clear decision tree for choosing the right approach
- Standards-compliant code structure

## When to Use This Skill

This skill should be used when:
- Creating a new n8n community node from scratch
- Integrating HTTP APIs with n8n
- Building batch operation nodes (notifications, data updates)
- Refactoring existing nodes to follow best practices
- Large-scale API integrations (50+ operations)
- Extending n8n's backend capabilities with custom logic

---

## 🔍 Core Development Principles

### 1. **Source Code is the Ultimate Truth**

When encountering problems or uncertainty about n8n behavior:

**DO:**
- ✅ **Search n8n source code first** - Use grep/ripgrep to find real examples
- ✅ **Copy proven patterns** - If it works in n8n's official nodes, it will work for you
- ✅ **Verify assumptions** - Check `packages/nodes-base` for actual usage patterns

**DON'T:**
- ❌ **Guess syntax** - n8n's expression syntax is specific and must be exact
- ❌ **Rely on documentation alone** - Docs may be outdated or incomplete
- ❌ **Assume standard JS/TS patterns work** - n8n has its own conventions

**Example workflow:**
```bash
# Problem: Unsure about subtitle syntax
# Solution: Search n8n source code
cd /path/to/n8n
grep -r "subtitle.*parameter" packages/nodes-base --include="*.ts" | head -10

# Result: Found 20+ real examples showing '={{...}}' syntax
# Outcome: Use proven pattern instead of guessing
```

**Common search patterns:**
- Subtitle syntax: `grep -r "subtitle.*parameter" packages/nodes-base --include="*.ts"`
- Routing config: `grep -r "routing.*request" packages/nodes-base --include="*.ts"`
- Credential test: `grep -r "test:.*ICredentialTestRequest" packages/nodes-base --include="*.ts"`
- DisplayOptions: `grep -r "displayOptions.*show" packages/nodes-base --include="*.ts"`

### 2. **Test in Real Environment**

Development checklist:
1. Build the node: `npm run build`
2. Link to n8n: `npm link` → `cd ~/.n8n/custom && npm link your-node`
3. **Restart n8n** (critical - changes won't load otherwise)
4. Hard refresh browser: `Cmd+Shift+R` / `Ctrl+Shift+R`
5. If still broken, **delete and re-add node** in workflow

**Common mistake:** Changing code but not restarting n8n, then assuming the code is wrong.

---

## Quick Start

### For New Projects

To create a new n8n community node:

1. **Initialize**: Create node in fixed directory
   ```bash
   cd ~/Github/Tools
   npm create @n8n/node my-integration
   ```
2. **Choose architecture**: Review Phase 1 decision tree
3. **Implement pattern**: Follow Phase 3 guidance for chosen pattern
4. **Build**: `npm run build`
5. **Test**: Load in local n8n instance (`npm run dev`)

**Note**: All n8n community nodes are generated in `~/Github/Tools` directory for consistency.

### For Migrating Existing Nodes

To upgrade an existing node to @n8n/node-cli:

1. **Review current setup**: Check Phase 2 "Migrating to @n8n/node-cli"
2. **Install CLI**: `npm install --save-dev @n8n/node-cli`
3. **Update scripts**: Replace `tsc` with `npx --yes @n8n/node-cli build`
4. **Create ESLint config**: Minimal `eslint.config.js` (see Phase 2)
5. **Test build**: `npm run build && npm run lint`

### For Understanding Patterns

Load specific reference files based on the pattern:

- **Declarative**: `references/DECLARATIVE.md`
- **Config + Action**: `references/CONFIG_ACTION_PATTERN.md`
- **UI Design**: `references/UI_DESIGN_PATTERNS.md`
- **Troubleshooting**: `references/TROUBLESHOOTING.md`

---

## Core Workflow

### Phase 1: Architecture Decision Tree

Answer these questions to choose the right architecture pattern:

#### Question 1: What's the project scope?

**Simple Single-Purpose** (1-3 operations)
- Example: Send notification, Get weather, Convert currency
- Pattern: → **Simple Programmatic**

**Standard API Integration** (5-30 operations, RESTful)
- Example: GitHub Issues, Slack messages, Database CRUD
- Pattern: → **Declarative**

**Large-Scale Integration** (50+ operations)
- Example: WeChat Official Account, Salesforce, Complex ERPs
- Pattern: → **Dynamic Modular**

#### Question 2: Does the node involve configuration + execution?

If your node handles BOTH:
- Configuration (manage recipients, prepare params, complex logic)
- Execution (send message, create record, API call)

→ Consider **Config + Action Pattern** (separation of concerns)

Example:
- ServerChan: Config (manage SendKeys) + Send (deliver message)
- Email Batch: Config (prepare recipients) + Send (deliver email)

#### Question 3: Check Declarative limitations

Declarative can ONLY handle:
- ✅ 1 item → 1 HTTP request
- ✅ Simple parameter mapping
- ✅ Standard RESTful CRUD

Declarative CANNOT handle:
- ❌ 1 item → N requests (batch operations)
- ❌ Loops, conditionals
- ❌ Using third-party SDK
- ❌ Complex pre/post processing

If you hit any ❌ → Use **Programmatic**

---

### Phase 2: Initialize Project

Use official n8n CLI in the fixed directory:

```bash
cd ~/Github/Tools
npm create @n8n/node my-integration
```

This generates:
- package.json with @n8n/node-cli
- Standard tsconfig.json
- Example nodes (Example + GithubIssues)
- Configured build scripts using @n8n/node-cli

**Important**:
- All n8n community nodes should be created in `~/Github/Tools` for consistency
- Do NOT manually create tsconfig.json or build scripts - @n8n/node-cli handles this

#### Migrating Existing Projects to @n8n/node-cli

If you have an existing node using custom `tsc` scripts:

**Old way (manual):**
```json
{
  "scripts": {
    "build": "tsc && cp icons dist/",
    "dev": "tsc --watch"
  }
}
```

**New way (@n8n/node-cli):**
```json
{
  "scripts": {
    "build": "npx --yes @n8n/node-cli build",
    "dev": "npx --yes @n8n/node-cli dev",
    "lint": "npx --yes @n8n/node-cli lint",
    "lint:fix": "npx --yes @n8n/node-cli lint --fix"
  },
  "devDependencies": {
    "@n8n/node-cli": "^0.13.0",
    "eslint": "^9.38.0",
    "@typescript-eslint/parser": "^8.46.2",
    "@typescript-eslint/eslint-plugin": "^8.46.2"
  }
}
```

**Benefits:**
- ✅ Automatic icon/SVG copying (no manual `cp` commands)
- ✅ Consistent build process across all community nodes
- ✅ Built-in linting support
- ✅ Hot reload during development (`npm run dev`)
- ✅ Future-proof (updates handled by @n8n/node-cli)

**Migration checklist:**
1. Install @n8n/node-cli: `npm install --save-dev @n8n/node-cli`
2. Replace build scripts with @n8n/node-cli commands
3. Remove manual icon copy scripts (handled automatically)
4. Create minimal `eslint.config.js` (see below)
5. Test build: `npm run build`
6. Test lint: `npm run lint`

**Minimal ESLint config (eslint.config.js):**
```javascript
// eslint.config.js
export default [
  {
    ignores: ['dist/**', 'node_modules/**'],
  },
];
```

---

### Phase 3: Choose Implementation Pattern

Based on your decision tree result, load the corresponding reference:

#### Pattern A: Simple Programmatic

**Use when**: Simple single-purpose nodes, using SDK, or custom logic

**Structure**:
```
nodes/MyNode/
├── MyNode.node.ts        # Main node (with execute method)
├── GenericFunctions.ts   # Helper functions
└── mynode.svg
```

**Key**: Write `async execute()` method manually

See: `references/SIMPLE_PROGRAMMATIC.md`

---

#### Pattern B: Declarative (Recommended for HTTP APIs)

**Use when**: Standard RESTful API, simple CRUD operations

**Structure**:
```
nodes/MyNode/
├── MyNode.node.ts        # Main node (with routing config)
├── resources/
│   ├── user/
│   │   ├── index.ts      # Operation definitions with routing
│   │   ├── get.ts        # Parameters for Get
│   │   └── create.ts     # Parameters for Create
│   └── message/
└── shared/
    └── descriptions.ts   # Shared parameters
```

**Key**: Use `routing.request` config, n8n handles execution

See: `references/DECLARATIVE.md`

---

#### Pattern C: Config + Action (Separation of Concerns) ⭐

**Use when**: Node needs both configuration AND execution

**Structure**:
```
nodes/
├── MyServiceConfig/      # Config node (Programmatic)
│   └── MyServiceConfig.node.ts
└── MyServiceSend/        # Action node (Declarative)
    └── MyServiceSend.node.ts
```

**Benefits**:
- Config node: Handle complex logic (Programmatic)
- Action node: Pure execution (Declarative, ~10 lines!)
- High reusability
- Easy to test independently

**Workflow**:
```
[MyService Config] → [MyService Send]
  Prepare data         Execute (auto-loops)
```

See: `references/CONFIG_ACTION_PATTERN.md`

---

#### Pattern D: Dynamic Modular (Large Projects)

**Use when**: 50+ operations, need zero-config extensibility

**Structure**:
```
nodes/MyNode/
├── MyNode.node.ts
├── resource/
│   ├── UserResource.ts
│   └── user/
│       ├── UserGetOperate.ts    # Auto-discovered!
│       ├── UserListOperate.ts
│       └── UserCreateOperate.ts
└── help/
    ├── builder/ResourceBuilder.ts
    └── utils/ModuleLoadUtils.ts
```

**Key**: Use glob pattern to auto-discover operations

**Benefits**:
- Add operation = create new file (zero config!)
- Unified abstraction layer
- Perfect for large-scale projects

See: `references/DYNAMIC_MODULAR.md`

---

### Phase 4: Implement Credentials

All patterns need credentials:

```typescript
// credentials/MyServiceApi.credentials.ts
export class MyServiceApi implements ICredentialType {
  name = 'myServiceApi';
  displayName = 'My Service API';
  properties: INodeProperties[] = [
    {
      displayName: 'API Key',
      name: 'apiKey',
      type: 'string',
      typeOptions: { password: true },
      default: '',
    },
  ];

  // Test credential
  test: ICredentialTestRequest = {
    request: {
      method: 'GET',
      baseURL: 'https://api.example.com',
      url: '/validate',
    },
  };
}
```

#### Advanced Credential Test Patterns

**For complex credentials (JSON arrays, dynamic URLs):**

```typescript
// Example: Testing credentials with JSON configuration
test: ICredentialTestRequest = {
  request: {
    // Use IIFE (Immediately Invoked Function Expression) for complex logic
    baseURL: '={{(() => {
      const recipients = JSON.parse($credentials.recipients);
      if (!Array.isArray(recipients) || recipients.length === 0) {
        throw new Error("No recipients configured");
      }
      const sendKey = recipients[0].sendKey;
      const match = sendKey.match(/^sctp(\\d+)t/);
      if (!match) {
        throw new Error("Invalid SendKey format");
      }
      return "https://" + match[1] + ".push.ft07.com";
    })()}}',
    url: '={{(() => {
      const recipients = JSON.parse($credentials.recipients);
      return "/send/" + recipients[0].sendKey + ".send";
    })()}}',
    method: 'POST',
    body: {
      title: 'n8n Credential Test',
      desp: 'Testing credentials from n8n. If you receive this, credentials are valid.',
    },
  },
  rules: [
    {
      type: 'responseSuccessBody',
      properties: {
        key: 'code',
        value: 0,
        message: 'Invalid credentials. Please check your configuration.',
      },
    },
  ],
};
```

**Key takeaways:**
- Use IIFE `={{(() => { ... })()}}` for complex URL/body construction
- Parse JSON credentials carefully with error handling
- Test with the FIRST item if credentials contain arrays
- Provide clear error messages in `rules`

---

### Phase 5: Design Node UI (JSON Configuration)

**⚠️ Important: Node developers write JSON configuration, not Vue components.**

The n8n frontend automatically renders UI based on the `properties` array in the node definition.

#### How UI Generation Works

```typescript
// In your node class (TypeScript)
description: INodeTypeDescription = {
  properties: [
    {
      displayName: 'Email',           // ← Label shown in UI
      name: 'email',                   // ← Field name
      type: 'string',                  // ← n8n renders text input
      default: '',
      placeholder: 'user@example.com', // ← Placeholder text
      description: 'Email address',    // ← Help text
    },
    {
      displayName: 'Priority',
      name: 'priority',
      type: 'options',                 // ← n8n renders dropdown
      options: [
        { name: 'High', value: 'high' },
        { name: 'Low', value: 'low' },
      ],
      default: 'low',
    },
  ],
}
```

**Result in n8n UI:**
- Vue frontend reads this JSON
- Automatically renders appropriate form controls
- `type: 'string'` → text input
- `type: 'options'` → dropdown
- `type: 'number'` → number input
- `type: 'json'` → JSON editor
- etc.

**Developers write JSON, n8n renders Vue components automatically!**

#### Supported Property Types

Common types that n8n can render:
- `string`, `number`, `boolean`
- `options`, `multiOptions` (dropdowns)
- `json` (JSON editor)
- `collection`, `fixedCollection` (nested fields)
- `resourceLocator` (dynamic resource picker)
- `credentialsSelect` (credential selector)
- `notice` (informational text, not a field)
- `color`, `dateTime`

See `references/UI_DESIGN_PATTERNS.md` for complete reference.

#### UI Design Best Practices

Follow UI design best practices from `references/UI_DESIGN_PATTERNS.md`:

**Key Principles:**
1. **Flat over nested** - Show all optional fields, don't hide them
2. **Multi-delimiter support** - Accept newline, comma, semicolon, space
3. **Notice fields** - Explain multi-node workflows and non-obvious patterns
4. **Dynamic subtitles** - Show real-time status (but keep expressions simple)
5. **English-first (with exceptions)** - See Internationalization Guidelines below for regional services
6. **Remove unnecessary fields** - No single-option dropdowns

#### Dynamic Subtitle Syntax

**⚠️ CRITICAL: Always check n8n source code for correct syntax!**

The subtitle expression syntax uses **double curly braces `={{...}}`** (NOT single quotes with `='...'`):

```typescript
// ✅ CORRECT - Double curly braces with string concatenation
subtitle: '={{$parameter["operation"] + ": " + $parameter["resource"]}}',
subtitle: '={{$parameter["topN"] + " keywords"}}',

// ❌ WRONG - Single quote expression syntax (doesn't work)
subtitle: '="Extract " + $parameter["topN"] + " keywords"',
subtitle: '=`Extract ${$parameter["topN"]} keywords`',

// ❌ WRONG - Mixed syntax (shows raw expression text)
subtitle: 'Extract {{$parameter["topN"]}} keywords',
```

**Examples from n8n source code:**
- Flow node: `subtitle: '={{$parameter["operation"] + ": " + $parameter["resource"]}}'`
- Stripe node: `subtitle: '={{$parameter["operation"] + ": " + $parameter["resource"]}}'`
- Ftp node: `subtitle: '={{$parameter["protocol"] + ": " + $parameter["operation"]}}'`

**Best Practices:**
- Use `={{...}}` syntax (double curly braces)
- Use string concatenation with `+` operator
- Keep expressions simple (avoid complex logic)
- **When in doubt, grep n8n source code**: `grep -r "subtitle.*parameter" packages/nodes-base --include="*.ts"`

**UI Checklist:**
- [ ] All optional fields visible (not hidden behind displayOptions)
- [ ] Clear placeholders showing expected format
- [ ] Concise descriptions with format hints
- [ ] Notice field for multi-node workflows (Config + Action)
- [ ] Dynamic subtitle (if useful and simple)
- [ ] Logical parameter order (required first, optional last)
- [ ] Correct `type` for each field (n8n will render appropriate control)
- [ ] Text follows internationalization guidelines (see below)

#### Internationalization Guidelines

**Default Rule: English-First**

For most nodes, all user-facing text should use English for international compatibility:
- ✅ `displayName`, `action` - English (used for search, AI integration, and discoverability)
- ✅ `description`, `placeholder` - English (helps international developers)
- ✅ Error messages - English (easier to search for solutions online)

**Exception: Regional Services**

For nodes integrating **region-specific services** where 95%+ users speak the local language (examples: WeChat, DingTalk, Line, KakaoTalk):

**Option A: Bilingual - Recommended for Public Packages** ⭐
```typescript
{
    displayName: 'User (用户)',  // English primary + Native
    name: 'user',
    value: 'user',
}

{
    name: 'Get User Info',  // English
    value: 'user:info',
    action: 'Get user basic information',  // English (for Action List search)
    description: 'Get user basic information (获取用户基本信息)',  // Bilingual
    options: [
        {
            displayName: 'OpenID',  // Technical terms stay as-is
            name: 'openid',
            description: 'User identifier for this Official Account (普通用户的标识)',  // Bilingual
        }
    ]
}
```

**Benefits:**
- ✅ Follows n8n standards (`name`, `action` in English)
- ✅ Action List searchable by English keywords
- ✅ AI tools can understand and use the node
- ✅ Chinese users see familiar terms in descriptions
- ✅ International developers can understand structure

**Option B: Pure Native Language - Use with Caution** ⚠️
```typescript
{
    name: '获取用户信息',  // Pure Chinese
    value: 'user:info',
    action: '获取用户基本信息',  // Pure Chinese (still required!)
}
```

**Acceptable for:**
- Internal/private nodes (not published to npm)
- Company-internal deployments
- Nodes guaranteed to never need international users

**Not recommended for:**
- Public npm packages
- Nodes that might be referenced in tutorials
- Use cases where AI tools need to interact with the node

**Key Requirements (both options):**
- ✅ `action` property is MANDATORY (even if in native language)
- ✅ `value` should always use English/technical terms (used in code)
- ✅ Technical terms (OpenID, AppID, API Key) should remain as-is

**Decision Factors:**
- Publishing to npm publicly? → Use Option A (Bilingual)
- Service only available in one country? → Option B acceptable
- Will non-native speakers ever use this? → Use English or Bilingual
- Private company use only? → Any language OK

**Load UI_DESIGN_PATTERNS.md for detailed guidance and templates**

---

### Phase 6: Build and Test

#### Local Development Testing (Recommended Method)

For testing your custom node in a real n8n installation:

**Step 1: Build your node**
```bash
cd ~/Github/Tools/your-custom-node
npm run build
```

**Step 2: Create symlink to n8n's custom directory**
```bash
# Method 1: Using npm link (Recommended)
cd ~/Github/Tools/your-custom-node
npm link

cd ~/.n8n/custom
npm link your-node-package-name

# Method 2: Direct npm install (Alternative)
cd ~/.n8n/custom
npm install ~/Github/Tools/your-custom-node
```

Both methods create a symlink in `~/.n8n/custom/node_modules/` pointing to your development directory.

**⚠️ CRITICAL: Avoid Duplicate Node Loading**

n8n scans **TWO** directories for custom nodes:
1. `~/.n8n/nodes/node_modules/` - Production nodes (installed via n8n UI or npm)
2. `~/.n8n/custom/node_modules/` - Development/custom nodes (linked manually)

**Common Problem:** If the same node exists in both locations, it will appear **twice** in n8n UI!

**Prevention:**
- ✅ **Development**: ONLY link to `~/.n8n/custom/`
- ✅ **Production**: ONLY install to `~/.n8n/nodes/` (via n8n UI or npm)
- ✅ **Never have the same node in both locations**

**Fix duplicates:**
```bash
# Check for duplicates
ls ~/.n8n/nodes/node_modules/ | grep "^n8n-nodes-"
ls ~/.n8n/custom/node_modules/ | grep "^n8n-nodes-"

# Remove from production if developing locally
rm -rf ~/.n8n/nodes/node_modules/n8n-nodes-YOUR-NODE

# Keep only in custom/ for development
# Restart n8n to see changes
```

**Step 3: Restart n8n**
```bash
# If running n8n via systemd, docker, or other service manager
# Restart the service

# If running n8n directly
n8n start
# Or if using pnpm in n8n monorepo
pnpm dev
```

**Step 4: Verify node is loaded**
- Open n8n UI (usually http://localhost:5678)
- Search for your node in the nodes panel
- Your custom node should appear

**Development workflow with symlinks:**
```bash
# 1. Make changes to your node source code
# 2. Rebuild
npm run build

# 3. Restart n8n (symlink automatically reflects new build)
# 4. Refresh browser to see changes
```

**Important notes:**
- ✅ Changes to source code require rebuild (`npm run build`)
- ✅ n8n automatically scans `~/.n8n/custom/node_modules/` on startup
- ✅ No need to reinstall after changes (symlink keeps connection)
- ✅ Always use `~/.n8n/custom/` for local development
- ❌ Do NOT use `pnpm link --global` (n8n doesn't read from global pnpm store)
- ❌ Do NOT install to `~/.n8n/nodes/` during development (causes duplicates)
- ❌ Do NOT add to n8n monorepo's `node_modules` (wrong location)

---

#### Quick Reference: Build and Lint

```bash
# Build for production
npm run build

# Lint check
npm run lint

# Auto-fix linting issues
npm run lint:fix
```

---

### Phase 7: Quality Checklist

Before publishing, verify the following:

**Structure**:
- [ ] Uses @n8n/node-cli (NOT custom tsc scripts)
- [ ] Follows chosen pattern consistently
- [ ] Icons in correct locations
- [ ] No hardcoded credentials

**Code Quality**:
- [ ] Passes `npm run lint`
- [ ] Error handling with continueOnFail support
- [ ] Uses `UnexpectedError`/`OperationalError`/`UserError` (NOT deprecated `ApplicationError`)
- [ ] TypeScript strict mode
- [ ] Proper type definitions (prefer `satisfies` over `as`, except in tests)
- [ ] Never use `any` type — use `unknown` instead
- [ ] Zero runtime dependencies (or justified if needed)
- [ ] Icon with `as const` type assertion
- [ ] Text follows internationalization guidelines (English-first, bilingual for regional services)

**Functionality**:
- [ ] Credential test works
- [ ] All operations execute correctly
- [ ] Handles edge cases (empty input, API errors)
- [ ] Works with n8n's automatic item looping

**UI/UX**:
- [ ] Optional fields visible (not hidden)
- [ ] Operation dropdown exists for ALL resources (even single-option)
- [ ] Each operation has `action` property for UI visibility
- [ ] Multi-delimiter support for list inputs
- [ ] Notice field for multi-node workflows
- [ ] Clear placeholders and descriptions
- [ ] Helpful error messages in loadOptions

**Documentation**:
- [ ] README with installation instructions
- [ ] Example workflows
- [ ] API documentation links
- [ ] Clear parameter descriptions

---

## Architecture Pattern Comparison

| Pattern | Code Amount | Complexity | Best For |
|---------|-------------|------------|----------|
| Simple Programmatic | ~200 lines | Low | Single-purpose nodes |
| Declarative | ~100 lines | Medium | RESTful APIs |
| Config + Action | ~150 lines | Medium | Batch operations |
| Dynamic Modular | ~300+ lines | High | Large integrations (50+ ops) |

---

## Common Pitfalls to Avoid

### Code & Architecture

1. ❌ **Don't use custom build scripts** → Use @n8n/node-cli
2. ❌ **Don't mix patterns** → Choose one and stick to it
3. ❌ **Don't add runtime dependencies without reason** → Aim for zero dependencies (see ZERO_DEPENDENCIES.md)
4. ❌ **Don't skip credential testing** → Always implement test request
5. ❌ **Don't forget continueOnFail** → Let users handle errors gracefully
6. ❌ **Don't hardcode baseURL** → Use credentials or node parameters
7. ❌ **Don't use `ApplicationError`** → It's deprecated. Use `UnexpectedError`, `OperationalError`, or `UserError` from `n8n-workflow`
8. ❌ **Don't use `any` type** → Use `unknown` and type guards instead

### UI/UX

7. ❌ **Don't hide optional fields** → Use flat structure, show all fields
8. ⚠️ **ALWAYS keep Operation dropdowns (even with single option)** → Required for action list visibility in n8n UI
9. ❌ **Don't accept only newline delimiters** → Support comma, semicolon, space too
10. ❌ **Don't use static subtitles on dynamic nodes** → Show real-time status
11. ❌ **Don't fail silently in loadOptions** → Return helpful error messages

### Technical

12. ❌ **Don't forget `as const` on icon** → Will cause TypeScript errors
13. ❌ **Don't use complex subtitle expressions** → Keep them simple or use static text
14. ❌ **Don't put example data in credential defaults** → Use empty array/object + placeholder

### Development & Deployment

15. ⚠️ **CRITICAL: Don't install the same node in both directories** → Causes duplicate nodes in n8n UI
    - n8n scans TWO directories: `~/.n8n/nodes/` (production) and `~/.n8n/custom/` (development)
    - Having the same node in both = **node appears twice** in UI
    - **Development**: ONLY use `~/.n8n/custom/` (symlink with `npm install /path/to/node`)
    - **Production**: ONLY use `~/.n8n/nodes/` (install via n8n UI or npm)
    - **Check for duplicates**: `ls ~/.n8n/{nodes,custom}/node_modules/ | grep "^n8n-nodes-"`
    - **Fix**: Remove from `~/.n8n/nodes/` during development, restart n8n

### Internationalization

16. 🌍 **Follow internationalization guidelines based on service type**

    **For global/international services:**
    - ❌ Don't use non-English text → Use English for all user-facing text
    - ✅ English for: displayName, action, description, placeholders, error messages

    **For region-specific services (WeChat, DingTalk, etc.):**
    - ⭐ **Recommended**: Bilingual (English primary + native in parentheses)
      - `displayName: 'User (用户)'` ✅
      - `action: 'Get user info'` (English required for search)
      - `description: 'Get user info (获取用户信息)'` ✅
    - ⚠️ **Acceptable for private use**: Pure native language
      - Must still include `action` property (even in native language)
      - Not recommended for public npm packages

    **Key rules for ALL nodes:**
    - ✅ `value` fields always use English/technical terms (used in code)
    - ✅ `action` property is MANDATORY for all operations
    - ✅ Technical terms (OpenID, API Key, etc.) keep original casing
    - ✅ Links to localized documentation are acceptable

    See Phase 5 "Internationalization Guidelines" for detailed guidance

---

## ⚠️ Critical: Operation Dropdown Requirement

### Why EVERY Resource Must Have an Operation Dropdown

**Even if a resource has only ONE operation, you MUST include the Operation dropdown.**

#### The Problem

If you omit the Operation dropdown for single-operation resources:

```typescript
// ❌ WRONG - Missing Operation dropdown
properties: [
  {
    displayName: 'Resource',
    name: 'resource',
    options: [{ name: 'Article', value: 'article' }],
  },
  // Directly showing fields without Operation dropdown
  {
    displayName: 'Column Slug',
    name: 'slug',
    displayOptions: { show: { resource: ['article'] } },
  },
]
```

**Result:**
- ❌ n8n UI cannot find `operation` parameter
- ❌ No `action` property available to read
- ❌ **Action list in nodes panel shows NOTHING for this resource**
- ❌ Users cannot discover or select this functionality

#### The Solution

```typescript
// ✅ CORRECT - Always include Operation dropdown
properties: [
  {
    displayName: 'Resource',
    name: 'resource',
    options: [{ name: 'Article', value: 'article' }],
  },
  {
    displayName: 'Operation',
    name: 'operation',  // n8n UI looks for this
    displayOptions: { show: { resource: ['article'] } },
    options: [
      {
        name: 'Get Articles',
        value: 'getArticles',
        action: 'Get articles from a column',  // This appears in action list
      },
    ],
    default: 'getArticles',
  },
  // Now show fields
  {
    displayName: 'Column Slug',
    name: 'slug',
    displayOptions: {
      show: {
        resource: ['article'],
        operation: ['getArticles'],  // More specific
      }
    },
  },
]
```

**Result:**
- ✅ n8n UI finds `operation` parameter
- ✅ Reads `action` property from options
- ✅ **Displays "Get articles from a column" in action list**
- ✅ Users can discover and select functionality

#### How n8n Action List Works

```
n8n UI Scans Node Definition
       ↓
Finds ALL parameters named 'operation'
       ↓
For each operation parameter:
  - Gets options array
  - Extracts 'action' property from each option
       ↓
Displays actions in nodes panel grouped by resource
```

#### Design Trade-off

**Option A: Keep Single-Option Dropdown** ✅ Recommended
- Pro: Action list displays correctly
- Pro: Users can discover functionality
- Pro: Consistent with n8n patterns
- Con: UI has "redundant" dropdown

**Option B: Remove Single-Option Dropdown** ❌ Not Recommended
- Pro: Cleaner UI (one less click)
- Con: **Action list empty → users can't find it**
- Con: Poor discoverability
- Con: Inconsistent user experience

#### Real-World Example

From n8n-nodes-jintiankansha optimization:

**Before (Broken):**
```typescript
// Article resource - no Operation dropdown
// Result: Action list shows only 4 Subscription actions
// Article and Search actions: INVISIBLE
```

**After (Fixed):**
```typescript
// Article Operations
{
  displayName: 'Operation',
  name: 'operation',
  options: [{
    name: 'Get Articles',
    value: 'getArticles',
    action: 'Get articles from a column',  // Now visible!
  }],
}

// Search Operations
{
  displayName: 'Operation',
  name: 'operation',
  options: [{
    name: 'Search WeChat',
    value: 'searchWeChat',
    action: 'Search for WeChat public accounts',  // Now visible!
  }],
}
```

**Result:** Action list shows all 6 actions (1 Article + 4 Subscription + 1 Search) ✅

#### Key Takeaway

**Discoverability > UI Cleanliness**

Even if it feels redundant, ALWAYS include the Operation dropdown. Users need to see what your node can do in the action list.

---

## Decision Summary

Use this quick reference:

```
Need batch operations + result aggregation?
→ Config + Action Pattern

Standard HTTP API (GET/POST/PUT/DELETE)?
→ Declarative

Using third-party SDK?
→ Simple Programmatic

50+ operations, frequent additions?
→ Dynamic Modular

Unsure?
→ Start with Declarative (can always refactor)
```

---

## Verified Success Cases

This skill has been validated on production projects:

**✅ n8n-nodes-serverchan v3.1.0** (Config + Action pattern)
- Credential test with complex JSON array configuration
- IIFE pattern for dynamic URL construction
- Simplified subtitle expressions
- TypeScript strict mode enabled
- Migrated to @n8n/node-cli build system
- Zero runtime dependencies

**✅ n8n-nodes-jintiankansha v1.1.0** (Dynamic Modular pattern)
- Auto-discovery architecture for extensibility
- Operation dropdown requirement (action list visibility)
- Multi-resource node design

All patterns and best practices documented in this skill have been tested and verified in real-world production nodes.

---

## Reference Materials

Load these for detailed guidance:

**Architecture Patterns:**
- **DECLARATIVE.md**: Complete routing configuration reference
  - Search: `routing.request` for declarative routing examples
  - Search: `routing.output` for response handling
- **CONFIG_ACTION_PATTERN.md**: Job separation pattern (⭐ Recommended for batch operations)
  - Search: `separation.*concerns` for design pattern examples
  - Search: `pairedItem` for data tracking patterns
- **UI_DESIGN_PATTERNS.md**: UI/UX patterns for better node interfaces (⭐ Read for all nodes)
  - Search: `displayOptions` for conditional field visibility
  - Search: `Multi-delimiter` for input parsing patterns
  - Search: `subtitle` for dynamic status display

**Best Practices:**
- **ZERO_DEPENDENCIES.md**: Strategy for building lightweight nodes
  - Search: `Built-in.*alternatives` for avoiding dependencies
  - Search: `axios.*fetch` for HTTP client choices

**Troubleshooting:**
- **TROUBLESHOOTING.md**: Common issues and solutions
  - Search: `icon.*not.*showing` for icon issues
  - Search: `credential.*test.*fail` for auth debugging

---

## Success Criteria

Your node is production-ready when:

1. ✅ Loads in n8n without errors
2. ✅ All operations work as expected
3. ✅ Passes `npm run lint` without issues
4. ✅ Credential test succeeds
5. ✅ Handles errors gracefully
6. ✅ README documents all features
7. ✅ Published to npm with `n8n-community-node-package` keyword

---

## Version

**Version**: 3.7.0
**Last Updated**: 2025-11-13
**Based on**:
- n8n-nodes-starter (official)
- ServerChan v3.1.0 optimization experience ⭐ (production validated)
- Jintiankansha v1.1.0 development experience
- WeChat Official Account Node audit experience
- n8n-nodes-jieba subtitle debugging experience ⭐
- Community best practices

**Changes in v3.7 (Source Code First + Subtitle Syntax):**
- 🔍 **NEW SECTION**: "Core Development Principles" - Emphasizes checking n8n source code first
- 📝 **SUBTITLE SYNTAX DOCUMENTED**: Added dedicated "Dynamic Subtitle Syntax" section with correct `={{...}}` syntax
- ✅ **REAL EXAMPLES**: Included grep commands to search n8n source code for proven patterns
- 🎯 **ANTI-PATTERNS**: Documented wrong subtitle syntaxes to avoid (`='...'`, `` `...` ``, mixed syntax)
- 🛠️ **TROUBLESHOOTING WORKFLOW**: Step-by-step guide on how to debug issues using source code
- 💡 **KEY LESSON**: "Don't guess - grep!" principle for all uncertain syntax questions

**Changes in v3.6 (Simplified Testing Method):**
- 🎯 **SINGLE METHOD**: Simplified to ONLY use `npm link` to `~/.n8n/custom/` for local testing
- ❌ **REMOVED**: Removed built-in dev server and n8n source code options (not production-like)
- 📍 **CLEAR FOCUS**: Development workflow now exclusively uses `~/.n8n/custom/node_modules/`
- 🔗 **TWO LINKING METHODS**: Documented both `npm link` and direct `npm install` approaches
- ✅ **PRODUCTION PARITY**: Ensures testing in real n8n installation environment

**Changes in v3.5 (Duplicate Node Prevention):**
- ⚠️ **CRITICAL ISSUE DOCUMENTED**: Added warning about duplicate node loading
  - n8n scans TWO directories: `~/.n8n/nodes/` (production) + `~/.n8n/custom/` (development)
  - Same node in both locations → appears TWICE in UI
- ✅ **PREVENTION GUIDE**: Clear instructions to avoid duplicate installations
- 🛠️ **FIX INSTRUCTIONS**: Step-by-step guide to resolve duplicates
- 🔍 **DETECTION COMMANDS**: Quick check for duplicate installations

**Changes in v3.4 (Internationalization Flexibility):**
- 🌍 **INTERNATIONALIZATION REVISED**: Changed from strict English-only to pragmatic approach
  - Added "Internationalization Guidelines" section in Phase 5
  - **Option A (Recommended)**: Bilingual - English primary with native in parentheses
  - **Option B (Private use)**: Pure native language acceptable for internal nodes
- 📝 **COMMON PITFALLS UPDATED**: Internationalization section now reflects nuanced guidance
- ✅ **QUALITY CHECKLIST UPDATED**: Changed from "All text in English" to flexible guideline
- 🎯 **PRACTICAL EXAMPLES**: Added code examples for both bilingual and native approaches

**Changes in v3.3 (Testing & Deployment Clarity):**
- 🧪 **PHASE 6 EXPANDED**: Added three clear testing options (Built-in dev server, Standalone n8n, Source code)
- 📍 **CORRECT LINKING**: Documented proper use of `~/.n8n/custom/node_modules/` (NOT global pnpm)
- ⚠️ **COMMON MISTAKES**: Explicitly warns against `pnpm link --global` (n8n doesn't read from there)
- 🔗 **SYMLINK WORKFLOW**: Clear step-by-step guide for development with symlinks
- 🎯 **USE CASE CLARITY**: Each testing option labeled with "Best for" to guide choice

**Changes in v3.2 (Skill Quality - skill-creator Compliance):**
- ✅ **FILE NAMING**: Renamed `skill.md` → `SKILL.md` (official standard)
- ✏️ **WRITING STYLE**: Removed second-person references, using imperative form
- 📝 **IMPROVED DESCRIPTION**: More specific, mentions Node.js/TypeScript backend focus
- 🚀 **QUICK START**: Added for new projects, migrations, and pattern understanding
- ✅ **VERIFIED CASES**: Added production validation section (ServerChan v3.1.0, Jintiankansha v1.1.0)
- 🔍 **GREP PATTERNS**: Added search patterns for all reference files
- 🌍 **ENGLISH ONLY**: Removed all Chinese text, enforced English-only requirement for internationalization
- 📦 **VALIDATED**: Passed `package_skill.py` validation

**Changes in v3.1 (Practical Refinement - Production Validation):**
- 🧪 **CREDENTIAL TEST PATTERNS**: Added advanced examples with IIFE for complex credentials
  - How to test JSON array credentials
  - Dynamic URL construction in test requests
  - Error handling best practices
- 🔄 **MIGRATION GUIDE**: Complete @n8n/node-cli migration guide
  - Step-by-step migration from manual `tsc` scripts
  - ESLint v9 configuration
  - Benefits and checklist
- ⚡ **VERIFIED IN PRODUCTION**: All patterns tested on ServerChan v3.1.0
  - Credential test with JSON config ✅
  - Simplified subtitle expressions ✅
  - TypeScript strict mode ✅
  - @n8n/node-cli build system ✅

**Changes in v3.0 (Major Update):**
- 🎯 **RENAMED**: `n8n-node-builder` → `n8n-nodes-builder` (plural, more accurate)
- 🔧 **CLARIFIED SCOPE**: Explicitly states this is Node.js/TypeScript backend development, NOT Vue
- 📚 **ARCHITECTURE SECTION**: Added clear explanation of n8n's frontend vs backend separation
- 🎨 **UI GENERATION**: Emphasized JSON-based UI configuration vs writing Vue components
- 📋 **PROPERTY TYPES**: Added comprehensive list of supported property types
- 🔄 **RELATIONSHIP**: Clarified relationship with Vue development (separate concerns)
- ✨ **ENHANCED PHASE 5**: Completely rewritten to explain how n8n auto-renders UI from JSON config

**Changes in v2.1:**
- ⚠️ **CRITICAL**: Added Operation Dropdown Requirement section
- Reversed guidance on single-option dropdowns (MUST keep them)
- Explained n8n action list mechanism
- Added real-world example from Jintiankansha
- Updated Quality Checklist with operation dropdown checks
- Clarified UI discoverability vs cleanliness trade-off

**Changes in v2.0:**
- Added UI_DESIGN_PATTERNS.md reference
- Added ZERO_DEPENDENCIES.md reference
- Added TROUBLESHOOTING.md reference
- Enhanced Phase 5: Design Node UI
- Expanded Common Pitfalls (UI/UX and Technical sections)
- Updated Quality Checklist with UI/UX checks
- Improved Config + Action naming guidance
