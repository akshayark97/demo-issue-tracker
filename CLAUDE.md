# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
bun install       # install dependencies
bun run dev       # start dev server at http://localhost:3000
bun run build     # production build
bun run typecheck # run tsc --noEmit (no test suite exists)
```

There is no linter configured. Use `typecheck` to catch type errors.

## Architecture

This is a minimal Next.js 15 (App Router) kanban board. **All data is in-memory and resets on server restart** — there is no database.

### Data flow

- `lib/store.ts` — singleton `IssueStore` (a `Map<string, Issue>`) that lives in `globalThis` to survive Next.js hot-reloads in dev. All mutations happen here.
- `app/api/` — thin Next.js Route Handlers that delegate directly to `store`. No middleware, no auth.
- `components/Board.tsx` — the only stateful client component. Owns the `issues` array in React state, fetches from the API on mount, and keeps local state optimistically updated for drag events before the API call resolves.

### Drag-and-drop

Uses `@dnd-kit/core` + `@dnd-kit/sortable`. The drag lifecycle splits across two handlers in `Board.tsx`:
- `onDragOver` — optimistic UI: moves the card to the target column immediately in local state.
- `onDragEnd` — reconciles final order and calls `PUT /api/columns/:status/reorder` with the full ordered ID list.

`Column` is a `useDroppable` container; each `IssueCard` is a `useSortable` item. The status `select` inside `IssueCard` has `onPointerDown` propagation stopped to prevent drag activation when interacting with the dropdown.

### Statuses

Defined once in `lib/types.ts` as the `STATUSES` array and `Status` union type. Adding a new status requires only updating that file — the board, columns, and dropdowns all derive from it.

## API

- `GET /api/issues` — returns all issues sorted by `order`
- `POST /api/issues` — body: `{ title: string, description?: string, status?: Status }`
- `PATCH /api/issues/:id` — body: `Partial<{ title, description, status, order }>`; moving to a new status without specifying `order` appends to end of that column
- `DELETE /api/issues/:id` — returns 204
- `PUT /api/columns/:status/reorder` — body: `{ orderedIds: string[] }`; sets `order` = array index for each id and reassigns `status`
