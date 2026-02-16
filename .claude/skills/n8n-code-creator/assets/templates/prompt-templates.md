# Prompt Templates for Understanding Requirements

Templates for clarifying user requirements and extracting necessary information for code generation.

## Overview

When user requests are ambiguous or incomplete, use these templates to:
- Clarify transformation requirements
- Extract schema information
- Understand edge cases
- Confirm understanding

---

## 1. Schema Clarification Templates

### When Schema is Missing
"To generate accurate code, I need to understand the input data structure. Could you provide:
- A sample of the actual data, OR
- A description of the fields and their types?"

### When Field Names are Unclear
"I see you have [concept] in your data. What is the exact field name for this?
- For example, is it `userId`, `user_id`, `id`, or something else?"

### When Nesting is Unclear
"Is the [field] directly at the root level, or is it nested inside another object?
- If nested, what's the full path to access it?"

### When Array vs Single Value is Unclear
"Is [field] an array containing multiple items, or a single value?"

---

## 2. Transformation Requirement Templates

### Understanding "Extract"
"When you say 'extract', do you mean:
1. Get specific fields from each item (keeping all items)
2. Get values from a nested array (flattening to separate items)
3. Find and extract patterns (like emails or URLs) from text?"

### Understanding "Filter"
"What criteria should I use to filter?
- Examples: amount > 1000, status === 'active', date after 2024-01-01"

### Understanding "Transform"
"How would you like the output structured?
- Could you show me an example of the desired output format?"

### Understanding "Calculate"
"What calculation do you need?
- Sum, average, count, min/max, or something else?
- Should this be per item or across all items?"

---

## 3. Edge Case Templates

### Handling Missing Data
"How should I handle cases where [field] is missing or null?
- Use a default value? Skip the item? Return an error?"

### Handling Empty Arrays
"What should happen if the input array is empty?
- Return empty array? Return specific message?"

### Handling Multiple Matches
"If there are multiple [items] matching the criteria, should I:
- Keep all matches?
- Keep only the first one?
- Keep only the last one?"

---

## 4. Confirmation Templates

### Schema Confirmation
"Based on your data, I understand the structure as:
```
field1 (type)
field2 (type)
  └─ nestedField (type)
```
Is this correct?"

### Transformation Confirmation
"So you want to:
1. [Step 1 description]
2. [Step 2 description]
3. [Step 3 description]

Output will be: [output description]

Is this what you're looking for?"

---

## 5. Category Classification

Use these questions to classify the request type:

**Extraction**:
- "Extract names and emails"
- "Get the ID from each order"
- "Pull out the addresses"

**Filtering**:
- "Only keep items where..."
- "Filter out..."
- "Remove all except..."

**Transformation**:
- "Convert to..."
- "Restructure as..."
- "Change format to..."

**Aggregation**:
- "Calculate total..."
- "Count how many..."
- "Average of..."

**Combination**:
- Multiple operations combined
- "Extract and filter..."
- "Calculate totals per category"

---

## Quick Reference

| User Says | Ask About | Category |
|-----------|-----------|----------|
| "Extract" | Which fields? Flatten arrays? | Extraction |
| "Filter" | What criteria? | Filtering |
| "Transform" | Output structure? | Transformation |
| "Calculate" | Which operation? Per item or total? | Aggregation |
| "Parse" | What pattern? | String Processing |
| "Group by" | Which field? What aggregation? | Transformation |
