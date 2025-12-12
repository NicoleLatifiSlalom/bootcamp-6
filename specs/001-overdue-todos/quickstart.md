# Quickstart Guide: Overdue Todo Items Feature

**Feature**: Support for Overdue Todo Items  
**Branch**: `001-overdue-todos`  
**Date**: December 12, 2025

## Overview

This guide provides a high-level roadmap for implementing the overdue todo feature. It serves as a reference for developers working on the implementation tasks.

---

## Prerequisites

- Node.js v18+ installed
- Workspace cloned and dependencies installed (`npm install`)
- Familiarity with React, Express.js, and Jest
- Understanding of the project constitution (`.specify/memory/constitution.md`)

---

## Feature Summary

Add visual indicators and tracking for overdue todo items:
- **Visual indicators**: Color + warning icon (⚠️) for overdue todos
- **Overdue duration**: Display "X days overdue" or "X weeks overdue"
- **Count badge**: Show total overdue count in header (e.g., "My Todos (3 overdue)")
- **Real-time updates**: Automatically update status at midnight UTC
- **Accessibility**: WCAG AA compliant with multi-modal indicators

---

## Implementation Checklist

### Phase 1: Utility Functions (Foundation)

**Location**: `packages/frontend/src/utils/dateUtils.js` (NEW)

- [ ] Create `dateUtils.js` utility module
- [ ] Implement `calculateOverdueStatus(dueDate)` function
  - Accepts ISO 8601 date string (YYYY-MM-DD)
  - Returns `{ isOverdue, daysOverdue, weeksOverdue }` or `null`
  - Uses UTC date normalization
  - Handles invalid dates gracefully
- [ ] Implement `isOverdue(dueDate, completed)` helper
- [ ] Implement `formatOverdueDuration(daysOverdue)` formatter
- [ ] Write unit tests in `utils/__tests__/dateUtils.test.js`
  - Test null/invalid dates
  - Test days calculation (1-6 days)
  - Test weeks calculation (7+ days)
  - Test "due today" edge case
  - Use `jest.useFakeTimers()` for deterministic tests

**Expected Time**: 2-3 hours

---

### Phase 2: Visual Styling (Theme)

**Locations**: 
- `packages/frontend/src/styles/theme.css` (EXTEND)
- `packages/frontend/src/App.css` (EXTEND)

- [ ] Add warning color tokens to `theme.css`
  - Light mode: `--warning-color: #d97706`
  - Dark mode: `--warning-color: #fbbf24`
  - Background tints: `--warning-light`
- [ ] Add overdue card styles to `App.css`
  - `.todo-card.overdue` with left border + background tint
  - `.todo-title.overdue` with warning color
  - `.overdue-badge` styles (icon + text)
  - `.overdue-badge-header` for header count
- [ ] Verify 8px grid alignment for spacing
- [ ] Test contrast ratios (Chrome DevTools)
  - Light mode: min 4.5:1
  - Dark mode: min 4.5:1

**Expected Time**: 1-2 hours

---

### Phase 3: Component Updates

#### 3.1 Update TodoCard Component

**Location**: `packages/frontend/src/components/TodoCard.js` (EXTEND)

- [ ] Import `calculateOverdueStatus` from `utils/dateUtils`
- [ ] Use `useMemo` to calculate overdue status
  ```javascript
  const overdueStatus = useMemo(
    () => calculateOverdueStatus(todo.dueDate),
    [todo.dueDate]
  );
  ```
- [ ] Add conditional `.overdue` class to card container
- [ ] Add overdue indicator with icon and text
  ```jsx
  {overdueStatus && !todo.completed && (
    <span className="overdue-badge" role="status">
      <span aria-hidden="true">⚠️</span>
      {formatOverdueDuration(overdueStatus.daysOverdue)}
    </span>
  )}
  ```
- [ ] Add `aria-label` for screen readers
- [ ] Update tests in `components/__tests__/TodoCard.test.js`
  - Test overdue visual indicator renders
  - Test completed todos don't show overdue
  - Test days vs. weeks display

