# Data Model: Overdue Todo Items Feature

**Feature**: Support for Overdue Todo Items  
**Date**: December 12, 2025  
**Phase**: Phase 1 - Design

## Overview

This document defines the data structures and business logic for calculating and displaying overdue status for todo items. The overdue feature is entirely computed on-demand and does not require database schema changes.

---

## Entities

### 1. Todo Item (Existing)

The Todo entity already exists in the system. No schema changes required.

**Database Schema** (SQLite via better-sqlite3):
```sql
CREATE TABLE todos (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  title TEXT NOT NULL,
  dueDate TEXT,  -- ISO 8601 format (YYYY-MM-DD) or NULL
  completed BOOLEAN NOT NULL DEFAULT 0,
  createdAt TEXT NOT NULL DEFAULT (datetime('now'))
);
```

**JavaScript Representation**:
```javascript
{
  id: number,           // Auto-incremented primary key
  title: string,        // Required, max 255 characters
  dueDate: string|null, // ISO 8601 format (YYYY-MM-DD) or null
  completed: boolean,   // Default: false
  createdAt: string     // ISO 8601 timestamp
}
```

**Example**:
```javascript
{
  id: 1,
  title: "Complete project documentation",
  dueDate: "2025-12-10",
  completed: false,
  createdAt: "2025-12-01T14:30:00Z"
}
```

---

### 2. Overdue Status (Computed Property - NEW)

Overdue status is calculated on-demand in the frontend and is **not persisted** in the database.

**Interface**:
```javascript
interface OverdueStatus {
  isOverdue: boolean;      // True if todo is overdue
  daysOverdue: number;     // Number of days past due date
  weeksOverdue: number;    // Number of complete weeks overdue (floor(daysOverdue / 7))
}
```

**Example**:
```javascript
// For a todo due on 2025-12-10, viewed on 2025-12-15
{
  isOverdue: true,
  daysOverdue: 5,
  weeksOverdue: 0
}

// For a todo due on 2025-12-01, viewed on 2025-12-15
{
  isOverdue: true,
  daysOverdue: 14,
  weeksOverdue: 2
}
```

**Calculation Rules**:
1. **Not overdue** if:
   - `dueDate` is `null` or invalid
   - `completed` is `true`
   - `dueDate` >= current date (UTC)
   
2. **Overdue** if:
   - `dueDate` is valid
   - `dueDate` < current date (UTC, normalized to midnight)
   - `completed` is `false`

3. **Display format**:
   - Days: Used for 1-6 days overdue (e.g., "2 days overdue")
   - Weeks: Used for 7+ days overdue (e.g., "1 week overdue", "2 weeks overdue")
   - Singular/plural: "1 day", "2 days", "1 week", "2 weeks"

---

## Business Logic

### Date Calculation Utility

**Location**: `packages/frontend/src/utils/dateUtils.js`

```javascript
/**
 * Calculates overdue status for a todo item
 * @param {string|null} dueDate - ISO 8601 date string (YYYY-MM-DD) or null
 * @returns {OverdueStatus|null} Overdue status or null if not overdue
 */
export function calculateOverdueStatus(dueDate) {
  if (!dueDate) return null;
  
  try {
    const now = new Date();
    const due = new Date(dueDate);
    
    // Validate date
    if (isNaN(due.getTime())) {
      return null; // Invalid date treated as no due date
    }
    
    // Normalize to midnight UTC for fair comparison
    now.setUTCHours(0, 0, 0, 0);
    due.setUTCHours(0, 0, 0, 0);
    
    const diffMs = now - due;
    const diffDays = Math.floor(diffMs / (1000 * 60 * 60 * 24));
    
    if (diffDays <= 0) return null; // Not overdue (today or future)
    
    return {
      isOverdue: true,
      daysOverdue: diffDays,
      weeksOverdue: Math.floor(diffDays / 7)
    };
  } catch (error) {
    // Handle any date parsing errors
    console.error('Error calculating overdue status:', error);
    return null;
  }
}

/**
 * Checks if a todo is currently overdue
 * @param {string|null} dueDate - ISO 8601 date string
 * @param {boolean} completed - Whether todo is completed
 * @returns {boolean} True if overdue
 */
export function isOverdue(dueDate, completed) {
  if (completed) return false;
  const status = calculateOverdueStatus(dueDate);
  return status?.isOverdue ?? false;
}

/**
 * Formats overdue duration for display
 * @param {number} daysOverdue - Number of days overdue
 * @returns {string} Formatted string (e.g., "2 days overdue", "1 week overdue")
 */
export function formatOverdueDuration(daysOverdue) {
  if (daysOverdue < 7) {
    return `${daysOverdue} ${daysOverdue === 1 ? 'day' : 'days'} overdue`;
  }
  
  const weeks = Math.floor(daysOverdue / 7);
  return `${weeks} ${weeks === 1 ? 'week' : 'weeks'} overdue`;
}
```

