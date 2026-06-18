# AI Log — fix: Progress UUID ids

**Date:** 2026-06-18
**Branch:** fix/progress-uuid-ids

## Task / problem

Deep review of all tickets revealed that `ProgressId` and `CompletedLevelId` had public constructors that accepted any non-empty string. The `ProgressController` was generating IDs as `progress-${userId}` and `CompletedLevelId` was derived as `${progressId}-${levelId}`. Both formats are not valid UUIDs, causing `invalid input syntax for type uuid` errors against the `UUID PRIMARY KEY` schema in production.

## Tool and model

- Tool: Claude Code (claude.ai/code)
- Model: Claude Sonnet 4.6

## Prompt used

User requested a deep review of all tickets and instructed to start fixing critical issues.

## Agent Roles Used

| Agent | Status | How it was used | Evidence |
| --- | --- | --- | --- |
| Spec Partner | Not used | N/A | N/A |
| Planner/Slicer | Referenced | Deep review identified the bug and all affected files before touching code | review report |
| TDD Implementer | Referenced | Tests updated alongside each production file change; all 277 pass | tests/application/progress/, tests/domain/progress/ |
| Judge | Not used | N/A | N/A |
| Mutation Tester | Not used | N/A | N/A |

## Result obtained

- `ProgressId`: private constructor, UUID v4 validation, `create()` + `generate()` static factories, `InvalidArgumentError` on invalid input
- `CompletedLevelId`: same pattern as above
- `PlayerProgress.recordCompletion`: uses `CompletedLevelId.generate()` instead of derived string
- `LoadProgressService`, `CompleteLevelService`, `SyncProgressService`: removed `progressId`/`newProgressId` from inputs; use `ProgressId.generate()` server-side
- `ProgressController`: removed all `progress-${userId}` patterns
- `PgProgressRepository`: uses `ProgressId.create()` and `CompletedLevelId.create()` in mapper
- All test fixtures updated to use valid UUID constants

## Team modifications pending human review

- `SyncProgressService` now creates the transient `local` object using `remote.id` (when remote exists) to satisfy `ProgressMergePolicy` which verifies user ownership. Team should confirm this is the correct identity for the local in-memory snapshot.
- `LoadProgressService` still persists an empty progress on first load (CQS violation noted in review). Left as-is — fixing requires a separate decision on whether to defer creation to `completeLevel`.

## Lessons / limitations

- Non-UUID IDs in domain VOs are invisible in tests when pool mocks accept any string. The bug only surfaces against a real PostgreSQL DB. This is why UUID validation must be enforced in the VO itself, not delegated to the DB constraint.