**Expected Time**: 2-3 hours

---

#### 3.2 Update TodoList/App Component

**Location**: `packages/frontend/src/App.js` (EXTEND)

- [ ] Import `isOverdue` helper from `utils/dateUtils`
- [ ] Calculate overdue count
  ```javascript
  const overdueCount = todos.filter(todo => 
    !todo.completed && isOverdue(todo.dueDate, todo.completed)
  ).length;
  ```
- [ ] Add overdue count to header
  ```jsx
  <h1 className="app-title">
    <span className="app-icon">🎃</span>
    My Todos
    {overdueCount > 0 && (
      <span className="overdue-badge-header" role="status" aria-live="polite">
        ({overdueCount} overdue)
      </span>
    )}
  </h1>
  ```
- [ ] Update tests in `__tests__/App.test.js`
  - Test overdue count displays correctly
  - Test count updates when todos completed/deleted

**Expected Time**: 1-2 hours

---

### Phase 4: Midnight Rollover (Advanced)

**Location**: `packages/frontend/src/App.js` or custom hook (EXTEND)

- [ ] Implement `useEffect` to schedule midnight updates
  ```javascript
  useEffect(() => {
    function scheduleNextMidnight() {
      const now = new Date();
      const tomorrow = new Date(now);
      tomorrow.setUTCHours(24, 0, 0, 0);
      
      const msUntilMidnight = tomorrow - now;
      
      return setTimeout(() => {
        setRefreshKey(prev => prev + 1); // Trigger re-render
        scheduleNextMidnight();
      }, msUntilMidnight);
    }
    
    const timeoutId = scheduleNextMidnight();
    return () => clearTimeout(timeoutId);
  }, []);
  ```
- [ ] Add `refreshKey` state to force component updates
- [ ] Test manually by mocking system time

**Expected Time**: 1-2 hours

---

### Phase 5: Testing & Accessibility

- [ ] Run all unit tests: `npm test --workspace=frontend`
- [ ] Verify 80%+ code coverage
- [ ] Test with screen reader (NVDA or VoiceOver)
  - Verify overdue status announced
  - Verify count updates announced with `aria-live="polite"`
- [ ] Test color vision deficiencies (Chrome DevTools)
  - Protanopia, Deuteranopia, Tritanopia
  - Verify border + icon + text provide redundancy
- [ ] Test keyboard navigation
  - Tab through overdue todos
  - Verify focus indicators visible
- [ ] Test theme switching
  - Verify colors update in dark mode
  - Verify contrast maintained
- [ ] Manual testing scenarios
  - Create todo with past due date → Verify overdue indicator
  - Complete overdue todo → Verify indicator removed
  - Create todo with future date → Verify not overdue
  - Create todo without date → Verify never overdue

**Expected Time**: 2-3 hours

---

## File Changes Summary

### New Files
- `packages/frontend/src/utils/dateUtils.js`
- `packages/frontend/src/utils/__tests__/dateUtils.test.js`

### Modified Files
- `packages/frontend/src/styles/theme.css` (add warning colors)
- `packages/frontend/src/App.css` (add overdue styles)
- `packages/frontend/src/components/TodoCard.js` (add overdue indicator)
- `packages/frontend/src/components/__tests__/TodoCard.test.js` (extend tests)
- `packages/frontend/src/App.js` (add overdue count to header)
- `packages/frontend/src/__tests__/App.test.js` (extend tests)

### No Changes
- Backend files (no API changes)
- Database schema (no migrations)

---

## Development Workflow

### 1. Setup Branch
```bash
git checkout -b 001-overdue-todos
```

### 2. Run Development Environment
```bash
npm run start
# Opens frontend at http://localhost:3000
# Backend runs at http://localhost:3030
```

### 3. Development Loop
```bash
# Make changes
# Run tests continuously
npm run test:watch --workspace=frontend

# Check code coverage
npm test --workspace=frontend -- --coverage
```

### 4. Pre-Commit Checklist
- [ ] All tests pass
- [ ] No ESLint errors
- [ ] Code follows naming conventions
- [ ] Comments explain "why" not "what"
- [ ] No console.log statements