---

### Overdue Count Calculation

**Location**: `packages/frontend/src/App.js` (or custom hook)

```javascript
/**
 * Calculates the total number of overdue todos
 * @param {Array<Todo>} todos - Array of todo items
 * @returns {number} Count of overdue items
 */
function calculateOverdueCount(todos) {
  return todos.filter(todo => 
    !todo.completed && 
    isOverdue(todo.dueDate, todo.completed)
  ).length;
}
```

---

## State Transitions

### Overdue Status Lifecycle

```
[Todo Created] 
    ↓
[Has Due Date?]
    ├─ No  → Never Overdue
    └─ Yes → [Wait Until Due Date]
              ↓
        [Due Date Passes (UTC Midnight)]
              ↓
        [Completed?]
              ├─ Yes → Not Overdue (Completed)
              └─ No  → **OVERDUE** 
                        ↓
                  [User Completes Todo]
                        ↓
                  Not Overdue (Completed)
```

### State Diagram

```
┌─────────────────────┐
│ No Due Date         │ ──────────────────────┐
└─────────────────────┘                       │
                                              │
┌─────────────────────┐                       │
│ Future Due Date     │ ──────────────────────┤
└─────────────────────┘                       │
                                              ├──→ [Never Overdue]
┌─────────────────────┐                       │
│ Due Today           │ ──────────────────────┤
└─────────────────────┘                       │
                                              │
┌─────────────────────┐                       │
│ Completed           │ ──────────────────────┘
└─────────────────────┘

┌─────────────────────┐
│ Past Due Date       │
│ + Not Completed     │ ──────────────────────→ [OVERDUE]
└─────────────────────┘
```

---

## Validation Rules

### Due Date Validation

1. **Format**: ISO 8601 date string (YYYY-MM-DD)
2. **Timezone**: Stored as date-only (no time component), interpreted as UTC midnight
3. **Invalid Dates**: Treated as `null` (no due date, never overdue)
4. **Null Values**: Allowed (todos without due dates)

**Valid Examples**:
- `"2025-12-25"` ✅
- `"2026-01-01"` ✅
- `null` ✅

**Invalid Examples** (treated as null):
- `"12/25/2025"` ❌ (wrong format)
- `"2025-13-45"` ❌ (invalid month/day)
- `"not-a-date"` ❌ (malformed)
- `""` ❌ (empty string)

### Business Rules

1. **Overdue Determination**:
   - Must use UTC date comparison
   - Must normalize to midnight (00:00:00)
   - Comparison: `currentDate > dueDate`

2. **Completion Override**:
   - Completed todos are NEVER overdue
   - Completion status takes precedence over due date

3. **Display Threshold**:
   - Days: 1-6 days overdue
   - Weeks: 7+ days overdue
   - Use floor division for weeks (14 days = 2 weeks)

---

## Data Flow

### Frontend Data Flow

```
1. [Backend API] 
     ↓ (GET /api/todos)
   [Todo Data (with dueDate)]
     ↓
2. [TodoList Component]
     ↓ (map todos)
   [TodoCard Component]
     ↓
3. [calculateOverdueStatus(todo.dueDate)]
     ↓
   [OverdueStatus Object]
     ↓
4. [Conditional Rendering]
     ├─ isOverdue → Apply .overdue CSS class
     ├─ daysOverdue → Display "X days overdue"
     └─ weeksOverdue → Display "X weeks overdue" (if >= 7 days)
```

### Overdue Count Flow

```
1. [App Component receives todos]
     ↓
2. [calculateOverdueCount(todos)]
     ↓
   [Filter: !completed && isOverdue]
     ↓
3. [overdueCount state]
     ↓
4. [Header: "My Todos ({overdueCount} overdue)"]
```

---

## API Contracts

### Existing Endpoints (No Changes Required)

The overdue feature uses existing API endpoints with no modifications:

#### GET /api/todos
Returns all todos with their due dates. Overdue calculation happens client-side.

