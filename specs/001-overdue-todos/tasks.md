# Tasks: Support for Overdue Todo Items

**Feature Branch**: `001-overdue-todos`  
**Input**: Design documents from `/specs/001-overdue-todos/`  
**Prerequisites**: plan.md ✅, spec.md ✅, research.md ✅, data-model.md ✅

**Tests**: Not explicitly requested in specification - tasks focus on implementation only

**Organization**: Tasks are grouped by user story to enable independent implementation and testing of each story.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (e.g., US1, US2, US3)
- Include exact file paths in descriptions

## Path Conventions

- Web app structure: `packages/backend/src/`, `packages/frontend/src/`
- Tests colocated in `__tests__/` directories
- Utilities in `packages/frontend/src/utils/`
- Components in `packages/frontend/src/components/`
- Styles in `packages/frontend/src/styles/`

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Project initialization and basic structure

- [ ] T001 Create utility directory structure at packages/frontend/src/utils/
- [ ] T002 [P] Create test directory at packages/frontend/src/utils/__tests__/

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Core date calculation utilities that ALL user stories depend on

**⚠️ CRITICAL**: No user story work can begin until this phase is complete

- [ ] T003 [P] Implement calculateOverdueStatus function in packages/frontend/src/utils/dateUtils.js
- [ ] T004 [P] Implement isOverdue helper function in packages/frontend/src/utils/dateUtils.js
- [ ] T005 [P] Implement formatOverdueDuration function in packages/frontend/src/utils/dateUtils.js
- [ ] T006 [P] Add unit tests for calculateOverdueStatus in packages/frontend/src/utils/__tests__/dateUtils.test.js
- [ ] T007 [P] Add unit tests for edge cases (invalid dates, midnight UTC, completed todos) in packages/frontend/src/utils/__tests__/dateUtils.test.js
- [ ] T008 Add overdue CSS custom properties to packages/frontend/src/styles/theme.css for light/dark mode
- [ ] T009 [P] Add overdue visual styling classes to packages/frontend/src/App.css

**Checkpoint**: Foundation ready - utility functions tested and working, styles defined

---

## Phase 3: User Story 1 - Visual Identification of Overdue Items (Priority: P1) 🎯 MVP

**Goal**: Users can immediately distinguish overdue items from current or future tasks through clear visual indicators (color + icon)

**Independent Test**: Create todos with past due dates and verify they display with distinct visual indicators (red/orange color, warning icon, border styling). Mark one as completed and verify indicator is removed.

### Implementation for User Story 1

- [ ] T010 [US1] Import calculateOverdueStatus in packages/frontend/src/components/TodoCard.js
- [ ] T011 [US1] Add useMemo hook for overdue calculation in packages/frontend/src/components/TodoCard.js
- [ ] T012 [US1] Add conditional CSS class 'overdue' to card container in packages/frontend/src/components/TodoCard.js
- [ ] T013 [US1] Add conditional CSS class 'overdue' to title element in packages/frontend/src/components/TodoCard.js
- [ ] T014 [US1] Add warning icon (⚠️) with aria-hidden to overdue indicator in packages/frontend/src/components/TodoCard.js
- [ ] T015 [US1] Wrap TodoCard component with React.memo for performance in packages/frontend/src/components/TodoCard.js
- [ ] T016 [P] [US1] Update TodoCard component tests to verify overdue styling in packages/frontend/src/components/__tests__/TodoCard.test.js

**Checkpoint**: User Story 1 complete - overdue todos display with distinct visual indicators (color + icon + border)

---

## Phase 4: User Story 2 - Clear Overdue Status Context (Priority: P2)

**Goal**: Users understand how overdue each task is (days or weeks past due) to better prioritize among multiple overdue items

**Independent Test**: Create todos with different past due dates (1 day, 3 days, 7 days, 14 days) and verify each displays the correct overdue duration text (e.g., "2 days overdue", "1 week overdue"). Verify singular/plural forms are correct.

### Implementation for User Story 2

- [ ] T017 [US2] Import formatOverdueDuration in packages/frontend/src/components/TodoCard.js
- [ ] T018 [US2] Add overdue duration badge element in packages/frontend/src/components/TodoCard.js
- [ ] T019 [US2] Display formatted overdue text using formatOverdueDuration in packages/frontend/src/components/TodoCard.js
- [ ] T020 [US2] Add CSS styling for overdue-badge class in packages/frontend/src/App.css
- [ ] T021 [P] [US2] Update TodoCard tests to verify overdue duration display in packages/frontend/src/components/__tests__/TodoCard.test.js
- [ ] T022 [P] [US2] Add test cases for singular/plural forms in packages/frontend/src/components/__tests__/TodoCard.test.js

