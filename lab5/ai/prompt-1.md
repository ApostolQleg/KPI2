# Ітерація 1:

## Промт
> Act as a software systems architecture assistant. Your task is to analyze `lab5/spec.md` and `lab5/adr/0008-modular-monolith-and-system-boundaries.md` (which serve as the single source of truth for Task 5), verify their alignment with previous artifacts (`lab1` ERD, `lab2` Use Cases & ADRs 0005–0006, `lab3` Sequence diagrams & ADR-0007, `lab4` Activity diagram), and produce a UML Component Diagram in PlantUML syntax.
>
> Execute this task in two strictly separated, sequential phases:
>
> Phase 1: Critical Analysis & Clarifications (STOP & ASK FIRST)
> - Review all components, system boundaries, internal layers, external integrations, and provided/required interfaces defined in `lab5/spec.md`.
> - Check for traceability gaps against Labs 1–4 (e.g., how `Litematica Parser Service` accesses the `.litematica` binary via `Message Queue` vs `S3`, how `UC09` rollback/timeout interacts with the queue, whether `Web UI` needs direct `S3` download links for `UC05`/`UC14`, or how `Microsoft OAuth` browser redirect involves `Web UI`).
> - Do NOT generate any PlantUML code yet. Stop execution, list concise clarifying questions in English with concrete trade-off options, and wait for my answers.
>
> Phase 2: Declarative Model Generation & Render (AFTER CLARIFICATIONS)
> - Once I answer your questions, generate `lab5/model/component-diagram.puml` strictly using PlantUML Component Diagram syntax.
> - Show a clear system boundary (`SchemaPostol`) containing `Web UI (SPA)`, the `Backend` modular monolith (`REST API`, `Auth & Session Service`, `Schematic & Catalog Service`, `Community Service`, `Data Access Layer`), `PostgreSQL Database`, and `Message Queue`.
> - Place external integrations strictly OUTSIDE the system boundary: `Microsoft OAuth`, `Object Storage (S3)`, and `Litematica Parser Service`.
> - Explicitly model UML provided and required interfaces (lollipop/socket notation or named interface nodes) and dependencies between components. Do NOT allow any service or UI to bypass `Data Access Layer` to reach `PostgreSQL Database`.
> - Do not invent out-of-scope components (no Payments, SMS, Email, Redis, or in-game integrations).
> - Optimize layout directives (`skinparam`, direction, package grouping, concise multiline labels) so the auto-rendered diagram has zero overlapping arrows over text and readable typography.
> - Render `lab5/model/component-diagram.png` directly from the `.puml` file.

## Контекст
- Файл: `lab5/spec.md` (commit: `a5dbe358ee058d22be2efc124d22a3af2face5b9`)
- Файл: `lab5/adr/0008-modular-monolith-and-system-boundaries.md` (commit: `8dbbd112811cc43583ec3b97396381e0c8789cd6`)
- Суміжні контексти: `lab1/model/er-diagram.mmd`, `lab2/model/use-case-diagram.puml`, `lab2/adr/0005-oauth-for-authoring.md`, `lab2/adr/0006-litematica-parsing.md`, `lab3/model/01-auth.puml`, `lab3/model/02-oauth.puml`, `lab3/model/03-token-refresh.puml`, `lab3/adr/0007-sessions-storage.md`, `lab4/model/uc-09-activity-diagram.puml`

## Очікуваний результат
- Фаза 1: отримано список уточнювальних питань щодо архітектурних меж, інтерфейсів та взаємодії через чергу й S3
- Фаза 2: після відповідей згенеровано `lab5/model/component-diagram.puml` та якісно відрендерений `lab5/model/component-diagram.png` без накладань ліній на блоки і подібного

## Отриманий результат:
<details>
<summary>Відповідь агента</summary>
## Phase 1 — Critical Analysis & Clarifications (STOP & ASK FIRST)

I've reviewed all single-source-of-truth and prior artifacts. Below is my alignment review, the traceability gaps I found, and clarifying questions. **No PlantUML is produced yet** — I'm waiting for your answers before Phase 2.

---

### Artifacts reviewed

