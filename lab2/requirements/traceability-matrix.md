# Traceability Matrix — SchemaPostol

> Lab 2 · Task 2 · Requirements ↔ Use Cases.
> Requirements: `lab2/requirements/ears.md` (15 EARS requirements). Use cases: `lab2/model/use-case-diagram.puml` (15 use cases).
> Per `lab2/spec.md` the mapping is a **single many-to-many ($M:N$) matrix**: every EARS requirement is a row, every use case is a column, and `✔` marks a direct link. One requirement is covered by several use cases and one use case implements several requirements.

## 1. Use case legend

| Use case ID | Use case | Primary actor |
|---|---|---|
| UC-01 | Register Account | Guest |
| UC-02 | Log In | Guest |
| UC-03 | Search Schematics | Guest |
| UC-04 | Apply Filters (Category, Tag, Version) | Guest |
| UC-05 | View Schematic Details (incl. Comments) | Guest |
| UC-06 | Write Comment | Registered User |
| UC-07 | Save Schematic | Registered User |
| UC-08 | Manage Profile (Microsoft OAuth account link) | Registered User |
| UC-09 | Publish Schematic | Author |
| UC-10 | Classify Schematic (Category, Tag, Version) | Author |
| UC-11 | Parse Litematica File (integrity + metadata) | — (dependent external background service `Litematica Parser Service`, **not an actor** per `lab2/spec.md`; reached via `<<delegate>>` from UC-11) |
| UC-12 | Edit Own Publication | Author |
| UC-13 | Attach World Archive (optional, `world_zip_url`) | Author |
| UC-14 | Download World Archive (optional, `world_zip_url`) | Guest |
| UC-15 | Delete Own Publication | Author |

> `Log In` (UC-02) is a **precondition** of the authenticated use cases (UC-06, UC-07, UC-08, UC-09, UC-10, UC-12, UC-13, UC-15), not an `<<include>>` (`lab2/spec.md`, rule 2).

## 2. Consolidated M:N matrix — EARS requirement ↔ Use case

| EARS requirement | Pattern | UC-01 | UC-02 | UC-03 | UC-04 | UC-05 | UC-06 | UC-07 | UC-08 | UC-09 | UC-10 | UC-11 | UC-12 | UC-13 | UC-14 | UC-15 | Status |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| REQ-SYS-001 — Public catalog access for every visitor | Ubiquitous |  |  | ✔ | ✔ | ✔ |  |  |  |  |  |  |  |  |  |  | COVERED |
| REQ-SYS-002 — Registration/auth with unique `nickname`/`email`, `Author` via OAuth | Ubiquitous | ✔ | ✔ |  |  |  |  |  | ✔ |  |  |  |  |  |  |  | COVERED |
| REQ-SYS-003 — Referential integrity of the ER model | Ubiquitous |  |  |  |  |  | ✔ | ✔ |  | ✔ | ✔ |  | ✔ |  |  |  | COVERED |
| REQ-EVN-001 — Submission creates `Schematic` + dispatches parsing | Event-driven |  |  |  |  |  |  |  |  | ✔ | ✔ | ✔ |  |  |  |  | COVERED |
| REQ-EVN-002 — Parser success enriches and publishes `Schematic` | Event-driven |  |  |  |  |  |  |  |  | ✔ |  | ✔ |  |  |  |  | COVERED |
| REQ-EVN-003 — Comment submission persists a `Comment` | Event-driven |  |  |  |  |  | ✔ |  |  |  |  |  |  |  |  |  | COVERED |
| REQ-EVN-004 — Microsoft OAuth elevates `Registered User` → `Author` | Event-driven |  |  |  |  |  |  |  | ✔ | ✔ |  |  |  |  |  |  | COVERED |
| REQ-STA-001 — Unvalidated `Schematic` stays hidden | State-driven |  |  | ✔ |  | ✔ |  |  |  |  |  | ✔ |  |  |  |  | COVERED |
| REQ-STA-002 — Guests are read-only | State-driven |  | ✔ |  |  |  | ✔ | ✔ |  |  |  |  |  |  |  |  | COVERED |
| REQ-OPT-001 — World archive → `world_zip_url` download | Optional |  |  |  |  |  |  |  |  |  |  |  |  | ✔ | ✔ |  | COVERED |
| REQ-OPT-002 — Direct M:N `saves` relation (`User`–`Schematic`) | Optional |  |  |  |  |  |  | ✔ |  |  |  |  |  |  |  |  | COVERED |
| REQ-UNW-001 — Corrupted `.litematica` file rejected | Unwanted behaviour |  |  |  |  |  |  |  |  | ✔ |  | ✔ |  |  |  |  | COVERED |
| REQ-UNW-002 — Non-owner edit/delete denied | Unwanted behaviour |  |  |  |  |  |  |  |  |  |  |  | ✔ |  |  | ✔ | COVERED |
| REQ-UNW-003 — Publishing without OAuth elevation denied | Unwanted behaviour |  |  |  |  |  |  |  |  | ✔ | ✔ |  |  |  |  |  | COVERED |
| REQ-OUT-01 — No in-game integration | Out of scope |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  | **UNCOVERED** |
| **Links per use case** | | 1 | 2 | 2 | 1 | 2 | 3 | 3 | 2 | 6 | 3 | 4 | 2 | 1 | 1 | 1 | **34 links** |

