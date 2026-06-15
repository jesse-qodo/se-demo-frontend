# Code Review Skills — se-demo-frontend

This is the React/TypeScript frontend for the Qodo demo suite. It consumes a REST API served by the `se-demo-api` backend.

## Primary Skill: API Contract Alignment

The types in `src/types/` are exact mirrors of Go structs in the backend. Every field name and enum value must match the backend's `json:"..."` tags precisely — mismatches are invisible at compile time but break at runtime.

- `src/types/todo.ts` mirrors `api/services/todos.go → Todo`
- `src/types/mail.ts` mirrors `api/services/emails.go → Email`
- `src/types/event.ts` mirrors `api/services/events.go → Event`
- `src/types/contact.ts` mirrors `api/services/contacts.go → Contact`
- `src/types/note.ts` mirrors `api/services/notes.go → Note`
- `src/api/agent.ts` types mirror `api/handlers/agent.go → chatResponse` and `api/chat/store.go → Message`

Flag any PR that renames a field in `src/types/` without a confirmed matching backend change, or that adds an enum value the backend doesn't accept.

## Skill: Endpoint Path Correctness

All fetch calls in `src/api/` must reference real backend paths. A renamed or removed endpoint on the backend is a silent 404.

- PATCH todos: `/api/todos/${id}` (not `/update`, not `/patch`)
- Boolean filter: `?completed=true` or `?completed=false` (not `1`/`0`)
- Auth: every request must include `Authorization: Bearer <token>` via `authHeaders()`

## Skill: Auth Token Freshness

`authHeaders()` in `src/api/base.ts` must call `auth.currentUser?.getIdToken()` on every request. A module-level token cache without expiry refresh means requests silently fail after one hour.

## Skill: React Hooks Correctness

`useEffect` hooks that read a prop or state value must include that value in the dependency array. An empty `[]` array is only correct for true mount-only effects. Missing deps cause stale-data bugs when navigating between items.

## Skill: XSS Prevention

User-supplied content (todo descriptions, email bodies, contact notes) must be rendered as React text children — never via `dangerouslySetInnerHTML` or `innerHTML` without explicit DOMPurify sanitization.

## Cross-Repo Awareness

This repo is the consumer side of contracts defined in `se-demo-api`. Qodo should flag PRs where a type change here is not matched by a backend change, and vice versa.
