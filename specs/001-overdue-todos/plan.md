# Implementation Plan: Support for Overdue Todo Items

**Branch**: `001-overdue-todos` | **Date**: December 12, 2025 | **Spec**: [spec.md](spec.md)
**Input**: Feature specification from `/specs/001-overdue-todos/spec.md`

**Note**: This template is filled in by the `/speckit.plan` command. See `.specify/templates/commands/plan.md` for the execution workflow.

## Summary

Add visual identification and tracking for overdue todo items. Users need a clear way to identify which todos have not been completed by their due date. The feature includes:
- Visual indicators (color + icon) for overdue todos
- Display of days/weeks overdue
- Aggregate count of overdue items in the header
- Real-time calculation based on UTC dates

## Technical Context

**Language/Version**: JavaScript (Node.js v18+, React 18.2.0)  
**Primary Dependencies**: React, React DOM, Express.js, better-sqlite3, axios  
**Storage**: SQLite database (via better-sqlite3) for todo persistence  
**Testing**: Jest + @testing-library/react (frontend), Jest + supertest (backend)  
**Target Platform**: Desktop web application (monorepo with npm workspaces)
**Project Type**: Web (React frontend + Express backend)  
**Performance Goals**: NEEDS CLARIFICATION (likely sub-100ms date calculations, instant UI updates)  
**Constraints**: Desktop-focused, single-user, UTC-based date calculations, 8px grid system UI  
**Scale/Scope**: Small application, single user, ~5-10 components affected

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

Verify feature compliance with `.specify/memory/constitution.md` principles:

- [x] **Single Responsibility & SOLID**: Components have single responsibility, follow SOLID
  - TodoCard displays overdue status (presentation only)
  - Utility functions handle date calculations (business logic)
  - Services manage data access (separation of concerns)
  - No violations detected
  
- [x] **Code Quality (DRY, KISS)**: No duplication planned, simple approach preferred
  - Date calculation logic will be centralized in a utility module
  - Overdue visual styling will use shared CSS classes
  - Simple UTC date comparison approach
  - No violations detected
  
- [x] **Comprehensive Testing**: 80%+ test coverage planned, TDD approach defined
  - Unit tests for date calculation utilities
  - Unit tests for overdue visual components
  - Integration tests for overdue count updates
  - Edge case tests (midnight rollover, invalid dates, completed todos)
  - TDD approach: write tests first for date logic and component rendering
  - No violations detected
  
- [x] **Error Handling**: All failure points have graceful error handling
  - Invalid/malformed dates treated as null (never overdue, no error shown per spec)
  - Date calculation failures default to non-overdue state
  - No network calls involved (all client-side calculation)
  - No violations detected
  
- [x] **Code Style & Organization**: Follows naming conventions, import order, ESLint rules
  - Follow existing project conventions (camelCase, PascalCase for components)
  - Tests in __tests__/ directories colocated with source
  - ESLint rules will be enforced
  - No violations detected
  
- [x] **User-Centered Design**: Accessible, Material Design compliant, 8px grid system
  - Color + icon for overdue (not color alone - WCAG compliant)
  - Warning icon (⚠️) with text alternative for screen readers
  - 8px grid system for spacing (using existing theme variables)
  - Material Design card elevation for visual hierarchy
  - No violations detected

**Violations (if any)**: None - feature aligns with all constitution principles

## Project Structure

### Documentation (this feature)

```text
specs/001-overdue-todos/
├── plan.md              # This file (/speckit.plan command output)
├── research.md          # Phase 0 output (/speckit.plan command)
├── data-model.md        # Phase 1 output (/speckit.plan command)
├── quickstart.md        # Phase 1 output (/speckit.plan command)
├── contracts/           # Phase 1 output (/speckit.plan command)
│   └── overdue-api.yaml # API contract for overdue calculations
└── tasks.md             # Phase 2 output (/speckit.tasks command - NOT created by /speckit.plan)
```

### Source Code (repository root)

```text
packages/
├── backend/
│   ├── src/
│   │   ├── services/
│   │   │   └── todoService.js          # [EXISTING] May need date query methods
│   │   └── app.js                      # [EXISTING] Express app
│   └── __tests__/
│       └── app.test.js                 # [EXTEND] Add tests for date queries
│
└── frontend/
    ├── src/
    │   ├── components/
    │   │   ├── TodoCard.js             # [EXTEND] Add overdue visual indicators
    │   │   ├── TodoList.js             # [EXTEND] Add overdue count to header
    │   │   └── __tests__/
    │   │       ├── TodoCard.test.js    # [EXTEND] Test overdue rendering
    │   │       └── TodoList.test.js    # [EXTEND] Test overdue count
    │   │
    │   ├── utils/                      # [NEW] Create utility directory
    │   │   ├── dateUtils.js            # [NEW] Overdue calculation logic
    │   │   └── __tests__/
    │   │       └── dateUtils.test.js   # [NEW] Date calculation tests
    │   │
    │   ├── styles/
    │   │   └── theme.css               # [EXTEND] Add overdue color/icon styles
    │   │
    │   └── App.js                      # [EXISTING] No changes needed
    │
    └── __tests__/
        └── App.test.js                 # [EXTEND] Integration test for overdue feature
```

**Structure Decision**: Web application structure with React frontend and Express backend.
- **Frontend changes**: New utility module for date calculations, extend TodoCard and TodoList components
- **Backend changes**: Minimal - may add helper methods to todoService.js if needed for queries
- **New files**: `utils/dateUtils.js` for centralized date logic
- **Testing**: Colocated tests in `__tests__/` directories following existing pattern

## Complexity Tracking

> **Fill ONLY if Constitution Check has violations that must be justified**

No violations detected. This feature aligns with all constitution principles.
