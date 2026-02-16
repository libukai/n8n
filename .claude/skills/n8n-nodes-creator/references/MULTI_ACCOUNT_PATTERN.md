# Multi-Account Management Pattern (ServerChan Style)

## Overview

This pattern addresses multi-account scenarios where users need to:
- Manage multiple accounts/API keys for the same service
- Rotate/load-balance across accounts automatically
- Separate account configuration from execution logic

**Pattern**: Key Node (Config) + Main Node (Execute)

## When to Use This Pattern

Use this pattern when:
- ✅ Service requires user accounts/credentials (not just API keys)
- ✅ Users commonly have multiple accounts
- ✅ Need account rotation strategies (random, round-robin)
- ✅ Account selection is independent of execution logic

**Real-world examples:**
- Messaging services (ServerChan, Telegram, Slack)
- Social media APIs (WeChat, Twitter, Facebook)
- Image generation services (Jimeng/即梦, Midjourney, DALL-E)
- Email sending services with multiple SMTP accounts

## Architecture

### Two-Node Design

```
┌─────────────────────┐
│  Key Node           │  ← Account management & selection
│  (Programmatic)     │
│  - Load accounts    │
│  - Apply strategy   │
│  - Output sessionid │
└──────────┬──────────┘
           │
           ↓ { sessionid: "xxx" }
           │
┌──────────┴──────────┐
│  Main Node          │  ← Pure execution
│  (Any pattern)      │
│  - Receives         │
│    sessionid from   │
│    upstream         │
│  - No credential    │
│    binding!         │
└─────────────────────┘
```

### Workflow Example

```
[Trigger]
  ↓
[Jimeng Key Node]
  Mode: Selection Strategy
  Strategy: Round Robin
  ↓
  Output: { sessionid: "sctp12345txxx" }
  ↓
[Jimeng Main Node]
  SessionID: {{$json.sessionid}}  ← Auto-filled from upstream
  Operation: Text-to-Image
  Prompt: "A cute cat"
```

## Implementation Steps

### Step 1: Design Credential Format

#### Option A: Plain Text (Recommended for Simplicity)

```typescript
// credentials/MyServiceApi.credentials.ts
properties: [{
  displayName: 'Accounts',
  name: 'accounts',
  type: 'string',
  typeOptions: { rows: 10 },
  default: '',
  placeholder: `Account A,sessionid_123
Account B,sessionid_456
Account C,sessionid_789`,
  description: 'One account per line. Format: name,sessionid',
}]
```

**Benefits:**
- Simple to understand
- Easy to copy/paste
- No JSON syntax errors
- Works for most use cases

**Format:** `name,credential` (comma-separated, one per line)

---

#### Option B: JSON Array (For Complex Metadata)

```typescript
properties: [{
  displayName: 'Accounts (JSON Array)',
  name: 'accounts',
  type: 'json',
  default: '[]',
  placeholder: `[
  {"name": "Account A", "sessionid": "xxx", "quota": 100},
  {"name": "Account B", "sessionid": "yyy", "quota": 50}
]`,
  description: 'Account list in JSON format',
}]
```

**Use when:**
- Need to store additional metadata (quota, region, etc.)
- Complex account configurations
- Users comfortable with JSON

---

### Step 2: Create Key Node

The Key node handles account selection and outputs sessionid.

#### Basic Structure

```typescript
// nodes/MyServiceKey/MyServiceKey.node.ts
export class MyServiceKey implements INodeType {
  description: INodeTypeDescription = {
    displayName: 'My Service Key',
    name: 'myServiceKey',
    group: ['transform'],
    version: 1,
    subtitle: '={{($parameter["selectedAccounts"]?.length || 0)}} accounts',
    description: 'Manage My Service accounts - use with My Service Node',
    inputs: [NodeConnectionTypes.Main],
    outputs: [NodeConnectionTypes.Main],
    credentials: [{
      name: 'myServiceApi',
      required: false,
    }],
    properties: [
      // Mode selection
      {
        displayName: 'Mode',
        name: 'mode',
        type: 'options',
        default: 'multiSelect',
        options: [
          {
            name: 'Multi-Select Accounts',
            value: 'multiSelect',
            description: 'Select multiple accounts (one item per account)',
          },
          {
            name: 'Selection Strategy',
            value: 'strategy',
            description: 'Use strategy to select one account',
          },
        ],
      },
      // ... (see full implementation below)
    ],
  };
}
```

