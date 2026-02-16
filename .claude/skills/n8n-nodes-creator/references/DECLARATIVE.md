# Declarative Routing Reference

## What is Declarative?

Declarative routing is n8n's syntax sugar for HTTP requests. Instead of writing execute() methods, you configure how requests should be made using JSON.

## Core Concept

```typescript
// You write (configuration):
routing: {
  request: {
    method: 'GET',
    url: '=/users/{{$parameter.userId}}'
  }
}

// n8n automatically does:
async execute() {
  const userId = this.getNodeParameter('userId', 0);
  const response = await this.helpers.httpRequest({
    method: 'GET',
    url: `/users/${userId}`,
  });
  return response;
}
```

## Basic Syntax

### Simple GET Request

```typescript
{
  name: 'Get User',
  value: 'getUser',
  routing: {
    request: {
      method: 'GET',
      url: '=/api/users/{{$parameter.userId}}',
    },
  },
}
```

### POST Request with Body

```typescript
{
  name: 'Create User',
  value: 'createUser',
  routing: {
    request: {
      method: 'POST',
      url: '/api/users',
      body: {
        name: '={{$parameter.userName}}',
        email: '={{$parameter.userEmail}}',
      },
    },
  },
}
```

### Query Parameters

```typescript
routing: {
  request: {
    method: 'GET',
    url: '/api/users',
    qs: {
      page: '={{$parameter.page}}',
      limit: '={{$parameter.limit}}',
    },
  },
}
```

## Parameter Interpolation

### From Node Parameters

Use `$parameter`:

```typescript
url: '=/users/{{$parameter.userId}}'
```

### From Input Data

Use `$json`:

```typescript
url: '=/users/{{$json.userId}}'
```

### From Credentials

Use `$credentials`:

```typescript
headers: {
  'Authorization': '=Bearer {{$credentials.apiKey}}'
}
```

## When to Use Declarative

✅ **Perfect for**:
- Standard RESTful CRUD operations
- Each operation = 1 HTTP request
- Simple parameter mapping
- 80% of API integrations

❌ **Not suitable for**:
- Batch operations (1 operation → N requests)
- Complex logic (loops, conditionals)
- Using third-party SDKs
- Multi-step workflows

## Complete Example

```typescript
// resources/user/index.ts
export const userDescription: INodeProperties[] = [
  {
    displayName: 'Operation',
    name: 'operation',
    type: 'options',
    options: [
      {
        name: 'Get',
        value: 'get',
        routing: {
          request: {
            method: 'GET',
            url: '=/users/{{$parameter.userId}}',
          },
        },
      },
      {
        name: 'Create',
        value: 'create',
        routing: {
          request: {
            method: 'POST',
            url: '/users',
            body: {
              name: '={{$parameter.name}}',
              email: '={{$parameter.email}}',
            },
          },
        },
      },
    ],
    default: 'get',
  },
];
```

## Benefits

1. **Less Code**: 60-80% reduction
2. **Less Bugs**: n8n handles execution
3. **Auto Features**: Retry, error handling, pagination (if configured)
4. **Easy to Read**: JSON config is self-documenting

## Limitations

1. **No Loops**: Can't iterate over arrays
2. **No Conditionals**: Can't use if/else
3. **Single Request**: One operation = one HTTP call
4. **No SDK**: Must use HTTP directly

## When in Doubt

Start with Declarative if your operation is a simple HTTP request. You can always refactor to Programmatic later if needed.