**Checkpoint**: User Stories 1 AND 2 complete - overdue todos show visual indicator AND time elapsed

---

## Phase 5: User Story 3 - Overdue Count Summary (Priority: P3)

**Goal**: Users see a summary count of how many overdue tasks they have in the header for quick workload assessment

**Independent Test**: Create 3 overdue todos and verify header shows "My Todos (3 overdue)". Complete one overdue todo and verify count decreases to 2. Create scenario with 0 overdue todos and verify count is hidden or shows 0.

### Implementation for User Story 3

- [ ] T023 [US3] Import isOverdue helper in packages/frontend/src/App.js
- [ ] T024 [US3] Implement calculateOverdueCount function in packages/frontend/src/App.js
- [ ] T025 [US3] Add useMemo hook for overdue count calculation in packages/frontend/src/App.js
- [ ] T026 [US3] Update header JSX to display overdue count in packages/frontend/src/App.js
- [ ] T027 [US3] Add conditional rendering to hide count when zero in packages/frontend/src/App.js
- [ ] T028 [US3] Add CSS styling for overdue count in header in packages/frontend/src/App.css
- [ ] T029 [P] [US3] Update App integration tests to verify overdue count updates in packages/frontend/src/__tests__/App.test.js

**Checkpoint**: All user stories (1, 2, 3) complete - full overdue feature functional

---

## Phase 6: Real-Time Updates (Cross-Story Enhancement)

**Goal**: Overdue status updates automatically when date changes (midnight UTC rollover)

**Independent Test**: Create a todo due today, wait until midnight UTC (or manually advance system clock), verify todo becomes overdue without page refresh

### Implementation for Real-Time Updates

- [ ] T030 Implement scheduleNextMidnight function in packages/frontend/src/App.js
- [ ] T031 Add useEffect hook for midnight rollover handler in packages/frontend/src/App.js
- [ ] T032 Add refreshKey state for triggering re-renders on date change in packages/frontend/src/App.js
- [ ] T033 Pass refreshKey as dependency to overdue calculations in packages/frontend/src/App.js
- [ ] T034 [P] Add tests for midnight rollover behavior in packages/frontend/src/__tests__/App.test.js

**Checkpoint**: Real-time updates working - overdue status automatically updates at midnight UTC

---

## Phase 7: Polish & Cross-Cutting Concerns

**Purpose**: Improvements that affect multiple user stories

- [ ] T035 [P] Add accessibility aria-labels for warning icons in packages/frontend/src/components/TodoCard.js
- [ ] T036 [P] Verify WCAG AA color contrast for overdue indicators in both light/dark modes
- [ ] T037 Test invalid date handling (malformed dates treated as null, no errors shown)
- [ ] T038 Test completed todo handling (completed todos never show as overdue)
- [ ] T039 Performance testing with 50+ todos (verify < 5ms render time for overdue calculations)
- [ ] T040 [P] Update project documentation if needed in docs/
- [ ] T041 Code review and cleanup across all modified files

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies - can start immediately
- **Foundational (Phase 2)**: Depends on Setup completion - BLOCKS all user stories
- **User Stories (Phase 3, 4, 5)**: All depend on Foundational phase completion
  - User stories can then proceed in parallel (if staffed)
  - Or sequentially in priority order (P1 → P2 → P3)
- **Real-Time Updates (Phase 6)**: Depends on User Story 3 completion (needs overdue count to update)
- **Polish (Phase 7)**: Depends on all user stories being complete

### User Story Dependencies

- **User Story 1 (P1)**: Can start after Foundational (Phase 2) - No dependencies on other stories
- **User Story 2 (P2)**: Extends User Story 1 - Can start after US1 T010-T015 complete (needs overdue status calculation in TodoCard)
- **User Story 3 (P3)**: Can start after Foundational (Phase 2) - Independent of US1/US2 (works on App.js, not TodoCard.js)

### Within Each User Story

- **User Story 1**: T010-T015 must be sequential (building up TodoCard), T016 can run after T015
- **User Story 2**: Extends User Story 1, T017-T020 sequential, T021-T022 parallel after T020
- **User Story 3**: T023-T028 sequential (building up App.js), T029 after T028