| Lab | Artifact | Key content relevant to the component diagram |
|-----|----------|-----------------------------------------------|
| lab1 | `model/er-diagram.mmd` | `User`, `Schematic`, `Comment`, `Category`, `Tag`, `Version`; `Schematic.litematica_url`, `world_zip_url`; M:N `saves` |
| lab2 | `model/use-case-diagram.puml` | UC-01…UC-15; **UC-11 `Parse Litematica File` is an external background service, NOT an actor**, reached via `<<delegate>>`, outside system boundary |
| lab2 | `adr/0005` | Microsoft OAuth gates `Author` elevation (`isAuthor`); browser redirect flow |
| lab2 | `adr/0006` | Async `Litematica Parser Service` + **Message Queue**; lab4 skipped the queue for simplicity |
| lab2 | `requirements/traceability-matrix.md` + `ears.md` | REQ-EVN-001/002 (dispatch parsing + publish on success), REQ-STA-001 (unvalidated hidden), REQ-OPT-001 (world archive download), REQ-OUT-01 (no in-game) |
| lab3 | `adr/0007` | `Session` table in **PostgreSQL** (no Redis); JWT accessToken 15 min + refreshToken 7 days HttpOnly cookie |
| lab3 | `model/02-oauth.puml` | **Browser redirect passes through Web UI (Frontend)**: `FE → MS` redirect, `MS → FE` redirect with code, then `FE → BE` callback |
| lab4 | `model/uc-09-activity-diagram.puml` | UC09 fork/join: S3 upload w/ retry + parser call directly (no queue); **rollback = delete S3 objects + delete DB draft**; **no queue modeled** |
| lab5 | `spec.md` + `adr/0008` | **Chosen: Option 1** — modular monolith + internal `Message Queue`; `Microsoft OAuth`, `S3`, `Litematica Parser Service` **outside** boundary; explicit provided/required interface list |

---

### Alignment verification (what checks out)

- **Lab1 ERD → Data Access Layer**: DAL encapsulates repositories for `User`, `Session`, `Schematic`, `Comment`, `Category`, `Tag`, `Version`. ✓ Consistent with spec §Backend.
- **Lab2 UC-11 / ADR-0006 → external parser outside boundary**: spec places `Litematica Parser Service` strictly outside `SchemaPostol`, reached via `IParseJobQueue`. ✓ Consistent.
- **Lab2 ADR-0005 → Auth & Session Service → `IMicrosoftOAuthApi` ← Microsoft OAuth**. ✓ Consistent.
- **Lab3 ADR-0007 → PostgreSQL-only sessions**: no Redis in scope (matches "Out of Scope" — no Redis). ✓ Consistent.
- **ADR-0008 decision = Option 1** (modular monolith, internal queue, external integrations) is what the diagram must render. ✓ Settled, no question needed.

---

### Traceability gaps / ambiguities identified

1. **Parser binary access — S3 link vs queue payload**  
   spec.md says `Litematica Parser Service → requires IS3StorageApi (reading file by link OR getting payload from queue)` — the "or" is ambiguous. Lab4's activity diagram shows Backend uploads to S3 *first*, then hands the binary to the parser directly (no queue). So does the parser fetch `.litematica` from S3 via a URL carried in the queue job, or does the queue message carry the binary payload itself?

2. **UC09 rollback/timeout vs the Message Queue**  
   ADR-0008 admits the queue "requires handling timeouts waiting for a response from the external parser." Lab4 models fork/join rollback (delete S3 objects + delete draft → 422) but with **no queue**. Is the queue request/reply (correlation ID + reply-to + timeout, rollback triggered on timeout/parser failure), or fire-and-forget with a separate result message? Where does compensation rollback trigger from in the component model?

3. **Web UI → direct S3 download for UC-05 / UC-14?**  
   UC-05 (View Schematic Details, `litematica_url`) and UC-14 (Download World Archive, `world_zip_url`, Guest) need the binary reach the browser. The spec's interface list only has `Schematic & Catalog Service → IS3StorageApi ← S3` (upload direction implied). Does Web UI get pre-signed URLs and fetch directly from S3 (requires a download interface on Web UI), or does REST API proxy the download (Web UI only → `IRestApi`)?