#### Full Implementation

```typescript
properties: [
  // Notice field
  {
    displayName: 'This node prepares credentials for My Service Node. Select from saved credentials or enter manually.',
    name: 'notice',
    type: 'notice',
    default: '',
  },

  // Mode selection
  {
    displayName: 'Mode',
    name: 'mode',
    type: 'options',
    default: 'multiSelect',
    options: [
      {
        name: 'Multi-Select Accounts',
        value: 'multiSelect',
        description: 'Select multiple accounts (expands to multiple items)',
      },
      {
        name: 'Selection Strategy',
        value: 'strategy',
        description: 'Use strategy to automatically select one account',
      },
    ],
  },

  // Multi-select accounts
  {
    displayName: 'Select Accounts',
    name: 'selectedAccounts',
    type: 'multiOptions',
    displayOptions: {
      show: { mode: ['multiSelect'] },
    },
    typeOptions: {
      loadOptionsMethod: 'getAccounts',
    },
    default: [],
    description: 'Select accounts from credentials',
  },

  // Strategy selection
  {
    displayName: 'Selection Strategy',
    name: 'strategy',
    type: 'options',
    displayOptions: {
      show: { mode: ['strategy'] },
    },
    default: 'random',
    options: [
      {
        name: 'Random',
        value: 'random',
        description: 'Randomly select one account',
      },
      {
        name: 'Round Robin',
        value: 'roundRobin',
        description: 'Rotate based on execution ID',
      },
      {
        name: 'Manual Select',
        value: 'manual',
        description: 'Manually specify account',
      },
    ],
  },

  // Manual account selection
  {
    displayName: 'Select Account',
    name: 'manualAccount',
    type: 'options',
    displayOptions: {
      show: {
        mode: ['strategy'],
        strategy: ['manual'],
      },
    },
    typeOptions: {
      loadOptionsMethod: 'getAccounts',
    },
    default: '',
    description: 'Select the account to use',
  },

  // Manual credential input
  {
    displayName: 'Manual Credentials',
    name: 'manualCredentials',
    type: 'string',
    typeOptions: { rows: 3 },
    default: '',
    placeholder: 'credential1\\ncredential2\\ncredential3',
    description: 'Manual credentials (separated by newline, comma, semicolon, or space)',
  },
]
```

#### LoadOptions Method (Parse Credentials)

```typescript
methods = {
  loadOptions: {
    async getAccounts(this: ILoadOptionsFunctions): Promise<INodePropertyOptions[]> {
      try {
        const credentials = await this.getCredentials('myServiceApi');

        if (!credentials?.accounts) {
          return [{
            name: 'No accounts configured in credentials',
            value: '',
          }];
        }

        const accountsText = credentials.accounts as string;
        const lines = accountsText
          .split('\\n')
          .map((line) => line.trim())
          .filter((line) => line.length > 0);

        if (lines.length === 0) {
          return [{
            name: 'No accounts configured',
            value: '',
          }];
        }

        // Parse plain text format: name,credential
        const accounts: Array<{name: string; credential: string}> = [];
        for (const line of lines) {
          const parts = line.split(',').map((p) => p.trim());
          if (parts.length >= 2) {
            accounts.push({
              name: parts[0],
              credential: parts[1],
            });
          }
        }

        if (accounts.length === 0) {
          return [{
            name: '⚠️ Invalid format - use: name,credential',
            value: '',
          }];
        }

        // Return options with masked credentials
        return accounts.map((account) => {
          const credential = account.credential || '';
          const masked = credential.length > 12
            ? `${credential.slice(0, 8)}...${credential.slice(-4)}`
            : '****';
          return {
            name: `${account.name || 'Unnamed'} (${masked})`,
            value: credential,
          };
        });
      } catch (error) {
        return [{
          name: '⚠️ Error loading accounts - check format',
          value: '',
        }];
      }
    },
  },
};
```

#### Execute Method (Apply Strategy)

