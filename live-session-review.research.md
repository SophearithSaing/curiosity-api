## Spec Summary

Source: `review.md`.

Add an authenticated review endpoint for an active live session. The endpoint
must return a question set, not an individual generated question, and may review
levels from `0` through `currentLevel - 1` (for example, levels `0` through `29`
when the current level is `30`). Persist a new relationship on saved live
question sets so the review query can retrieve the session's generated sets.

Stated constraints:

- Review is limited to completed prior levels and excludes the current level.
- The response is one question set.
- Saved live questions/sets require a session identifier for review.
- The review lookup must use the newly attached identifier described as
  `questionId` in the source.

Assumptions, not requirements:

- "Active session" means `LiveSession`, because only its generated question sets
  are created at runtime.
- Existing `QuestionSetResponseDto` is the intended response shape.

## Impacted Areas

### 1. Live Review Retrieval And Authorization

Priority: High

Complexity: Medium

Impact: The sessions API has no review route or service operation. The new flow
must validate the authenticated student's active live session, restrict the
eligible level range, and return a saved live question set associated with that
session.

Files / Expected Changes:

#### src/sessions/sessions.controller.ts

`SessionsController` // Expose the authenticated live-session review endpoint.

#### src/sessions/sessions.service.ts

`SessionsService` // Authorize the active live session and retrieve one eligible saved set.

#### src/sessions/dtos/session.dto.ts

`ContinueLiveSessionDto` // Provide a validated session identifier if review follows existing body-based routes.

#### src/sessions/dtos/index.ts

`sessions DTO barrel` // Export any new review request DTO.

Risks / Questions: The specification does not define the HTTP method, route, or
whether selection is deterministic (for example, most recent eligible level) or
requested by a level/question-set identifier. It also does not explicitly say
whether review applies to `LiveSession` only.

Resolution: TBD

### 2. Saved Live Set Session Relationship

Priority: High

Complexity: Medium

Impact: A completed live level currently creates a `QuestionSet` with its topic,
level, type, and embedded questions only. It cannot be safely filtered to the
student's live session, so the required review query cannot be implemented.

Files / Expected Changes:

#### src/questions/schemas/question-set.schema.ts

`QuestionSet` // Persist the live-session or specified review lookup reference on generated live sets.

#### src/sessions/sessions.service.ts

`submitLiveAnswer` // Assign the active live-session reference when saving a completed live set.

Risks / Questions: `review.md` first calls the required new field `sessionId`,
then says the set is queried by a newly attached `questionId`. These identify
different entities in the current model: a live session has an ObjectId, while
each embedded question has a string `id` and each `LiveQuestion` document has an
ObjectId. The source must establish the persisted field and lookup key.

Resolution: TBD

Risks / Questions: Existing live `questionSets` have no session relationship.
They cannot be attributed to a specific session after deployment without a
backfill source or an explicit decision to support only newly completed sets.

Resolution: TBD

### 3. Automated Review Coverage

Priority: Medium

Complexity: Medium

Impact: Session tests use mocked Mongoose models and already cover live set
creation, ownership, active-state checks, and controller-to-service delegation.
The new retrieval and persistence behavior needs regression coverage at both
layers.

Files / Expected Changes:

#### test/sessions/sessions.service.spec.ts

`SessionsService live-session tests` // Verify saved-set linkage and review eligibility, ownership, and current-level exclusion.

#### test/sessions/sessions.controller.spec.ts

`SessionsController review tests` // Verify authenticated user and review identifier delegation.

Risks / Questions: Tests must define behavior when no completed saved set exists,
a session is completed or stopped, and the current level is `0`; these outcomes
are not specified.

Resolution: TBD

### 4. API Documentation

Priority: Low

Complexity: Low

Impact: The maintained API document lists every session route and documents live
set persistence, but contains no review endpoint or session-to-question-set
relationship.

Files / Expected Changes:

#### docs/api-documentation.md

`Session endpoints` // Document the review route, request, response, eligible-level rule, and errors.

Risks / Questions: The endpoint contract must be resolved before a stable
frontend-facing example can be documented.

Resolution: TBD

## Side Notes

- Unrelated: `submitAnswer` loads a question set by ID without verifying that its
  topic belongs to the authenticated user's session, allowing an attempt to be
  recorded against a different topic's set.
- Unrelated: `DELETE /sessions/live/:id` deletes only the live-session document;
  its live questions, generated question sets, attempts, and evaluations remain
  persisted as orphaned records.
