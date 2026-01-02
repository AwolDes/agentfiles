# Task Summary Command

Create a comprehensive markdown summary of the completed task in `.claude/tasks_completed/` directory, which exists.

## Instructions

1. Create a new file named with today's date and task description: `.claude/tasks_completed/YYYY-MM-DD-task-name.md`

2. Include the following sections in the summary:

### Required Sections:
- **Title**: Clear task name
- **Date**: YYYY-MM-DD format
- **Status**: ✅ Completed (or current status)
- **Summary**: 2-3 sentence overview of what was accomplished

### Detailed Sections:
- **What Was Built/Changed**: Detailed list of:
  - Models/Controllers/Views created or modified
  - Migrations and database changes
  - Configuration changes
  - New features or functionality

- **Key Implementation Details**: Important technical decisions, patterns used, or architectural choices

- **Files Created/Modified**: Complete list organized by type:
  - Models
  - Controllers
  - Views
  - Migrations
  - Tests
  - Configuration files
  - Other

- **Testing**:
  - Test results
  - Test coverage added
  - Fixtures/test data created

- **Design Decisions**: Document any important choices made during implementation and why

- **Next Steps**: What tasks are now unblocked or what should be done next

- **Notes**: Any additional context, caveats, or important information

## Format Example:

```markdown
# [Task Name]

**Date**: YYYY-MM-DD
**Status**: ✅ Completed

## Summary

[2-3 sentences describing what was accomplished]

## What Was Built/Changed

- Item 1
- Item 2

## Key Implementation Details

- Detail 1
- Detail 2

## Files Created/Modified

### Models
- `app/models/example.rb`

### Tests
- `test/models/example_test.rb`

## Testing

[Test results and coverage info]

## Design Decisions

[Important choices and rationale]

## Next Steps

[What's unblocked or what comes next]

## Notes

[Additional context]
```

Now create the task summary based on the work that was just completed.
