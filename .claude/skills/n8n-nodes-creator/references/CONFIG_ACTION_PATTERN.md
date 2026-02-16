# Config + Action Pattern (Separation of Concerns)

## Overview

The Config + Action pattern separates node responsibilities into two distinct nodes:
- **Config Node**: Manages configuration, prepares data, handles complex logic
- **Action Node**: Pure execution, typically using Declarative routing

## When to Use

Use this pattern when your node needs to handle:

1. **Batch Operations**: Send to multiple recipients, update multiple records
2. **Complex Configuration**: Multiple data sources, conditional logic, validation
3. **Result Aggregation**: Track success/failure counts, compile reports

## Pattern Structure

```
[Config Node] → [Action Node]
   Programmatic    Declarative
   Complex logic   Simple execution
   Outputs items   Auto-loops
```

---

## Benefits

### 1. Single Responsibility Principle

Each node does ONE thing well:
- Config: Manage complexity
- Action: Execute simply

### 2. Code Simplicity

- Config Node: ~120 lines (handles all complexity)
- Action Node: ~10 lines (pure Declarative!)

### 3. High Reusability

Action node can be used in multiple workflows:

```
Scenario 1: Batch send
[Config] → [Action]

Scenario 2: Conditional send
[Config] → [IF] → [Action]

Scenario 3: Delayed send
[Config] → [Wait] → [Action]

Scenario 4: With retry
[Config] → [Action] → [Error Trigger] → [Action]
```

### 4. Easy Testing

Test each node independently:
- Config: Check output items
- Action: Feed test items directly

---

## Implementation Guide

### Step 1: Design Data Flow

Define the contract between nodes:

```typescript
// Config node outputs:
{
  json: {
    // Required fields for Action node
    key1: value1,
    key2: value2,
    // Optional fields
    metadata: { ... }
  }
}

// Action node expects:
{
  json: {
    key1: string,  // Required
    key2: string,  // Required
  }
}
```

### Step 2: Implement Config Node (Programmatic)

```typescript
// nodes/MyServiceConfig/MyServiceConfig.node.ts
export class MyServiceConfig implements INodeType {
  description: INodeTypeDescription = {
    displayName: 'My Service Config',
    name: 'myServiceConfig',
    icon: 'file:myservice.svg',
    group: ['transform'],  // ⚠️ Use 'transform' group
    version: 1,
    description: 'Configure My Service parameters',
    inputs: [NodeConnectionTypes.Main],
    outputs: [NodeConnectionTypes.Main],
    credentials: [...],
    properties: [
      // Configuration parameters
      {
        displayName: 'Data Source',
        name: 'source',
        type: 'options',
        options: [
          { name: 'From Credentials', value: 'credentials' },
          { name: 'Manual Input', value: 'manual' },
        ],
        default: 'credentials',
      },
      // Message parameters
      {
        displayName: 'Title',
        name: 'title',
        type: 'string',
        default: '',
      },
      // ... more parameters
    ],
  };

  async execute(this: IExecuteFunctions): Promise<INodeExecutionData[][]> {
    const items = this.getInputData();
    const returnData: INodeExecutionData[] = [];

    for (let i = 0; i < items.length; i++) {
      // 1. Collect configuration
      const source = this.getNodeParameter('source', i) as string;
      const title = this.getNodeParameter('title', i) as string;

      // 2. Process configuration (complex logic here)
      let recipients: string[] = [];
      if (source === 'credentials') {
        // Load from credentials
        const creds = await this.getCredentials('myServiceApi');
        recipients = JSON.parse(creds.recipients as string);
      } else {
        // Parse manual input
        const manual = this.getNodeParameter('manualInput', i) as string;
        recipients = manual.split('\n').map(r => r.trim()).filter(r => r);
      }

      // 3. Deduplicate, validate, etc.
      const uniqueRecipients = [...new Set(recipients)];

      // 4. Generate output items (one per recipient)
      for (const recipient of uniqueRecipients) {
        returnData.push({
          json: {
            recipient,  // ✅ Action node will use this
            title,
            // ... other fields
          },
        });
      }
    }

    return [returnData];
  }
}
```

### Step 3: Implement Action Node (Declarative)