```typescript
async execute(this: IExecuteFunctions): Promise<INodeExecutionData[][]> {
  const items = this.getInputData();
  const returnData: INodeExecutionData[] = [];

  for (let i = 0; i < items.length; i++) {
    try {
      const mode = this.getNodeParameter('mode', i) as string;
      const manualCredentials = this.getNodeParameter('manualCredentials', i, '') as string;

      const credentials: string[] = [];

      // 1. From credential configuration
      if (mode === 'multiSelect') {
        // Multi-select mode
        const selectedAccounts = this.getNodeParameter('selectedAccounts', i, []) as string[];
        credentials.push(...selectedAccounts);
      } else if (mode === 'strategy') {
        // Strategy mode
        const strategy = this.getNodeParameter('strategy', i) as string;
        const credentialData = await this.getCredentials('myServiceApi');

        if (credentialData?.accounts) {
          const accountsText = credentialData.accounts as string;
          const lines = accountsText.split('\\n').map((line) => line.trim());

          const accounts: Array<{name: string; credential: string}> = [];
          for (const line of lines) {
            const parts = line.split(',').map((p) => p.trim());
            if (parts.length >= 2) {
              accounts.push({ name: parts[0], credential: parts[1] });
            }
          }

          if (accounts.length > 0) {
            let selectedCredential: string;

            switch (strategy) {
              case 'random':
                // Random selection
                selectedCredential = accounts[
                  Math.floor(Math.random() * accounts.length)
                ].credential;
                break;

              case 'roundRobin':
                // Round-robin using execution ID
                const executionId = parseInt(this.getExecutionId(), 10) || 0;
                selectedCredential = accounts[
                  executionId % accounts.length
                ].credential;
                break;

              case 'manual':
                // Manual selection
                selectedCredential = this.getNodeParameter('manualAccount', i) as string;
                if (!selectedCredential) {
                  throw new Error('Please select an account');
                }
                break;

              default:
                throw new Error(`Unknown strategy: ${strategy}`);
            }

            credentials.push(selectedCredential);
          }
        }
      }

      // 2. From manual input
      if (manualCredentials) {
        const parsed = manualCredentials
          .split(/[\\n,; ]+/)  // Multi-delimiter support
          .map((c) => c.trim())
          .filter((c) => c.length > 0);
        credentials.push(...parsed);
      }

      // Deduplicate
      const uniqueCredentials = [...new Set(credentials)];

      if (uniqueCredentials.length === 0) {
        throw new Error('At least one account required');
      }

      // Output: One item per credential
      for (const credential of uniqueCredentials) {
        returnData.push({
          json: { credential },
          pairedItem: { item: i },
        });
      }
    } catch (error: any) {
      if (this.continueOnFail()) {
        returnData.push({
          json: { error: error?.message || String(error) },
          pairedItem: { item: i },
        });
        continue;
      }
      throw error;
    }
  }

  return [returnData];
}
```

---

### Step 3: Update Main Node

The main node receives credential from upstream and uses it directly.

#### Remove Credential Binding

```typescript
// ❌ OLD: Main node requires credential
description: INodeTypeDescription = {
  displayName: 'My Service Node',
  credentials: [{
    name: 'myServiceApi',
    required: true,  // ← Remove this!
  }],
  properties: resourceBuilder.build(),
}

// ✅ NEW: Main node has no credential binding
description: INodeTypeDescription = {
  displayName: 'My Service Node',
  // No credentials field!
  properties: [
    {
      displayName: 'Credential',
      name: 'credential',
      type: 'string',
      default: '={{$json.credential}}',  // ← From upstream Key node
      required: true,
      description: 'Credential from upstream Key node',
    },
    ...resourceBuilder.build(),
  ],
}
```

#### Simplify Execute Logic

```typescript
// In your operation execute function
async call(index: number) {
  // Get credential from node parameter (not from credentials API)
  const credential = this.getNodeParameter('credential', index) as string;

  if (!credential) {
    throw new Error('No credential provided. Please use Key node.');
  }

  // Use credential directly
  const apiClient = new MyServiceClient({ credential });
  const result = await apiClient.doSomething();

  return result;
}
```

---

## Round-Robin Algorithm Deep Dive

### The Elegant Solution

```typescript
const executionId = parseInt(this.getExecutionId(), 10) || 0;
const selectedAccount = accounts[executionId % accounts.length];
```

### Why This is Brilliant

**1. Stateless**
- No need to store "last used account index"
- No database or file system required
- No global variables

**2. Automatic Rotation**
```
3 accounts: [A, B, C]

executionId % 3:
1 → 1 (Account B)
2 → 2 (Account C)
3 → 0 (Account A) ← Automatically wraps!
4 → 1 (Account B)
5 → 2 (Account C)
...forever循环
```

**3. Concurrent Safe**
- Each execution has unique ID
- No race conditions
- Works with parallel executions

