# Feature Specification: Support for Overdue Todo Items

**Feature Branch**: `001-overdue-todos`  
**Created**: December 12, 2025  
**Status**: Draft  
**Input**: User description: "Support for Overdue Todo Items - Users need a clear, visual way to identify which todos have not been completed by their due date."

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Visual Identification of Overdue Items (Priority: P1)

As a user viewing my todo list, I need to immediately distinguish overdue items from current or future tasks through clear visual indicators, so I can quickly identify tasks that require urgent attention.

**Why this priority**: This is the core value proposition of the feature. Without visual distinction, users must manually calculate whether each task is overdue, defeating the purpose of the feature.

**Independent Test**: Can be fully tested by creating todos with past due dates and verifying they display distinct visual indicators. Delivers immediate value by making overdue status obvious at a glance.

**Acceptance Scenarios**:

1. **Given** a todo with a due date in the past that is not completed, **When** I view the todo list, **Then** the overdue todo displays with a distinct visual indicator (color, icon, or styling)
2. **Given** multiple todos with various due dates (past, today, future), **When** I view the todo list, **Then** only todos with past due dates that are not completed show the overdue indicator
3. **Given** a todo that was overdue, **When** I mark it as completed, **Then** the overdue indicator is removed or replaced with a completed status indicator
4. **Given** a todo with a due date of today, **When** viewing before midnight on that date, **Then** the todo does NOT show as overdue
5. **Given** a todo with no due date, **When** I view the todo list, **Then** the todo never displays with an overdue indicator

---

### User Story 2 - Clear Overdue Status Context (Priority: P2)

As a user, I want to understand how overdue each task is (days or weeks past due), so I can better prioritize among multiple overdue items.

**Why this priority**: Enhances the basic visual indicator by providing actionable prioritization information. Users with multiple overdue tasks need to know which are most urgent.

**Independent Test**: Can be tested by creating todos with different past due dates and verifying each displays the correct time elapsed since due date. Delivers value by helping users triage overdue work.

**Acceptance Scenarios**:

1. **Given** an overdue todo, **When** I view the todo item, **Then** I see text indicating how many days overdue it is (e.g., "2 days overdue", "1 week overdue")
2. **Given** a todo that became overdue today, **When** I view the todo item, **Then** it shows as "Due today" or "Overdue today"
3. **Given** a todo overdue by less than 24 hours, **When** I view the todo item, **Then** it displays in days rather than hours
4. **Given** a todo overdue by more than 7 days, **When** I view the todo item, **Then** it may optionally display in weeks (e.g., "2 weeks overdue")

---

### User Story 3 - Overdue Count Summary (Priority: P3)

As a user, I want to see a summary count of how many overdue tasks I have, so I can quickly assess my workload without scanning the entire list.

**Why this priority**: Nice-to-have feature that provides overview information. Less critical than the core visual indicators but improves user experience.

**Independent Test**: Can be tested by creating multiple overdue todos and verifying the count updates accurately. Delivers value as a dashboard-style metric.

**Acceptance Scenarios**:

1. **Given** I have 3 uncompleted todos with past due dates, **When** I view the todo list header or summary area, **Then** I see "3 overdue tasks" or similar count display
2. **Given** I complete an overdue todo, **When** the list refreshes, **Then** the overdue count decreases by 1
3. **Given** I have no overdue todos, **When** I view the summary area, **Then** the overdue count shows "0" or is hidden
4. **Given** a todo's due date passes while I'm viewing the list, **When** the date changes to the next day, **Then** the overdue count updates to reflect the newly overdue item

---

### Edge Cases

- What happens when a todo's due date is exactly at midnight (start or end of day)?
- How does the system handle timezone differences if viewing from different locations?
- What happens when viewing a todo that becomes overdue while the page is open (date rolls over)?
- How are todos with invalid or malformed due dates handled?
- What happens if the system clock is incorrect or changes (daylight saving time)?

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: System MUST display a distinct visual indicator for incomplete todos with due dates in the past
- **FR-002**: System MUST determine overdue status by comparing the todo's due date against the current date
- **FR-003**: System MUST NOT mark completed todos as overdue, regardless of due date
- **FR-004**: System MUST NOT mark todos without due dates as overdue
- **FR-005**: System MUST update overdue status dynamically as dates change (todos can become overdue while app is open)
- **FR-006**: System MUST display the number of days overdue for each overdue todo
- **FR-007**: System MUST calculate overdue duration by comparing due date to current date
- **FR-008**: System MUST display an aggregate count of total overdue items
- **FR-009**: System MUST update the overdue count when todos are completed or deleted
- **FR-010**: System MUST treat "due today" as not overdue until the date rolls to the next day

### Key Entities *(include if feature involves data)*

- **Todo Item**: Extended with overdue status calculation logic
  - Existing attributes: id, title, dueDate, completed, createdAt
  - Computed properties: isOverdue (boolean), daysOverdue (number)
  - Rules: isOverdue = true when (dueDate < currentDate AND completed = false AND dueDate is not null)

- **Overdue Status**: Computed property, not stored
  - Calculated on-demand based on current date and todo's due date
  - Updated automatically as time passes

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Users can identify overdue todos within 2 seconds of viewing the list, without reading due dates
- **SC-002**: 90% of users correctly identify overdue items in usability testing on first attempt
- **SC-003**: Overdue status updates reflect current date within 60 seconds of date change (midnight rollover)
- **SC-004**: Users can distinguish between todos due today, overdue, and future due dates at a glance
- **SC-005**: System accurately calculates days overdue for todos up to 365 days past due
- **SC-006**: Overdue count reflects accurate total with 100% accuracy (matches manual count)

## Scope *(mandatory)*

### In Scope

- Visual indicators (color, icons, styling) for overdue todos
- Text display showing days/weeks overdue
- Aggregate count of overdue items
- Real-time calculation of overdue status based on current date
- Treatment of completed todos (not marked as overdue)

### Out of Scope

- Notifications or reminders about overdue items
- Filtering or sorting by overdue status
- Automatic prioritization or reordering of overdue items
- Batch operations on overdue todos
- Historical tracking of how long items were overdue before completion
- User-configurable thresholds for what counts as "overdue"
- Snoozing or postponing overdue items
- Calendar integration or due date suggestions

## Dependencies & Assumptions *(mandatory)*

### Dependencies

- Existing todo data structure must include `dueDate`, `completed`, and `id` fields
- Frontend must have access to accurate current date/time
- Todo list display must support visual styling customization

### Assumptions

- The application already has a todo list with due dates implemented
- Todos use a standard date format (ISO 8601 or similar) for due dates
- The system operates in a single timezone (user's local time)
- The browser/device clock is reasonably accurate
- Users understand standard date representations (days, weeks)
- The application already persists todo completion status

## Non-Functional Considerations *(optional)*

### Performance

- Overdue status calculation should not add noticeable delay to list rendering
- Overdue count should update without requiring full page refresh

### Accessibility

- Visual indicators must not rely solely on color (use icons or text in addition to color)
- Overdue status information must be available to screen readers
- Color contrast ratios must meet WCAG AA standards for overdue indicators

### Usability

- Overdue indicators should be consistent with the application's existing design language
- The visual design should clearly communicate urgency without being alarming
- Users should not need training to understand overdue indicators