4. **Microsoft OAuth browser redirect — does Web UI need a front-channel interface?**  
   Lab3 `02-oauth.puml` shows `Web UI (Frontend) → Microsoft OAuth` redirect and `Microsoft OAuth → Web UI` redirect-with-code, i.e. the user-agent redirect passes through Web UI. The spec interface list only models `Auth & Session Service → IMicrosoftOAuthApi ← Microsoft OAuth` (server-side back-channel token exchange). Should the diagram add a separate Web UI front-channel interface to Microsoft OAuth, or treat the redirect as runtime behavior and model only the back-channel?

5. **Naming consistency**  
   spec.md uses "Auth & Session Service" (interfaces) vs "Auth Service" (component list); "Schematic & Catalog Service" vs "Schematic Service"; "Community Service" vs "Comment Service". Also: is `REST API` a standalone component consuming `IAuthService`/`ISchematicService`/`ICommunityService`, or a thin facade inside those services? The diagram labels should be consistent.

---

### Clarifying questions (with trade-off options)

**Q1 — Parser binary transport:** How does `Litematica Parser Service` obtain the `.litematica` binary?
- **A)** Queue job carries `schematic_id` + S3 URLs; Parser fetches the file from S3 via `IS3StorageApi` (matches lab4's S3-first upload).
- **B)** Queue message carries the binary payload directly; Parser does not read S3 for the file (`IS3StorageApi` only for optional world archive / not needed).
- **C)** Hybrid: job has an S3 URL; Parser reads from S3 and publishes the metadata result back to the queue.

**Q2 — UC09 queue interaction & rollback:** Model the Message Queue as request/reply or fire-and-forget?
- **A)** Request/reply with correlation ID + reply-to + timeout; Parser publishes result to queue; on timeout/parser failure → compensation rollback (S3 delete + draft delete, `is_published = false`). Explicit timeout/rollback edges on the diagram.
- **B)** Fire-and-forget job submission; result arrives via a separate result message consumed by Backend → DAL; rollback triggered by result status. Simpler queue semantics, slower failure detection.
- **C)** Fire-and-forget only, no reply channel modeled; rollback is purely a backend concern (lab4-style), queue treated as unreliable transport.

**Q3 — Web UI ↔ S3 for UC-05/UC-14 downloads:** Does Web UI download directly from S3?
- **A)** Yes: Web UI `requires IS3DownloadApi ← S3` (browser fetches pre-signed URLs returned by REST API); keeps backend out of the binary data path.
- **B)** No: REST API proxies all downloads; Web UI only `requires IRestApi ← REST API`; all S3 access behind `Schematic & Catalog Service`.
- **C)** Split interface: Web UI `requires IS3DownloadApi` (read/download) and `Schematic & Catalog Service` `requires IS3StorageApi` (write/upload) — two distinct interfaces to S3.

**Q4 — Microsoft OAuth front-channel through Web UI:** Model Web UI → Microsoft OAuth?
- **A)** Yes: add Web UI `requires IMicrosoftOAuthRedirect ← Microsoft OAuth` (browser redirect channel) **plus** Auth & Session Service `requires IMicrosoftOAuthApi ← Microsoft OAuth` (back-channel token exchange).
- **B)** No: model only `Auth & Session Service → IMicrosoftOAuthApi ← Microsoft OAuth`; document the Web UI redirect as a runtime note (no interface node).
- **C)** Merge into one `IMicrosoftOAuthApi` on Auth & Session Service, with a note that the user-agent redirect is brokered by Web UI.

**Q5 — Naming:** Adopt spec's interface-section names consistently (`Auth & Session Service`, `Schematic & Catalog Service`, `Community Service`, `REST API` as separate facade component)? Or align with the shorter component-list names (`Auth Service`, `Schematic Service`, `Comment Service`)?

---

### What happens next (Phase 2)