**Legend:** `✔` — the use case implements / is required by the requirement; empty cell — no link; `COVERED` — at least one use case implements the requirement; `UNCOVERED` — intentionally not implemented in the current release (out of scope, tracked for future releases).

## 3. Why the mapping is $M:N$

* **One requirement → many use cases:** e.g. REQ-SYS-003 is realised jointly by *Write Comment* (UC-06), *Save Schematic* (UC-07), *Publish Schematic* (UC-09), *Classify Schematic* (UC-10) and *Edit Own Publication* (UC-12); REQ-SYS-001 spans UC-03, UC-04, UC-05; REQ-EVN-001 spans UC-09, UC-10, UC-11.
* **One use case → many requirements:** e.g. *Publish Schematic* (UC-09) simultaneously implements REQ-SYS-003 (integrity invariants), REQ-EVN-001 (submission event), REQ-EVN-002 (parser callback), REQ-EVN-004 (OAuth role gate), REQ-UNW-001 (rejection path) and REQ-UNW-003 (denial without elevation) — six requirements; *Parse Litematica File* (UC-11) implements four.
* **Cardinality check:** 14 of the 15 requirements are linked to at least one of the 15 use cases through **34 distinct requirement↔use-case links** (see the "Links per use case" totals in the matrix above) — far beyond a $1:1$ mapping. Only REQ-OUT-01 has an empty row, by design.

## 4. Uncovered requirement

| Requirement | Reason for being UNCOVERED | Planned for |
|---|---|---|
| REQ-OUT-01 (no in-game integration) | Explicitly out of the current release scope: no use case may deliver schematics into Minecraft clients/servers, therefore no use case exists for it by design. Marked **UNCOVERED** to track the future release that will lift this exclusion. | Future release ("in-game installer" epic) |

## 5. Consistency check: Use Cases ↔ ER model (`lab1/model/er-diagram.mmd`)

| Use Cases | `User` | `Schematic` | `Comment` | `Category` | `Tag` | `Version` |
|---|---|---|---|---|---|---|
| UC-01, UC-02, UC-08 | ✔ (`user_id`, `nickname`, `email`, `password_hash`, `created_at`) | — | — | — | — | — |
| UC-03, UC-04, UC-05 | — | ✔ (`schematic_id`, `slug`, `title`, `description`, `litematica_url`) | ✔ (read) | ✔ (`name`) | ✔ (`name`) | ✔ (`version_num`) |
| UC-06 | ✔ (`author_id`) | ✔ (`schematic_id`) | ✔ (`comment_id`, `content`, `created_at`) | — | — | — |
| UC-07 | ✔ (`saves` M:N side) | ✔ (`saves` M:N side) | — | — | — | — |
| UC-09, UC-10 | ✔ (`author_id` → `User`) | ✔ (`author_id`, `category_id`, `slug`, `title`, `description`, `litematica_url`, `world_zip_url`, `created_at`) | — | ✔ | ✔ | ✔ |
| UC-11 | — | ✔ (`litematica_url`, extracted metadata) | — | — | — | — |
| UC-12, UC-15 | ✔ (`author_id` ownership check) | ✔ (full row lifecycle) | ✔ (comments removed with publication) | ✔ | ✔ | ✔ |
| UC-13, UC-14 | — | ✔ (`world_zip_url`) | — | — | — | — |

* UC-08 links a Microsoft game account to the `User` for the OAuth role elevation (REQ-EVN-004, ADR-0005); the ER model keeps no `role` attribute, so the elevation introduces no new entities or attributes — it is authorization data of the `User` account.