**Response**:
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
  }
]
```

#### POST /api/todos
Creates a new todo (no changes to schema).

#### PUT /api/todos/:id
Updates a todo (no changes to schema).

#### DELETE /api/todos/:id
Deletes a todo (no changes to schema).

---

## Edge Cases

### 1. Invalid Due Dates

**Scenario**: Todo has malformed due date (e.g., `"invalid-date"`)

**Handling**: Treat as `null` (no due date), never mark as overdue

```javascript
calculateOverdueStatus("invalid-date") // Returns null
```

### 2. Completed Overdue Todos

**Scenario**: Todo was overdue but user completed it

**Handling**: Do not display overdue indicator (completion takes precedence)

```javascript
isOverdue("2025-12-10", true) // Returns false (completed)
```

### 3. Midnight Rollover

**Scenario**: Todo's due date passes while user is viewing the app

**Handling**: Use `setTimeout` to trigger re-render at next UTC midnight

```javascript
useEffect(() => {
  const msUntilMidnight = getMsUntilNextMidnightUTC();
  const timeoutId = setTimeout(() => {
    setRefreshKey(prev => prev + 1); // Force recalculation
  }, msUntilMidnight);
  
  return () => clearTimeout(timeoutId);
}, []);
```

### 4. Due Today vs. Overdue

**Scenario**: Todo is due today (same date as current date)

**Handling**: NOT considered overdue until the next day

```javascript
// Today is 2025-12-15
calculateOverdueStatus("2025-12-15") // Returns null (not overdue)
calculateOverdueStatus("2025-12-14") // Returns { isOverdue: true, daysOverdue: 1 }
```

### 5. Timezone Differences

**Scenario**: User in different timezone views overdue todos

**Handling**: All calculations use UTC midnight; todos become overdue at the same moment worldwide

```javascript
// UTC-based calculation ensures consistency
now.setUTCHours(0, 0, 0, 0);
due.setUTCHours(0, 0, 0, 0);
```

---

## Performance Considerations

### Memoization Strategy

```javascript
// In TodoCard.js - memoize overdue calculation per todo
const overdueStatus = useMemo(
  () => calculateOverdueStatus(todo.dueDate),
  [todo.dueDate] // Only recalculate if dueDate changes
);

// In TodoList.js - memoize TodoCard to prevent unnecessary re-renders
export default React.memo(TodoCard);
```

### Scalability

- **Small Lists (< 100 todos)**: No special optimization needed
- **Large Lists (100-1000 todos)**: Memoization and `React.memo` sufficient
- **Very Large Lists (1000+ todos)**: Consider virtualized list (out of scope)

**Performance Target**: < 5ms per render cycle for date calculations

---

## Testing Strategy

### Unit Tests (dateUtils.test.js)

```javascript
describe('calculateOverdueStatus', () => {
  beforeEach(() => {
    jest.useFakeTimers();
  });
  
  afterEach(() => {
    jest.useRealTimers();
  });
  
  it('returns null for null due dates', () => {
    expect(calculateOverdueStatus(null)).toBeNull();
  });
  
  it('returns null for invalid due dates', () => {
    expect(calculateOverdueStatus('invalid-date')).toBeNull();
  });
  
  it('calculates days overdue correctly', () => {
    jest.setSystemTime(new Date('2025-12-15'));
    const result = calculateOverdueStatus('2025-12-10');
    expect(result).toEqual({
      isOverdue: true,
      daysOverdue: 5,
      weeksOverdue: 0
    });
  });
  
  it('calculates weeks overdue for 7+ days', () => {
    jest.setSystemTime(new Date('2025-12-15'));
    const result = calculateOverdueStatus('2025-12-01');
    expect(result).toEqual({
      isOverdue: true,
      daysOverdue: 14,
      weeksOverdue: 2
    });
  });
  
  it('returns null for future due dates', () => {
    jest.setSystemTime(new Date('2025-12-15'));
    expect(calculateOverdueStatus('2025-12-20')).toBeNull();
  });
  
  it('returns null for due today', () => {
    jest.setSystemTime(new Date('2025-12-15'));
    expect(calculateOverdueStatus('2025-12-15')).toBeNull();
  });
});

describe('isOverdue', () => {
  beforeEach(() => {
    jest.useFakeTimers().setSystemTime(new Date('2025-12-15'));
  });
  
  it('returns false for completed todos', () => {
    expect(isOverdue('2025-12-10', true)).toBe(false);
  });
  
  it('returns true for overdue incomplete todos', () => {
    expect(isOverdue('2025-12-10', false)).toBe(true);
  });
});
```

---

## Summary

### Data Changes
- ✅ **No database schema changes required**
- ✅ **No new API endpoints needed**
- ✅ **Computed properties only** (calculated on-demand)

### Key Components
1. **dateUtils.js**: Pure utility functions for overdue calculation
2. **TodoCard.js**: Displays overdue status with memoization
3. **App.js**: Calculates and displays overdue count

### Business Rules
- UTC-based date comparison
- Completed todos never overdue
- Invalid dates treated as null
- Days for 1-6, weeks for 7+
- Midnight rollover via setTimeout
