# API Contracts: Overdue Todo Items Feature

**Feature**: Support for Overdue Todo Items  
**Date**: December 12, 2025  
**Phase**: Phase 1 - Design

## Overview

This document specifies the API contracts for the overdue todo feature. **Note**: No new endpoints or backend changes are required. The overdue feature uses existing todo API endpoints with client-side calculation.

---

## Existing API Endpoints

All overdue calculations happen on the client side. The backend continues to serve todo data with no modifications.

### Base URL

```
http://localhost:3030/api
```

---

## Endpoints

### 1. GET /api/todos

Retrieves all todos. Client calculates overdue status based on `dueDate` and `completed` fields.

**Request**:
```http
GET /api/todos
Content-Type: application/json
```

**Response** (200 OK):
```json
[
  {
    "id": 1,
    "title": "Complete project documentation",
    "dueDate": "2025-12-10",
    "completed": false,
    "createdAt": "2025-12-01T14:30:00Z"
  },
  {
    "id": 2,
    "title": "Review pull request",
    "dueDate": "2025-12-15",
    "completed": true,
    "createdAt": "2025-12-05T09:00:00Z"
  },
  {
    "id": 3,
    "title": "Plan next sprint",
    "dueDate": null,
    "completed": false,
    "createdAt": "2025-12-08T11:20:00Z"
  }
]
```

**Client-Side Processing**:
```javascript
// Client calculates overdue status for each todo
const todosWithOverdueStatus = todos.map(todo => ({
  ...todo,
  overdueStatus: calculateOverdueStatus(todo.dueDate, todo.completed)
}));

// Client counts overdue items
const overdueCount = todos.filter(todo => 
  !todo.completed && isOverdue(todo.dueDate)
).length;
```

---

### 2. POST /api/todos

Creates a new todo. No changes to request/response format.

**Request**:
```http
POST /api/todos
Content-Type: application/json

{
  "title": "New todo item",
  "dueDate": "2025-12-20"
}
```

**Response** (201 Created):
```json
{
  "id": 4,
  "title": "New todo item",
  "dueDate": "2025-12-20",
  "completed": false,
  "createdAt": "2025-12-12T10:00:00Z"
}
```

**Client-Side Processing**:
```javascript
// After creation, client calculates if new todo is overdue
const overdueStatus = calculateOverdueStatus(newTodo.dueDate, newTodo.completed);
```

---

### 3. PUT /api/todos/:id

Updates an existing todo. No changes to request/response format.

**Request**:
```http
PUT /api/todos/1
Content-Type: application/json

{
  "title": "Updated title",
  "dueDate": "2025-12-18",
  "completed": false
}
```

**Response** (200 OK):
```json
{
  "id": 1,
  "title": "Updated title",
  "dueDate": "2025-12-18",
  "completed": false,
  "createdAt": "2025-12-01T14:30:00Z"
}
```

**Client-Side Processing**:
```javascript
// After update, client recalculates overdue status
const overdueStatus = calculateOverdueStatus(updatedTodo.dueDate, updatedTodo.completed);

// If todo was completed, remove overdue indicator
if (updatedTodo.completed) {
  // Overdue status = null
}
```

---

### 4. DELETE /api/todos/:id

Deletes a todo. No changes to request/response format.

**Request**:
```http
DELETE /api/todos/1
```

**Response** (204 No Content)

**Client-Side Processing**:
```javascript
// After deletion, client recalculates overdue count
const remainingTodos = todos.filter(t => t.id !== deletedId);
const overdueCount = remainingTodos.filter(todo => 
  !todo.completed && isOverdue(todo.dueDate)
).length;
```

---

## Client-Side Data Transformation

### Overdue Status Calculation

**Input**: Todo object from API
```json
{
  "id": 1,
  "title": "Complete project documentation",
  "dueDate": "2025-12-10",
  "completed": false,
  "createdAt": "2025-12-01T14:30:00Z"
}
```

**Transformation** (client-side):
```javascript
import { calculateOverdueStatus } from '../utils/dateUtils';

const overdueStatus = calculateOverdueStatus(todo.dueDate);
// Current date: 2025-12-15

// Result:
{
  isOverdue: true,
  daysOverdue: 5,
  weeksOverdue: 0
}
```

**Enhanced Todo Object** (in memory only):
```javascript
{
  id: 1,
  title: "Complete project documentation",
  dueDate: "2025-12-10",
  completed: false,
  createdAt: "2025-12-01T14:30:00Z",
  // Computed properties (not persisted)
  overdueStatus: {
    isOverdue: true,
    daysOverdue: 5,
    weeksOverdue: 0
  }
}
```

---

## Data Schema Reference

### Todo Object

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | number | Yes | Unique identifier (auto-generated) |
| `title` | string | Yes | Todo title (max 255 characters) |
| `dueDate` | string\|null | No | ISO 8601 date (YYYY-MM-DD) or null |
| `completed` | boolean | Yes | Completion status (default: false) |
| `createdAt` | string | Yes | ISO 8601 timestamp (auto-generated) |

### Overdue Status (Computed Client-Side)

| Field | Type | Description |
|-------|------|-------------|
| `isOverdue` | boolean | True if past due date and not completed |
| `daysOverdue` | number | Number of days past due date |
| `weeksOverdue` | number | Number of complete weeks past due date |

---

## Error Handling

All error handling follows existing API patterns. No new error scenarios introduced by overdue feature.

### Invalid Due Date Format

**Backend**: Accepts any string for `dueDate` field (no validation)

**Client**: Validates and handles invalid dates gracefully

```javascript
// Backend stores as-is
{ "dueDate": "invalid-date" }

// Client treats as null (not overdue)
calculateOverdueStatus("invalid-date") // Returns null
```

### Network Errors

Follow existing error handling patterns. Overdue calculations are deferred until data loads successfully.

```javascript
try {
  const todos = await fetchTodos();
  const todosWithOverdue = todos.map(todo => ({
    ...todo,
    overdueStatus: calculateOverdueStatus(todo.dueDate)
  }));
  setTodos(todosWithOverdue);
} catch (error) {
  console.error('Failed to fetch todos:', error);
  setError('Failed to load todos. Please try again.');
}
```

---

## WebSocket / Real-Time Updates

**Not Applicable**: Application does not use WebSocket or real-time sync. Overdue status updates on:
1. Page load
2. Manual refresh
3. CRUD operations (create, update, delete)
4. Midnight rollover (via `setTimeout`)

---

## Summary

### Backend Requirements
- ✅ **No changes required** to existing API endpoints
- ✅ **No new endpoints needed**
- ✅ **No schema migrations needed**

### Frontend Requirements
- ✅ Uses existing GET /api/todos endpoint
- ✅ Calculates overdue status client-side using `dateUtils.js`
- ✅ No new API calls introduced by overdue feature

### Performance
- **API Impact**: None (same number of API calls)
- **Computation**: < 1ms per todo for overdue calculation
- **Rendering**: < 5ms per render cycle (with memoization)

### Future Considerations

If backend optimization is needed in the future:

**Optional Backend Endpoint** (not implemented now):
```http
GET /api/todos?filter=overdue

Response:
[
  // Only return overdue todos
  // Calculated server-side
]
```

**Rationale for Not Implementing**:
- Single-user application (no load concerns)
- Client-side calculation is fast (< 1ms)
- No network latency benefit (same data transferred)
- KISS principle: Keep it simple