```typescript
// nodes/MyServiceSend/MyServiceSend.node.ts
export class MyServiceSend implements INodeType {
  description: INodeTypeDescription = {
    displayName: 'My Service Send',
    name: 'myServiceSend',
    icon: 'file:myservice.svg',
    group: ['output'],  // ⚠️ Use 'output' group
    version: 1,
    description: 'Send message via My Service',
    inputs: [NodeConnectionTypes.Main],
    outputs: [NodeConnectionTypes.Main],
    properties: [
      {
        displayName: 'Operation',
        name: 'operation',
        type: 'options',
        noDataExpression: true,
        options: [
          {
            name: 'Send Message',
            value: 'send',
            action: 'Send a message',
            description: 'Send message to recipient',
            // ✅ Pure Declarative!
            routing: {
              request: {
                method: 'POST',
                url: '=https://api.example.com/send/{{$json.recipient}}',
                body: {
                  title: '={{$json.title}}',
                  content: '={{$json.content}}',
                },
              },
            },
          },
        ],
        default: 'send',
      },
    ],
  };

  // ✅ No execute method needed! Declarative handles it.
}
```

**That's it! Action node is only ~60 lines total.**

---

## Real-World Example: ServerChan

### Before (One Node, Programmatic)

```typescript
// ServerChan.node.ts - 190 lines
export class ServerChan implements INodeType {
  properties: [
    { displayName: 'Select Recipients', ... },  // Config
    { displayName: 'Manual SendKeys', ... },     // Config
    { displayName: 'Title', ... },              // Message
    { displayName: 'Content', ... },            // Message
  ],

  async execute() {
    // Collect SendKeys
    // Merge, deduplicate
    // Send to all (parallel)
    // Aggregate results
    // ~150 lines of logic
  }
}
```

### After (Two Nodes, Separation)

```typescript
// ServerChanConfig.node.ts - 120 lines
export class ServerChanConfig implements INodeType {
  properties: [
    { displayName: 'Recipients Source', ... },
    { displayName: 'Select Recipients', ... },
    { displayName: 'Manual SendKeys', ... },
    { displayName: 'Title', ... },
    { displayName: 'Content', ... },
  ],

  async execute() {
    // Collect SendKeys
    // Merge, deduplicate
    // Output one item per SendKey
    // ~80 lines of logic
  }
}

// ServerChanSend.node.ts - 60 lines total
export class ServerChanSend implements INodeType {
  properties: [
    {
      name: 'Send Message',
      value: 'send',
      routing: {  // ✅ Only 10 lines!
        request: {
          method: 'POST',
          url: '=https://sctapi.ftqq.com/{{$json.sendKey}}.send',
          body: {
            title: '={{$json.title}}',
            desp: '={{$json.desp}}',
          },
        },
      },
    },
  ],
  // No execute method!
}
```

**Result**:
- Total lines: Similar (~180 lines)
- But Send node is MUCH simpler (10 lines of routing vs 150 lines of logic)
- Send node is reusable
- Each node is independently testable

---

## Data Flow Patterns

### Pattern 1: One-to-Many (Most Common)

```
Config: 1 input item → N output items
Action: N items → N executions (auto-loop)

Example:
Input: { recipients: ['a','b','c'], msg: 'Hi' }
         ↓
Config:  Outputs 3 items:
         - { recipient: 'a', msg: 'Hi' }
         - { recipient: 'b', msg: 'Hi' }
         - { recipient: 'c', msg: 'Hi' }
         ↓
Action:  Executes 3 times (automatically)
```

### Pattern 2: Transform-and-Execute

```
Config: Enrich/transform data
Action: Execute with enriched data

Example:
Input: { userId: '123', action: 'notify' }
         ↓
Config:  Fetch user details:
         { userId: '123', email: 'user@example.com', name: 'John' }
         ↓
Action:  Send email using enriched data
```

### Pattern 3: Filter-and-Execute

```
Config: Filter based on conditions
Action: Execute only for valid items

Example:
Input: { users: [...100 users...] }
         ↓
Config:  Filter active users:
         Output 30 items (only active)
         ↓
Action:  Execute 30 times (only for active users)
```

---

## Best Practices

### 1. Clear Node Naming ⭐

Use descriptive names that show the relationship:

✅ Good:
- `ServerChan Key` + `ServerChan Send` (Key better than Config)
- `Email Prepare` + `Email Deliver`
- `Notification Key` + `Notification Send`

