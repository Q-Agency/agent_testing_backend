# SPEC — #17368106: CRUD Table (Test 1 Task)
**Type:** Feature
**Status:** APPROVED
**Author:** BA / AI-assisted
**Date:** 2025-01-31

## Overview
A dedicated full-page CRUD table for managing **Notes** inside the Markdown Note App. The initial scope covers the Create operation (adding new notes via a modal form) and the Read operation (displaying all notes in a table). The UI must be displayed in Croatian language per client requirement.

## User Roles
| Role | Description |
|------|-------------|
| Single user | Single-user system per constitution. No authentication, access control, or ownership checks. All CRUD operations are available without restriction. |

## User Stories
- As a user, I want to view all my notes in a dedicated full-page table so that I can see all notes at a glance.
- As a user, I want to create a new note from the table page so that I can quickly add new content.

## Acceptance Criteria

### Note List (Read)
- **Given** I navigate to the `/notes` page, **when** the page loads, **then** a `GET /api/v1/notes` request is made and the table is populated with all non-deleted notes sorted by `created_at` descending.
- **Given** the API returns no notes (`[]`), **when** the table renders, **then** centered text "Nema bilješki" and hint "Kliknite 'Nova bilješka' za početak" are shown.
- **Given** the API returns one or more notes, **when** the table renders, **then** each row displays: Naslov (full title), Sadržaj (first 100 characters of content + "..."), Stvoreno (`dd.mm.yyyy. HH:mm`, e.g., `15.03.2025. 14:30`).
- **Given** the list is loading, **when** data is pending, **then** a spinner or skeleton row appears instead of the table.
- **Given** the API returns HTTP 500 or a network timeout (>30 s), **then** a toast error appears with message "Ne mogu učitati bilješke. Provjerite internetsku vezu" and a "Pokušaj ponovno" button that retries the list request.

### Note Creation (Create)
- **Given** I am on the Notes table page, **when** I click "Nova bilješka", **then** a modal titled "Nova bilješka" opens with: Naslov (text input, max 200 chars), Sadržaj (markdown textarea, max 100,000 chars), and buttons *Sačuvaj* and *Odustani*; *Sačuvaj* is initially disabled.
- **Given** the modal is open, **when** Naslov is non-empty (after trimming, ≤200 chars) and Sadržaj is non-empty (after trimming, >0 chars), **then** *Sačuvaj* becomes enabled.
- **Given** *Sačuvaj* is enabled and I click it, **when** the request begins, **then** *Sačuvaj* becomes disabled and displays loading text "Spremanje..."; the modal remains open; no form data is lost.
- **Given** the form is submitted with valid data, **when** the server returns HTTP 201 Created, **then** the modal closes and the new note appears at the top of the table.
- **Given** the form submission returns HTTP 422, **when** the response body contains an `errors` array with field-level messages, **then** each corresponding input shows its Croatian error message inline below the field, *Sačuvaj* re-enables, loading text disappears, and form data is preserved.
- **Given** the form submission returns HTTP 409 Conflict, **then** a toast appears at top-right with message "Naslov već postoji u odabranom direktoriju", *Sačuvaj* re-enables, loading text disappears, and modal stays open with data intact.
- **Given** the form submission returns HTTP 500 or a network error (timeout >30 s), **then** a toast appears with message "Dogodila se greška na poslužitelju. Pokušajte ponovno kasnije" (500) or "Ne mogu se povezati s poslužiteljem. Provjerite internetsku vezu" (network), with a "Pokušaj ponovno" button that retries the same create request.
- **Given** the modal is open, **when** I click *Odustani* or press `Esc`, **then** the modal closes without saving, all input is discarded, and no side effects occur.

## Scope

### In Scope
- Dedicated full-page Notes table view at `/notes` route
- Table columns: Naslov, Sadržaj (truncated to 100 chars + "..."), Stvoreno (`dd.mm.yyyy. HH:mm`)
- Create operation: "Nova bilješka" button opens modal form; on success note appears at top of table
- Croatian UI language throughout

### Out of Scope
- Update (edit) and Delete operations
- Folder management UI
- Search, filter, sort controls
- Pagination
- Live markdown preview in table
- Tags UI

## Business Rules