Once you answer Q1–Q5, I will:
1. Generate `lab5/model/component-diagram.puml` with explicit `SchemaPostol` system boundary containing `Web UI (SPA)`, the backend modular monolith (`REST API`, `Auth & Session Service`, `Schematic & Catalog Service`, `Community Service`, `Data Access Layer`), `PostgreSQL Database`, and `Message Queue`; external `Microsoft OAuth`, `Object Storage (S3)`, `Litematica Parser Service` strictly outside; UML provided/required interfaces (lollipop/socket notation) and dependencies; **no service/UI bypasses DAL to reach PostgreSQL**; no out-of-scope components (no Payments, SMS, Email, Redis, in-game).
2. Apply `skinparam`/layout directives for zero-overlapping arrows and readable typography.
3. Render `lab5/model/component-diagram.png` directly from the `.puml`.

**I'm waiting for your answers to Q1–Q5 before writing any PlantUML.**
</details>

---

**Мої відповіді:** 
- **Q1**: C
- **Q2**: A
- **Q3**: C
- **Q4**: A
- **Q5**: Use the shorter component names consistently everywhere: `REST API` (as a standalone facade component inside the Backend modular monolith), `Auth Service`, `Schematic Service`, `Comment Service`, and `Data Access Layer`, with matching interfaces `IAuthService`, `ISchematicService`, `ICommentService`.

## Аудит результатів
- **Статус**: Не прийнято
- **Хеш коміту згенерованих артефактів**: `0017cea8bb7279039c17685695fade0c9537f623`
- **Згенеровані файли**: 
  - `lab5/model/component-diagram.puml`
  - `lab5/model/component-diagram.png`
- **Огляд результатів**:
  - **`component-diagram.puml`**:
    - Склад компонентів, короткі назви (`Q5`), межа системи `SchemaPostol` і доступ до `PostgreSQL` виключно через `Data Access Layer` відповідають `spec.md`.
    - інтерфейси підключені до провайдерів як глухі гілки (`--> : provides`), а споживачі з'єднані напряму з компонентами (`--> : requires`) в обхід самих інтерфейсів.
    - усі 11 `interface` оголошено за межами `rectangle "SchemaPostol"`, через що `IRestApi`, `IAuthService`, `ISchematicService`, `ICommentService`, `IDataAccess`, `ISqlDriver` та `IParseJobQueue` випали назовні.
    - **Помилка по `Q1`/`Q3`**: для `Litematica Parser Service` вказано `IS3StorageApi` замість read-only `IS3DownloadApi`.
    - **Пропущено `Q2`**: немає нотатки про request/reply (`correlation_id`), таймаут черги та компенсаційний відкат у S3 і БД.
  - **`component-diagram.png`**:
    - Биті символи в заголовку.
    - Через винесені назовні інтерфейси сині стрілки пробивають межу системи в усі боки.
    - Написи накладаються один на одного (`requires IDataAccessrequires IDataAccess`) та на рамки блоків (`requires IRestApi`, `requires ICommentService`).
    - Стрілки ведуть в нікуди.
- **Необхідні зміни**:
  1. **Виправити UML-зв'язки інтерфейсів**: прибрати прямі стрілки між компонентами та з'єднати їх через самі інтерфейси (провайдер надає інтерфейс суцільною лінією `Component -- Interface`, споживач використовує його через socket `Consumer -( Interface` або залежність `Consumer ..> Interface`).
  2. **Повернути внутрішні інтерфейси в межу системи**: оголосити `IRestApi`, `IAuthService`, `ISchematicService`, `ICommentService`, `IDataAccess`, `ISqlDriver`, `IParseJobQueue` усередині `SchemaPostol` (у відповідних блоках), а зовні лишити тільки інтерфейси зовнішніх систем (`IMicrosoftOAuthApi`, `IMicrosoftOAuthRedirect`, `IS3StorageApi`, `IS3DownloadApi`).
  3. **Виправити інтерфейс парсера (`Q1`/`Q3`)**: підключити `Litematica Parser Service` до read-only `IS3DownloadApi` замість `IS3StorageApi`.
  4. **Додати нотатку (`Q2`)**: біля `Message Queue` або `Schematic Service` додати коротку нотатку про request/reply (`correlation_id`), таймаут і компенсаційний відкат (видалення файлів у S3 + видалення чернетки в БД).
  5. **Виправити рендер**: замінити `—` у заголовку на звичайний ASCII `-` та прибрати перетини/злипання підписів (перехід на коректні зв'язки через інтерфейси прибере дублювання написів `provides`/`requires` і суттєво розвантажить схему).