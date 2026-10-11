# Traceability Matrix — SchemaPostol

> Lab 2 · Task 2 · Requirements ↔ Use Cases.
> Requirements: `lab2/requirements/ears.md`. Use cases: `lab2/model/use-case-diagram.puml`.
> The mapping is **many-to-many ($M:N$)**: one requirement is covered by several use cases, and one use case implements several requirements.

## 1. Forward mapping — EARS Requirement → Use Cases

| EARS Requirement ID | Pattern | Short statement | Covered by Use Cases | Status |
|---|---|---|---|---|
| REQ-SYS-001 | Ubiquitous | Public catalog access (browse/search/view) | UC-03, UC-04, UC-05 | COVERED |
| REQ-SYS-002 | Ubiquitous | Registration, authentication, unique `nickname`/`email` | UC-01, UC-02, UC-08 | COVERED |
| REQ-SYS-003 | Ubiquitous | Referential integrity of the ER model | UC-06, UC-07, UC-09, UC-10, UC-12 | COVERED |
| REQ-EVN-001 | Event-driven | Submission creates `Schematic` + dispatches parsing | UC-09, UC-10, UC-11 | COVERED |
| REQ-EVN-002 | Event-driven | Parser success enriches and publishes `Schematic` | UC-09, UC-11 | COVERED |
| REQ-EVN-003 | Event-driven | Comment submission persists a `Comment` | UC-06 | COVERED |
| REQ-STA-001 | State-driven | Unvalidated `Schematic` stays hidden | UC-03, UC-05, UC-11 | COVERED |
| REQ-STA-002 | State-driven | Guests are read-only | UC-02, UC-06, UC-07 | COVERED |
| REQ-OPT-001 | Optional | World archive → `world_zip_url` download | UC-13, UC-14 | COVERED |
| REQ-OPT-002 | Optional | Direct M:N `saves` relation (`User`–`Schematic`) | UC-07 | COVERED |
| REQ-UNW-001 | Unwanted behaviour | Corrupted `.litematica` file rejected | UC-09, UC-11 | COVERED |
| REQ-UNW-002 | Unwanted behaviour | Non-owner edit/delete denied | UC-12, UC-15 | COVERED |
| REQ-OUT-01 | Out of scope | No in-game integration (delivery to Minecraft clients/servers) | — | **UNCOVERED** |

## 2. Reverse mapping — Use Case → EARS Requirements

| Use Case ID | Use Case | Actor | Implemented requirements |
|---|---|---|---|
| UC-01 | Register Account | Guest | REQ-SYS-002 |
| UC-02 | Log In | Guest | REQ-SYS-002, REQ-STA-002 |
| UC-03 | Search Schematics | Guest | REQ-SYS-001, REQ-STA-001 |
| UC-04 | Apply Filters (Category, Tag, Version) | Guest | REQ-SYS-001 |
| UC-05 | View Schematic Details | Guest | REQ-SYS-001, REQ-STA-001 |
| UC-06 | Write Comment | Registered User | REQ-SYS-003, REQ-EVN-003, REQ-STA-002 |
| UC-07 | Save Schematic | Registered User | REQ-SYS-003, REQ-STA-002, REQ-OPT-002 |
| UC-08 | Manage Profile | Registered User | REQ-SYS-002 |
| UC-09 | Publish Schematic | Author | REQ-SYS-003, REQ-EVN-001, REQ-EVN-002, REQ-UNW-001 |
| UC-10 | Classify Schematic (Category, Tag, Version) | Author | REQ-SYS-003, REQ-EVN-001 |
| UC-11 | Parse Litematica File | Litematica Parser Service | REQ-EVN-001, REQ-EVN-002, REQ-STA-001, REQ-UNW-001 |
| UC-12 | Edit Own Publication | Author | REQ-SYS-003, REQ-UNW-002 |
| UC-13 | Attach World Archive (optional) | Author | REQ-OPT-001 |
| UC-14 | Download World Archive (optional) | Guest | REQ-OPT-001 |
| UC-15 | Delete Own Publication (optional) | Author | REQ-UNW-002 |

## 3. Why the mapping is $M:N$

* **One requirement → many use cases:** e.g. REQ-SYS-001 is realised jointly by *Search Schematics* (UC-03), *Apply Filters* (UC-04) and *View Schematic Details* (UC-05); REQ-STA-001 spans *Search*, *View Details* and *Parse Litematica File*.
* **One use case → many requirements:** e.g. *Publish Schematic* (UC-09) simultaneously implements REQ-SYS-003 (integrity invariants), REQ-EVN-001 (submission event), REQ-EVN-002 (parser callback) and REQ-UNW-001 (rejection path).
* Cardinality check: 12 covered requirements ↔ 15 use cases, with 29 requirement↔use-case links — well beyond a $1:1$ mapping.

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

**Legend:** `COVERED` — at least one use case implements the requirement; `UNCOVERED` — intentionally not implemented in the current release (out of scope for future releases).
