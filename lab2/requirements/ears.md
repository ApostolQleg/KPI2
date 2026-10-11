# EARS Requirements — SchemaPostol

> Lab 2 · Task 2 · Requirements engineering.
> Sources: `lab2/spec.md` (actors, domain logic) and `lab1/model/er-diagram.mmd` (Task 1 ER model).
> All entity, attribute and role names below are copied **verbatim** from the ER model (`User`, `Schematic`, `Comment`, `Category`, `Tag`, `Version`).

## 1. EARS Pattern Conventions

| Pattern | Template | ID Prefix |
|---|---|---|
| Ubiquitous | The system SHALL `<response>` | `REQ-SYS-XXX` |
| Event-driven | WHEN `<trigger>`, the system SHALL `<response>` | `REQ-EVN-XXX` |
| State-driven | WHILE `<state>`, the system SHALL `<response>` | `REQ-STA-XXX` |
| Optional | WHERE `<feature>`, the system SHALL `<response>` | `REQ-OPT-XXX` |
| Unwanted behaviour | IF `<condition>`, THEN the system SHALL `<response>` | `REQ-UNW-XXX` |
| Out of scope | The system SHALL NOT `<response>` | `REQ-OUT-XX` |

## 2. Ubiquitous (`REQ-SYS-XXX`)

### REQ-SYS-001 — Public catalog access for everyone
* **Pattern:** Ubiquitous
* **Statement:** The system SHALL allow any visitor to browse published `Schematic` entries together with their `Category`, `Tag`, `Version` and `Comment` data, regardless of the request context.
* **Rationale:** The catalog is the core of the platform: free public access to view schematics is the product's main value proposition (`README.md`, `lab2/spec.md` → Гість).
* **ER entities touched:** `Schematic`, `Category`, `Tag`, `Version`, `Comment`

### REQ-SYS-002 — Registration, authentication and profile identity
* **Pattern:** Ubiquitous
* **Statement:** The system SHALL support user registration and authentication such that every `User` has a unique `nickname` and a unique `email`, and SHALL expose the `nickname` as the public identifier of the account.
* **Rationale:** `lab2/spec.md` defines `Registered User` as an authenticated player; ADR-0004 mandates `nickname` for human-readable references.
* **ER entities touched:** `User` (`user_id`, `nickname`, `email`, `password_hash`, `created_at`)

### REQ-SYS-003 — Referential integrity of the domain model
* **Pattern:** Ubiquitous
* **Statement:** The system SHALL preserve the ER model invariants at all times: every `Schematic` SHALL reference exactly one author (`author_id` → `User`) and exactly one `Category` (`category_id`), at least one `Tag` and at least one `Version`; every `Comment` SHALL reference exactly one `User` (`author_id`) and exactly one `Schematic` (`schematic_id`).
* **Rationale:** Directly mirrors the cardinalities of `lab1/model/er-diagram.mmd` / `lab1/spec.md` relations 1–7 (M:N kept direct, no associative tables without attributes — ADR-0002).
* **ER entities touched:** `User`, `Schematic`, `Comment`, `Category`, `Tag`, `Version`

## 3. Event-driven (`REQ-EVN-XXX`)

### REQ-EVN-001 — Schematic submission dispatches parsing
* **Pattern:** Event-driven
* **Statement:** WHEN an `Author` submits a `.litematica` file for publication, the system SHALL create a new `Schematic` record linked to that author (`author_id`) and dispatch the file to the `Litematica Parser Service` for validation and metadata extraction.
* **Rationale:** Publishing requires attaching a `.litematica` file (`README.md` → Schematics); the parser is a mandatory participant of the publication flow (`lab2/spec.md` → Парсер litematica файлів).
* **ER entities touched:** `Schematic` (`schematic_id`, `author_id`, `litematica_url`)

### REQ-EVN-002 — Parser success publishes enriched schematic
* **Pattern:** Event-driven
* **Statement:** WHEN the `Litematica Parser Service` confirms file integrity and returns extracted metadata, the system SHALL enrich the `Schematic` with the received metadata (title, description, dimensions) and make the publication discoverable through search and filters.
* **Rationale:** The parser runs in the background and "generates metadata that then flows into the main system" (`lab2/spec.md`); `description` in the ER model is optional because it may be supplemented by parser output.
* **ER entities touched:** `Schematic` (`title`, `description`, `slug`)

### REQ-EVN-003 — Comment submission persists a Comment
* **Pattern:** Event-driven
* **Statement:** WHEN a `Registered User` submits a comment on a schematic page, the system SHALL persist a new `Comment` record linking the user (`author_id`) with the `Schematic` (`schematic_id`) and store its `content` and `created_at`.
* **Rationale:** `lab2/spec.md` grants commenting to authenticated users; relations 3–4 of the ER model define comment ownership.
* **ER entities touched:** `Comment` (`comment_id`, `author_id`, `schematic_id`, `content`, `created_at`)

## 4. State-driven (`REQ-STA-XXX`)

### REQ-STA-001 — Unvalidated schematics stay hidden
* **Pattern:** State-driven
* **Statement:** WHILE a `Schematic` is awaiting validation by the `Litematica Parser Service`, the system SHALL NOT expose it in search results, filter results, category/tag pages or as a publicly accessible schematic page.
* **Rationale:** The parser works in the background (`lab2/spec.md`); a half-validated publication must never leak into the public catalog (ties to `slug`-based public links, ADR-0004).
* **ER entities touched:** `Schematic`