❌ Bad:
- `ServerChan Config` + `ServerChan Send` (Config is vague)
- `ServerChan 1` + `ServerChan 2`
- `Setup` + `Run`

**Naming Convention:**
- **Config Node**: Use specific nouns (Key, Prepare, Builder)
- **Action Node**: Use action verbs (Send, Deliver, Execute)

### 2. Group Assignment

- Config Node: `group: ['transform']`
- Action Node: `group: ['output']` or `group: ['input']`

This helps users find nodes in the correct category.

### 3. Icon Consistency

Both nodes should use the same icon for visual continuity.

### 4. Parameter Naming

Use consistent parameter names between nodes:

```typescript
// Config outputs:
{ sendKey, title, desp }

// Action expects:
routing: {
  url: '=.../{{$json.sendKey}}.send',  // ✅ Same name
  body: {
    title: '={{$json.title}}',          // ✅ Same name
    desp: '={{$json.desp}}',            // ✅ Same name
  }
}
```

### 5. Documentation

Clearly document in node descriptions:

```typescript
// Config node
description: 'Configure recipients and message (use with ServerChan Send)',

// Action node
description: 'Send message (use after ServerChan Config)',
```

---

## Testing Strategy

### Test Config Node

```
1. Create test workflow:
   [Manual Trigger] → [Config Node]

2. Input various configurations:
   - From credentials
   - Manual input
   - Both sources

3. Check output:
   - Correct number of items
   - Proper deduplication
   - All parameters present
```

### Test Action Node

```
1. Create test workflow:
   [Manual Trigger] → [Action Node]

2. Manually create items:
   {
     json: {
       sendKey: 'test-key',
       title: 'Test',
       desp: 'Content'
     }
   }

3. Verify:
   - Request sent correctly
   - Response handled properly
```

### Test Integration

```
[Manual Trigger] → [Config] → [Action]

Verify end-to-end flow works correctly.
```

---

## Common Pitfalls

### ❌ Pitfall 1: Mixing Concerns

Don't put execution logic in Config node:

```typescript
// ❌ Bad
async execute() {
  const recipients = this.getNodeParameter('recipients');

  // ❌ Don't send here!
  for (const r of recipients) {
    await sendMessage(r);
  }
}

// ✅ Good
async execute() {
  const recipients = this.getNodeParameter('recipients');

  // ✅ Just output items
  return recipients.map(r => ({ json: { recipient: r, ... } }));
}
```

### ❌ Pitfall 2: Complex Action Node

Keep Action node simple:

```typescript
// ❌ Bad - Action node with complex logic
async execute() {
  // ... 100 lines of logic
}

// ✅ Good - Pure Declarative
routing: {
  request: { ... }  // 10 lines max
}
```

### ❌ Pitfall 3: Not Leveraging Auto-Loop

Don't manually loop in Action node:

```typescript
// ❌ Bad
async execute() {
  const items = this.getInputData();
  for (const item of items) {  // ❌ n8n does this automatically!
    await send(item.json.recipient);
  }
}

// ✅ Good - Let n8n auto-loop
routing: {
  request: {
    url: '=.../{{$json.recipient}}',  // ✅ Executed per item automatically
  }
}
```

---

## Migration Path

To migrate existing all-in-one nodes:

### Step 1: Identify Responsibilities

Analyze your current node:
- Configuration logic → Config node
- Execution logic → Action node

### Step 2: Design Data Contract

Define the JSON structure passed between nodes.

### Step 3: Create Config Node

Extract configuration logic, output items.

### Step 4: Create Action Node

Use Declarative routing for execution.

### Step 5: Version Strategy

**Option A: Keep both**
- v1.x: Original node (deprecated)
- v2.x: Config + Action nodes (recommended)

**Option B: Replace**
- v2.0: Breaking change, new architecture
- Provide migration guide

---

## Success Criteria

Your Config + Action implementation is ready when:

1. ✅ Config node outputs standardized items
2. ✅ Action node uses pure Declarative routing
3. ✅ Action node is < 100 lines total
4. ✅ Both nodes work independently
5. ✅ Integration test passes
6. ✅ Documentation explains the relationship

---

## Summary

The Config + Action pattern is ideal when:
- You need batch operations
- Configuration is complex
- Execution is simple (HTTP request)

**Key principle**: Separate concerns, maximize simplicity of Action node.

**Result**: Maintainable, testable, reusable nodes that follow n8n best practices.