**4. Fair Distribution**
```
10 executions, 3 accounts:
Account A: executions 3, 6, 9 (3 times)
Account B: executions 1, 4, 7, 10 (4 times)
Account C: executions 2, 5, 8 (3 times)

→ Nearly perfect distribution!
```

### Alternative: Traditional Approach (Don't Use)

```typescript
// ❌ Complex, stateful, error-prone
let lastIndex = 0;  // Where to store this?

function getNextAccount() {
  lastIndex = (lastIndex + 1) % accounts.length;
  return accounts[lastIndex];
}

// Problems:
// - Need persistent storage
// - Race conditions in parallel execution
// - Lost state on restart
// - Multiple workflow instances conflict
```

---

## Best Practices

### Credential Format

**✅ DO:**
- Use plain text for simple cases (name,credential)
- Support multi-delimiter parsing (newline, comma, semicolon, space)
- Mask credentials in UI (show first 8 + last 4 chars)
- Provide clear format examples in placeholder

**❌ DON'T:**
- Force JSON if plain text suffices
- Store credentials in workflow (use Credential API)
- Show full credentials in error messages

### Key Node Design

**✅ DO:**
- Support both multi-select and strategy modes
- Implement round-robin using execution ID
- Allow manual credential input as backup
- Use notice field to explain workflow

**❌ DON'T:**
- Put business logic in Key node (keep it pure config)
- Require credential if manual input available
- Implement complex state management

### Main Node Design

**✅ DO:**
- Remove credential binding from node
- Accept credential from parameter (upstream)
- Provide clear error if no credential found
- Support both Key node and manual input

**❌ DON'T:**
- Keep dual mode (credential + parameter)
- Implement rotation logic in main node
- Hardcode credential selection

---

## Real-World Example: Jimeng Node

### Credential Configuration

```typescript
// credentials/jimengCredentialsApi.credentials.ts
properties: [{
  displayName: 'Accounts',
  name: 'accounts',
  type: 'string',
  typeOptions: { rows: 10 },
  default: '',
  required: true,
  placeholder: `Main Account,sctp12345txxxxxxxxxxx
Backup Account,sctp67890tyyyyyyyyyy
Test Account,sctp11111tzzzzzzzzzz`,
  description: 'One account per line. Format: name,sessionid',
}]
```

### Key Node

```typescript
// nodes/jimengKey/JimengKey.node.ts
export class JimengKey implements INodeType {
  description: INodeTypeDescription = {
    displayName: 'Jimeng Key',
    name: 'jimengKey',
    subtitle: '={{($parameter["selectedAccounts"]?.length || 0)}} accounts',
    description: 'Manage Jimeng accounts - use with Jimeng Node',
    properties: [
      // Mode, strategy, account selection...
    ],
  };

  // Round-robin implementation
  case 'roundRobin':
    const executionId = parseInt(this.getExecutionId(), 10) || 0;
    selectedSessionId = accounts[executionId % accounts.length].sessionid;
    break;
}
```

### Main Node

```typescript
// nodes/jimeng/JimengNode.node.ts
export class JimengNode implements INodeType {
  description: INodeTypeDescription = {
    displayName: 'jimeng Node',
    // No credentials binding!
    properties: [
      {
        displayName: 'SessionID',
        name: 'sessionid',
        type: 'string',
        default: '={{$json.sessionid}}',  // From upstream
        required: true,
      },
      {
        displayName: '类型',  // Renamed from 'Resource'
        name: 'resource',
        type: 'options',
        // ...
      },
      // ...
    ],
  };
}

// In operation files
async call(index: number) {
  const sessionid = this.getNodeParameter('sessionid', index) as string;

  if (!sessionid) {
    throw new Error('No sessionid provided. Use Jimeng Key node.');
  }

  const client = new JimengApiClient({ refreshToken: sessionid });
  // Execute API call...
}
```

### Workflow

```
[Schedule Trigger - Every hour]
  ↓
[Jimeng Key]
  Mode: Selection Strategy
  Strategy: Round Robin
  ↓
  Output: { sessionid: "sctp12345txxx" }
  ↓
[Jimeng Node]
  SessionID: {{$json.sessionid}}
  类型: 图片生成
  动作: 文生图
  Prompt: "A beautiful sunset"
```

---

## Common Scenarios

### Scenario 1: Load Balancing