- **Note Title**: Required; non-empty after trimming; maximum 200 characters. Leading/trailing whitespace trimmed before validation and storage. Must be unique within the same `folder_id` (including `null`).
- **Note Content**: Required; non-empty after trimming; maximum 100,000 characters.
- **Folder Association**: Optional. If `folder_id` is provided, it must reference an existing folder. When `null`, the note resides at root level.
- **Duplicate Title Behavior**: On create, if a note with the same `title` and `folder_id` already exists, the operation fails with HTTP 409 Conflict.
- **Timestamps**: `created_at` set server-side to UTC on creation; immutable. `updated_at` set to UTC on every write. Both stored as ISO 8601.
- **Form Submission State**: *Sačuvaj* is disabled during submission to prevent double-submission. Re-enabled after error.
- **Croatian Language Enforcement**: All user-facing UI text is in Croatian. No English text appears in the UI.
- **Sorting**: Notes sorted by `created_at` descending (newest first); enforced at the API level.
- **Validation Scope**: Client-side validates title/content length and empty checks. Server-side repeats all checks and additionally enforces uniqueness, folder existence, and max-length constraints. All validation errors use per-field details in RFC 7807 responses.
- **Soft-delete filtering**: All reads exclude notes where `deleted_at IS NOT NULL`.
- **SQLite NULL uniqueness**: The DB constraint `UNIQUE (title, folder_id)` permits multiple root-level notes with the same title (SQLite treats `NULL != NULL`). The repository layer must enforce root-level uniqueness via an explicit query check before insertion.

## Error Handling

All errors follow RFC 7807 Problem Details JSON format: `{"type": "...", "title": "...", "status": ..., "detail": "...", "instance": "/api/v1/notes"}`.

For 422 responses, the body includes an `errors` array for field-level failures:
```json
"errors": [
  {
    "field": "title",
    "code": "title_too_long",
    "message": "Naslov ne smije biti duži od 200 znakova"
  }
]
```

No logging is performed in this POC, per constitution.

#### Backend Error Mapping

| Error Code | HTTP Status | Trigger | User Message (Croatian) | Frontend Display |
|------------|-------------|---------|--------------------------|-----------------|
| `title_required` | 422 | Title missing or empty | "Naslov je obavezan" | Inline below title field |
| `title_too_long` | 422 | Title exceeds 200 chars | "Naslov ne smije biti duži od 200 znakova" | Inline below title field |
| `content_required` | 422 | Content missing or empty | "Sadržaj je obavezan" | Inline below content field |
| `content_too_long` | 422 | Content exceeds 100,000 chars | "Sadržaj ne smije biti duži od 100 000 znakova" | Inline below content field |
| `folder_not_found` | 404 | `folder_id` does not exist in DB | "Odabrana mapa ne postoji" | Toast/banner (red, top-right) |
| `title_duplicate` | 409 | Same title in same `folder_id` already exists | "Naslov je već zauzet u ovoj mapi" | Toast/banner (red, top-right) |
| `validation_failed` | 422 | Malformed JSON or missing required fields | "Podaci nisu ispravni. Provjerite unesene podatke" | Toast/banner (red, top-right) |
| `internal_error` | 500 | Unhandled server exception | "Dogodila se greška na poslužitelju. Pokušajte ponovno kasnije" | Toast/banner (red, top-right) |
| `network_error` | N/A | Backend unreachable, timeout (>30 s), or CORS failure | "Ne mogu se povezati s poslužiteljem. Provjerite internetsku vezu" | Toast/banner (red, top-right); persists until user retries |

#### Frontend Display Rules
- **Inline errors** (`title_required`, `title_too_long`, `content_required`, `content_too_long`): red text (`text-red-600`) beneath the corresponding field.
- **Toast errors** (all others): red background (`bg-red-50`), icon, message, auto-dismiss after 5 seconds (except `network_error`, which persists until user retries).
- All error messages are in Croatian; no English fallback.

#### Modal State on Error
- **422 field errors**: modal stays open, form data preserved, inline errors shown per field.
- **404, 409, 500**: modal stays open, form data preserved, error shown as toast only.
- **Network error**: modal stays open, form data preserved, toast persists; user must click "Pokušaj ponovno" to retry.
- Modal close button remains enabled in all error states.

#### Internal Error Response Handling
- **Structured 500 RFC 7807 response**: displayed as toast using verbatim `detail` field if present and non-empty (Croatian), otherwise generic `internal_error` message.
- **Unstructured crash** (empty body, non-JSON, proxy timeout): treated as `network_error` regardless of HTTP status.

## API Specification

### `GET /api/v1/notes`
Returns all non-deleted notes sorted by `created_at` descending.