### REQ-STA-002 — Guests are read-only
* **Pattern:** State-driven
* **Statement:** WHILE a visitor is unauthenticated (acting as `Guest`), the system SHALL restrict them to read-only access to public `Schematic`, `Category`, `Tag`, `Version` and `Comment` data, and SHALL route every attempt to comment, save, publish or edit to the authentication flow.
* **Rationale:** `lab2/spec.md` gives Guests browse/filter/view capabilities only; write operations belong to `Registered User` and `Author`.
* **ER entities touched:** `User`, `Schematic`, `Comment`

## 5. Optional (`REQ-OPT-XXX`)

### REQ-OPT-001 — Optional Minecraft world archive
* **Pattern:** Optional
* **Statement:** WHERE an `Author` attaches a full Minecraft world archive (`.zip`) during publication, the system SHALL store its location in `Schematic.world_zip_url` and expose it as an optional download on the schematic page.
* **Rationale:** `README.md` and the ER model mark `world_zip_url` as optional; the feature only applies when the archive is provided.
* **ER entities touched:** `Schematic` (`world_zip_url`)

### REQ-OPT-002 — Optional saved-schematics collection
* **Pattern:** Optional
* **Statement:** WHERE a `Registered User` chooses to save a schematic, the system SHALL maintain the direct many-to-many `saves` relation between `User` and `Schematic` and allow the user to remove the schematic from their saved collection at any time.
* **Rationale:** "Saving liked schematics" is an opt-in feature (`README.md` → Services); the ER model states `User }o--o{ Schematic : saves` as a direct M:N relation without an associative entity (ADR-0002).
* **ER entities touched:** `User`–`Schematic` (`saves` M:N)

## 6. Unwanted Behaviour (`REQ-UNW-XXX`)

### REQ-UNW-001 — Corrupted `.litematica` file is rejected
* **Pattern:** Unwanted behaviour
* **Statement:** IF a submitted `.litematica` file fails the integrity check performed by the `Litematica Parser Service`, THEN the system SHALL reject the publication attempt, keep the `Schematic` unpublished and invisible to other users, and notify the author with the parser's error details.
* **Rationale:** The parser explicitly "checks files for integrity" (`lab2/spec.md`); failure is an unwanted but expected condition that must never corrupt the public catalog.
* **ER entities touched:** `Schematic` (`litematica_url`)

### REQ-UNW-002 — Editing someone else's publication is denied
* **Pattern:** Unwanted behaviour
* **Statement:** IF a user attempts to edit or delete a `Schematic` where `Schematic.author_id` differs from the authenticated `User.user_id`, THEN the system SHALL deny the operation, SHALL NOT mutate any `Schematic`, `Tag`, `Version` or `Category` data, and SHALL NOT disclose non-public data of the publication.
* **Rationale:** Only authors may manage their own publications (`lab2/spec.md` → Автор); ER relation 1 (`User ||--o{ Schematic : creates`) defines exclusive ownership.
* **ER entities touched:** `Schematic` (`author_id`), `Category`, `Tag`, `Version`

## 7. Out of Scope (`REQ-OUT-XX`)

### REQ-OUT-01 — No in-game integration
* **Pattern:** Out of scope (NOT)
* **Statement:** The system SHALL NOT provide in-game integration — that is, automatic delivery or installation of schematics into Minecraft game clients or dedicated Minecraft servers; schematics are consumed exclusively through file downloads (`litematica_url`, optional `world_zip_url`) via the web platform.
* **Rationale:** Explicitly excluded from the current release scope to keep the platform a pure catalog/hosting service; deferred to future releases and therefore intentionally left **UNCOVERED** by use cases in `lab2/requirements/traceability-matrix.md`.
* **ER entities touched:** — (no ER entity exists for in-game delivery)

## 8. Summary

| ID | Pattern | Short statement |
|---|---|---|
| REQ-SYS-001 | Ubiquitous | Public catalog access for every visitor |
| REQ-SYS-002 | Ubiquitous | Registration/auth with unique `nickname` & `email` |
| REQ-SYS-003 | Ubiquitous | Referential integrity per the ER model |
| REQ-EVN-001 | Event-driven | Submission creates `Schematic` and dispatches parsing |
| REQ-EVN-002 | Event-driven | Parser success enriches and publishes the `Schematic` |
| REQ-EVN-003 | Event-driven | Comment submission persists a `Comment` |
| REQ-STA-001 | State-driven | Unvalidated `Schematic` stays hidden |
| REQ-STA-002 | State-driven | Guests are read-only |
| REQ-OPT-001 | Optional | World archive → `world_zip_url` download |
| REQ-OPT-002 | Optional | Direct M:N `saves` relation for registered users |
| REQ-UNW-001 | Unwanted behaviour | Corrupted file rejected, author notified |
| REQ-UNW-002 | Unwanted behaviour | Non-owner edit/delete denied |
| REQ-OUT-01 | Out of scope | No in-game integration (**UNCOVERED**) |

> Traceability to use cases: `lab2/requirements/traceability-matrix.md`.