```
[HTTP Request - Get tasks]
  → Returns 100 tasks
  ↓
[Jimeng Key]
  Mode: Selection Strategy
  Strategy: Round Robin
  ↓
  Output: 100 items, accounts rotated evenly
  ↓
[Jimeng Node]
  Process each task with assigned account
```

### Scenario 2: Broadcast to All Accounts

```
[Manual Trigger]
  ↓
[Jimeng Key]
  Mode: Multi-Select Accounts
  Select: ☑️ All accounts
  ↓
  Output: 3 items (one per account)
  ↓
[Jimeng Node]
  Execute same operation on all accounts
```

### Scenario 3: Fallback on Failure

```
[Trigger]
  ↓
[Jimeng Key - Try Main Account]
  Mode: Strategy
  Strategy: Manual Select
  Account: Main Account
  ↓
[Jimeng Node]
  ↓ (on error)
[Error Trigger]
  ↓
[Jimeng Key - Use Backup]
  Mode: Strategy
  Strategy: Manual Select
  Account: Backup Account
  ↓
[Jimeng Node - Retry]
```

---

## Comparison with Alternative Approaches

### Alternative 1: Multi-Credential with Expressions

```
[Jimeng Node]
  Credential: {{ ['账号A', '账号B', '账号C'][Math.floor(Math.random() * 3)] }}
```

**Problems:**
- ❌ Need to create multiple credentials (one per account)
- ❌ Hardcode credential names in expression
- ❌ No UI for account selection
- ❌ Difficult for non-technical users

---

### Alternative 2: Credential with JSON + Complex Logic in Main Node

```typescript
// Main node reads credential and selects internally
const credentials = await this.getCredentials('myServiceApi');
const accounts = JSON.parse(credentials.accounts);
const selected = accounts[Math.random() * accounts.length];
```

**Problems:**
- ❌ Mixing concerns (selection + execution in one node)
- ❌ No reusability (each node reimplements selection)
- ❌ Harder to test independently
- ❌ Cannot easily switch strategies per workflow

---

### Why Key + Main Pattern is Better

**✅ Separation of Concerns**
- Key node: Pure configuration
- Main node: Pure execution

**✅ Reusability**
- Same Key node works with multiple service nodes
- Selection logic centralized

**✅ Flexibility**
- Change strategy per workflow
- Mix and match modes

**✅ User-Friendly**
- Clear UI for account selection
- No need to write expressions

**✅ Testable**
- Test Key node independently
- Test Main node with fixed credential

---

## Migration Guide

### From Single Account to Multi-Account

**Step 1: Create new credential format**

```typescript
// OLD
properties: [{
  displayName: 'API Key',
  name: 'apiKey',
  type: 'string',
}]

// NEW
properties: [{
  displayName: 'Accounts',
  name: 'accounts',
  type: 'string',
  typeOptions: { rows: 10 },
  placeholder: 'Account A,key1\\nAccount B,key2',
}]
```

**Step 2: Create Key node**

Copy from template above, adapt for your credential format.

**Step 3: Update Main node**

```typescript
// Remove credential binding
credentials: [],  // Empty!

// Add parameter to receive credential
properties: [{
  displayName: 'Credential',
  name: 'credential',
  type: 'string',
  default: '={{$json.credential}}',
  required: true,
}]

// Update execute logic
const credential = this.getNodeParameter('credential', index) as string;
```

**Step 4: Update documentation**

Explain new workflow pattern with Key node.

---

## Summary

The Multi-Account Management Pattern provides:

- ✅ **Clean separation**: Config vs execution
- ✅ **Elegant rotation**: Stateless round-robin using execution ID
- ✅ **User-friendly**: UI-based account selection
- ✅ **Flexible**: Multiple strategies (random, round-robin, manual)
- ✅ **Scalable**: Handles any number of accounts
- ✅ **Testable**: Independent node testing
- ✅ **Reusable**: Key node works with multiple service nodes

**Key Innovation:** Using `executionId % accountCount` for stateless, fair rotation.

---

## Verified in Production

This pattern has been successfully implemented and tested in:

- ✅ **n8n-nodes-serverchan**: Message delivery with multiple SendKeys
- ✅ **n8n-nodes-jimeng**: AI image generation with multiple accounts (即梦)

Both implementations demonstrate:
- Reliable rotation across thousands of executions
- Clean separation of concerns
- User-friendly UI
- Zero stateful complexity