### 5. Commit & Push
```bash
git add .
git commit -m "feat: add overdue todo indicators

- Add dateUtils for overdue calculation
- Update TodoCard with visual indicators
- Add overdue count to header
- Implement midnight rollover updates
- Add comprehensive test coverage

Closes #001-overdue-todos"

git push origin 001-overdue-todos
```

---

## Testing Commands

```bash
# Run all tests
npm test

# Run frontend tests only
npm test --workspace=frontend

# Run tests in watch mode (for development)
npm run test:watch --workspace=frontend

# Run tests with coverage
npm test --workspace=frontend -- --coverage

# Run specific test file
npm test --workspace=frontend -- dateUtils.test.js
```

---

## Key Design Decisions

### 1. Client-Side Calculation
**Why**: Performance (< 1ms per todo), no backend changes needed, single-user app

### 2. Native Date API
**Why**: No dependencies, sufficient for UTC comparisons, fast execution

### 3. Unicode Emoji (⚠️)
**Why**: Zero dependencies, consistent with existing UI (🎃), accessible with aria-hidden

### 4. CSS Classes (not inline styles)
**Why**: Theme-aware, automatic dark mode, better performance, easier testing

### 5. useMemo + React.memo
**Why**: Prevent unnecessary recalculations and re-renders, < 5ms target

---

## Performance Targets

- **Date calculation**: < 1ms per todo
- **Component render**: < 5ms per update
- **Midnight rollover**: < 20ms batch update
- **Coverage**: 80%+ for all new code

---

## Accessibility Checklist

- [x] Multi-modal indicators (color + icon + border + text)
- [x] WCAG AA contrast ratios (4.5:1 minimum)
- [x] aria-label on overdue cards
- [x] role="status" on overdue badges
- [x] aria-live="polite" on count updates
- [x] aria-hidden="true" on decorative icons
- [x] Screen reader tested
- [x] Keyboard navigation works

---

## Common Pitfalls

### 1. Forgetting UTC Normalization
```javascript
// ❌ Wrong - compares with current time
const now = new Date();

// ✅ Correct - normalizes to midnight UTC
now.setUTCHours(0, 0, 0, 0);
```

### 2. Not Memoizing Calculations
```javascript
// ❌ Wrong - recalculates on every render
const overdueStatus = calculateOverdueStatus(todo.dueDate);

// ✅ Correct - memoizes based on dueDate
const overdueStatus = useMemo(
  () => calculateOverdueStatus(todo.dueDate),
  [todo.dueDate]
);
```

### 3. Showing Overdue for Completed Todos
```javascript
// ❌ Wrong - shows overdue even when completed
{overdueStatus && <OverdueIndicator />}

// ✅ Correct - checks completion status
{overdueStatus && !todo.completed && <OverdueIndicator />}
```

### 4. Invalid Date Handling
```javascript
// ❌ Wrong - throws error
const due = new Date(dueDate);

// ✅ Correct - validates and handles gracefully
try {
  const due = new Date(dueDate);
  if (isNaN(due.getTime())) return null;
} catch {
  return null;
}
```

---

## Resources

- **Feature Spec**: [spec.md](spec.md)
- **Research**: [research.md](research.md)
- **Data Model**: [data-model.md](data-model.md)
- **API Contracts**: [contracts/overdue-api.md](contracts/overdue-api.md)
- **Constitution**: `.specify/memory/constitution.md`
- **Coding Guidelines**: `docs/coding-guidelines.md`

---

## Next Steps

After reading this quickstart:
1. Review the feature spec and research documents
2. Set up your development branch
3. Start with Phase 1 (utility functions + tests)
4. Work through phases sequentially
5. Run tests continuously during development
6. Follow the constitution checklist before committing

**Estimated Total Time**: 10-15 hours (including testing and polish)

---

## Questions?

- Check the research.md for technical decisions
- Review data-model.md for calculation logic
- Consult coding-guidelines.md for style questions
- Follow constitution principles for any ambiguity