### Parallel Opportunities

- **Phase 1**: T001-T002 can run in parallel
- **Phase 2**: T003-T005 can run in parallel (different functions in same file), T006-T007 can run in parallel (test file), T008-T009 can run in parallel (different CSS files)
- **User Story 1**: T016 can be done independently after T015
- **User Story 2**: T021-T022 can run in parallel
- **User Story 3**: T029 can be done independently after T028
- **Phase 6**: T034 can be done independently after T033
- **Phase 7**: T035-T036 and T040 can run in parallel

---

## Parallel Example: Foundational Phase

```bash
# Launch all date utility functions together:
Task: "Implement calculateOverdueStatus function in packages/frontend/src/utils/dateUtils.js"
Task: "Implement isOverdue helper function in packages/frontend/src/utils/dateUtils.js"
Task: "Implement formatOverdueDuration function in packages/frontend/src/utils/dateUtils.js"

# Launch all test files together:
Task: "Add unit tests for calculateOverdueStatus in packages/frontend/src/utils/__tests__/dateUtils.test.js"
Task: "Add unit tests for edge cases in packages/frontend/src/utils/__tests__/dateUtils.test.js"

# Launch all styling tasks together:
Task: "Add overdue CSS custom properties to packages/frontend/src/styles/theme.css"
Task: "Add overdue visual styling classes to packages/frontend/src/App.css"
```

---

## Parallel Example: User Story 1

```bash
# After T015 completes, launch test task independently:
Task: "Update TodoCard component tests to verify overdue styling in packages/frontend/src/components/__tests__/TodoCard.test.js"
```

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete Phase 1: Setup (T001-T002) - 2 tasks
2. Complete Phase 2: Foundational (T003-T009) - 7 tasks (CRITICAL - blocks all stories)
3. Complete Phase 3: User Story 1 (T010-T016) - 7 tasks
4. **STOP and VALIDATE**: Test User Story 1 independently
   - Create todos with past due dates
   - Verify visual indicators (color + icon + border)
   - Mark as completed and verify indicator removed
5. Deploy/demo if ready - **16 tasks total for MVP**

### Incremental Delivery

1. Complete Setup + Foundational → Foundation ready (9 tasks)
2. Add User Story 1 → Test independently → Deploy/Demo (16 tasks - MVP! ✅)
3. Add User Story 2 → Test independently → Deploy/Demo (22 tasks - enhanced!)
4. Add User Story 3 → Test independently → Deploy/Demo (29 tasks - full feature!)
5. Add Real-Time Updates → Test independently → Deploy/Demo (34 tasks - complete!)
6. Add Polish → Final deployment (41 tasks - production ready!)

### Parallel Team Strategy

With multiple developers:

1. Team completes Setup + Foundational together (9 tasks)
2. Once Foundational is done:
   - Developer A: User Story 1 (T010-T016)
   - Developer B: User Story 3 (T023-T029) - Independent of US1!
   - Developer C: User Story 2 (waits for US1 T015, then T017-T022)
3. Stories complete and integrate independently

---

## Summary Statistics

- **Total Tasks**: 41
- **Setup Phase**: 2 tasks
- **Foundational Phase**: 7 tasks (BLOCKING)
- **User Story 1 (P1)**: 7 tasks - MVP
- **User Story 2 (P2)**: 6 tasks
- **User Story 3 (P3)**: 7 tasks
- **Real-Time Updates**: 5 tasks
- **Polish Phase**: 7 tasks
- **Parallel Tasks**: 17 tasks marked [P] can run concurrently
- **MVP Scope**: 16 tasks (Setup + Foundational + US1)
- **Files Affected**: 
  - 2 new files (dateUtils.js, dateUtils.test.js)
  - 6 modified files (TodoCard.js, TodoCard.test.js, App.js, App.test.js, theme.css, App.css)

---

## Notes

- [P] tasks = different files, no dependencies - safe to parallelize
- [Story] label maps task to specific user story for traceability
- Each user story is independently completable and testable
- Tests are integrated into implementation (not separate TDD phase) since not explicitly requested
- Commit after each task or logical group
- Stop at any checkpoint to validate story independently
- US2 extends US1 (same TodoCard.js file) so must wait for US1 core to complete
- US3 is independent (different file: App.js) so can run parallel to US1
- No backend changes required - all calculations client-side