**Response 200:**
```json
[
  {
    "id": 1,
    "title": "Moja bilješka",
    "content": "Sadržaj...",
    "folder_id": null,
    "created_at": "2025-01-31T10:00:00Z",
    "updated_at": "2025-01-31T10:00:00Z"
  }
]
```

### `POST /api/v1/notes`
Creates a new note.

**Request body:**
```json
{
  "title": "Moja bilješka",
  "content": "Sadržaj bilješke",
  "folder_id": null
}
```

**Response 201:** Created note object (same schema as GET item).
**Response 404:** `folder_id` does not exist.
**Response 409:** Duplicate title in same folder.
**Response 422:** Validation failure.

## Data Model

#### Entity: Note

| Field | Type | Required | Constraints | Notes |
|-------|------|----------|-------------|-------|
| id | integer | yes | Primary key, auto-increment | Generated by database on insert |
| title | string | yes | Max 200 characters; non-empty after trimming | Composite unique constraint with `folder_id` |
| content | string | yes | Max 100,000 characters; UTF-8 | Leading/trailing whitespace trimmed before validation |
| folder_id | integer | no | FK to `folders.id`; `NULL` = root-level note | `ON DELETE SET NULL`; root-level uniqueness enforced via application logic (see Business Rules) |
| created_at | datetime | yes | Non-null; server-side default | Set at insert; never updated |
| updated_at | datetime | yes | Non-null; server-side default | Updated on every write |
| deleted_at | datetime | no | Soft-delete marker | `NULL` = active; non-null = soft-deleted |

##### SQL DDL (reference only — use parameterized queries via repository layer)
```sql
CREATE TABLE notes (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  title TEXT NOT NULL,
  content TEXT NOT NULL,
  folder_id INTEGER,
  created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
  deleted_at DATETIME,
  FOREIGN KEY (folder_id) REFERENCES folders(id) ON DELETE SET NULL,
  UNIQUE (title, folder_id)
);
CREATE INDEX idx_notes_folder_id ON notes(folder_id);
CREATE INDEX idx_notes_deleted_at ON notes(deleted_at);
```

##### Table Display Columns (CRUD view)

| Column Header (Croatian) | Field | Display Format |
|--------------------------|-------|----------------|
| Naslov | title | Full text |
| Sadržaj | content | First 100 characters + "..." if truncated |
| Stvoreno | created_at | `dd.mm.yyyy. HH:mm` (e.g., `23.02.2026. 14:30`) |

##### Fields Not Exposed in This View
- `folder_id`: backing store; hidden in table view
- `updated_at`: tracked server-side; not displayed
- `deleted_at`: soft-delete marker; hidden but enforced at read time

##### Relationships
- **Parent**: `Folder` (optional, via `folder_id`)
- **Cardinality**: Many `Note` records per `Folder`; zero or one `Folder` per `Note`

## Dependencies

### Backend
| Layer | Technology | Version |
|-------|------------|---------|
| Language | Python | 3.12 |
| Framework | FastAPI | 0.115.x |
| Database | SQLite | 3.x (stdlib) |
| Testing | pytest | 8.x |
| Linting | Ruff | 0.9.x |
| Type Checking | mypy | 1.14.x |

### Frontend
| Layer | Technology | Version |
|-------|------------|---------|
| Language | TypeScript | 5.7 |
| Framework | React | 19 |
| Build Tool | Vite | 6.x |
| Testing | Vitest | 3.x |
| Styling | TailwindCSS | 4.x |

## Open Questions
*(None — all outstanding questions resolved. See Decision Log.)*

## Decision Log
| # | Question | Decision | Source | Confidence |
|---|----------|----------|--------|------------|
| 1 | UI language | Croatian | Teamwork Task #17368106 comment | high |
| 2 | Error response format | RFC 7807 Problem Details JSON | Constitution | high |
| 3 | API versioning | URL prefix `/api/v1/` | Constitution | high |
| 4 | Entity managed by CRUD table | Notes only | BA response, Round 1 | high |
| 5 | CRUD operations in scope | Create + Read only | BA response, Round 1 | high |
| 6 | Table placement | Dedicated full-page view at `/notes` | BA response, Round 1 | high |
| 7 | Note content max length | 100,000 characters | Constitution | high |
| 8 | Tags exposure in this view | Tags not exposed; out of scope for this feature | Assumption | medium |
| 9 | Soft-delete mechanism | `deleted_at` timestamp (null = active) | Constitution | high |
| 10 | Authentication / roles | Single-user; no auth or access control | Constitution | high |
