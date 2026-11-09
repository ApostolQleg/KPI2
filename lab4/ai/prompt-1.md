# Ітерація 1:

## Промт
> Act as a software systems modeling assistant. Your task is to analyze the provided specification `lab4/spec.md` (which serves as the absolute single source of truth) for the business process `UC09: Publish Schematic` and produce a corresponding PlantUML Activity Diagram using the Beta syntax.
>
> Execute this task in two strictly separated, sequential phases:
>
> Phase 1: Critical Analysis & Clarifications (STOP & ASK FIRST)
> - Perform a critical review of the control flows in `lab4/spec.md`.
> - Identify any architectural bottlenecks, potential concurrency deadlocks around the `fork`/`join` synchronization bar (such as S3 upload failures vs. parser failures), validation loops, or optional branches (`world_zip_url`).
> - Do NOT generate the diagram code immediately. Stop execution, formulate precise clarifying questions strictly in English, and present concrete trade-off options for each ambiguity. Await my answers before proceeding.
>
> Phase 2: Declarative Model Generation (AFTER CLARIFICATIONS)
> - Once I provide answers to your clarifying questions, generate the complete PlantUML code strictly incorporating the resolved decisions.
> - Adhere to the 6 swimlanes (`Author`, `Frontend`, `Backend`, `Storage (S3)`, `Litematica Parser Service`, `Database`), conditional decision nodes, parallel execution paths, and terminal points (`stop`).
> - Use new PlantUML Activity Beta syntax (`|Swimlane|`, `start`, `stop`, `if (...) then (...) else (...) endif`, `fork ... fork again ... end fork`). Ensure all control paths converge deterministically without deadlocks.
> - Do not invent domain entities, attributes, or business rules not present in `lab4/spec.md` or its referenced artifacts (`lab2`, `lab3`, ADRs).
> - Render generated file into `.png` format.

## Контекст
- Файл: `lab4/spec.md` (commit: `<10213906db6475c5a02677d90e5f8e128b1994fc>`)
- Суміжні контексти: `lab2/model/use-case-diagram.puml`, `lab3/model/02-oauth.puml`, `lab2/adr/0005-oauth-for-authoring.md`, `lab2/adr/0006-external-parser-service.md`

## Очікуваний результат
- Отримано список уточнювальних запитань щодо вузьких місць, обробки збоїв та поведінки синхронізації на вузлі `join`
- Після надання відповідей — згенеровано файл `lab4/model/uc-09-activity-diagram.puml` із повною реалізацією діаграми згідно з уточненими вимогами

## Отриманий результат:
<details>
<summary>Відповідь агента</summary>

I now have full context. I've reviewed `lab4/spec.md` plus all referenced artifacts: `lab2` (use-case diagram with UC09/UC10/UC11/UC13, ADR-0005 OAuth, ADR-0006 parser), `lab3` (session/token refresh, ADR-0007), and `lab1` (ERD: `Schematic.world_zip_url` optional). 

Below is **Phase 1: Critical Analysis & Clarifications**. Per your instructions, I am **stopping here** and will not generate any PlantUML code until you answer.

---

# Phase 1 — Critical Analysis & Clarifications (UC09: Publish Schematic)

## Baseline flow understood (from `lab4/spec.md`)

1. Author → Frontend: open publish page.
2. Frontend → Backend: check publish rights; Backend validates auth + `isAuthor`.
   - Fail → `403 Forbidden`, Frontend blocks form, offers Microsoft OAuth, terminal failure.
   - Pass → `200 OK`, show upload form.
3. Author fills metadata + `.litematica` + optional `.zip`; clicks Publish.
4. Frontend client-side validation (extensions + required fields).
   - Fail → show errors, return to form fill (loop).
   - Pass → `POST /api/schematics` (multipart) to Backend.
5. Backend validates MIME/size/classification links.
   - Fail → `400 Bad Request`, terminal failure.
6. Backend generates `slug`, creates DB draft (`is_published = false`), **forks** into:
   - Branch A (Storage): upload `.litematica` + optional `.zip` to S3 → S3 returns `schematic_url`, `world_zip_url`.
   - Branch B (Parser): send binary to Litematica Parser Service → parse NBT, checksums, dimensions, block count, materials → return JSON metadata.
7. Backend **joins** both branches, then analyzes parser output.
   - Parser error → delete S3 objects, delete DB draft, `422 Unprocessable Entity`, terminal failure.
   - Success → write URLs + final metadata + `is_published = true` to DB (commit), `201 Created`, Frontend redirects to UC05, terminal success.

## Critical issues identified

### Q1 — Fork/join failure asymmetry & deadlock risk (S3 upload vs. parser failure)

**Ambiguity:** The spec only defines an error path for **parser failure** (corrupted NBT → cleanup S3 + delete draft → `422`). It says nothing about:
- **Branch A (S3 upload) failing** while Branch B (parser) succeeds, or vice versa.
- One branch **hanging** (e.g., Parser Service down / slow NBT parse) while the other completes.
- Whether the `join` is a hard barrier (both must complete) or whether a failed branch can be cancelled.

**Deadlock risk:** If `join` is a hard barrier with no timeout / no cancel-sibling semantics, a single hung or silently-failed branch leaves the Backend blocked indefinitely — violating the "no deadlocks at join" acceptance criterion.

| Option | Description | Trade-offs |
|--------|-------------|------------|
| **A. Fail-fast with sibling cancellation** | If either branch fails, abort the other branch, roll back everything (S3 + DB draft), terminate with an error terminal. | Cleanest deadlock avoidance; matches the spec's "rollback on parser error" spirit. Requires modelling cancellation of the in-flight sibling (not explicitly in spec). |
| **B. Independent completion + combined outcome** | Each branch reports success/failure to a shared state; `join` proceeds once both branches report (even if one failed); Backend then decides based on the combined result. | Avoids hard deadlock; no cancellation needed. More complex state merging; one branch can succeed while the other fails, requiring a partial-failure decision at the join. |
| **C. Sequentialize (drop the fork)** | Do S3 upload then parser (or vice versa) in sequence. | Eliminates join deadlock entirely, but **contradicts the spec's mandatory parallel fork** and the acceptance criterion requiring `fork`/`join`. Not recommended. |

**My provisional lean:** Option B (or A) — the diagram must model a non-blocking join with explicit failure handling for *both* branches, not just the parser.

### Q2 — Optional `world_zip_url` (UC13) handling

**Ambiguity:** The world `.zip` is optional (REQ-OPT-001: `world_zip_url` stays empty if absent; ERD marks it optional). Unclear:
- Client-side: is the zip validated only **if present** (skip if absent), or does selecting an invalid zip fail the whole form (loop back to step 4)?
- Branch A (S3): if the `.zip` is absent, does Branch A still succeed with `world_zip_url = NULL`? If the `.zip` upload fails but the `.litematica` upload succeeds — proceed with `world_zip_url = NULL`, or abort the whole publication?
- Does the parser ever touch the `.zip`? (Spec says parser parses the `.litematica` binary only, so the zip is storage-only.)

| Option | Description | Trade-offs |
|--------|-------------|------------|
| **A. Zip validated only if present; absent → NULL; partial upload failure → proceed with NULL `world_zip_url`** | Most permissive; publication succeeds without the archive. | Simpler diagram; but a silently-missing archive may be undesirable. |
| **B. All-or-nothing: invalid/present-but-bad zip fails client validation (loop to step 4); S3 zip failure aborts whole branch → rollback + error** | Stricter consistency; no partial publications. | More decision nodes; risks rejecting a valid `.litematica` because of a bad optional zip. |
| **C. Zip validated if present; S3 zip failure → abort whole publication (rollback + error)** | Middle ground. | Publication requires both files if a zip was selected. |

**My provisional lean:** Option A for storage (proceed with `world_zip_url = NULL` if zip absent or its upload fails), with client-side validating the zip extension only when a file is selected.

### Q3 — S3 upload failure handling (uncovered by the spec)

**Ambiguity:** The spec's cleanup path (delete S3 objects + delete DB draft) only triggers on **parser error**. If Branch A's S3 PUT fails (network, permissions, bucket error), there is no defined path. Does S3 failure:
- Trigger the same rollback as parser failure (delete any already-uploaded objects, delete DB draft, return error)?
- Get retried (and how many times)?
- Cause the `join` to wait forever (deadlock)?

This is the other half of Q1's deadlock risk but specific to storage.

### Q4 — Client-side validation loop bounds (step 6)

**Ambiguity:** On client-side validation failure, the process "returns to step 4" (refill form). The spec implies an **unbounded** loop.
- Should the diagram model an unbounded loop (merge back to form fill), or a capped retry (e.g., N attempts then terminal failure)?
- No max-retry rule exists in `lab4/spec.md`, `lab2`, or ADRs.

### Q5 — Auth / session handling at the rights check (step 3)

**Ambiguity:** Spec step 3 says: invalid session → `403 Forbidden`, offer Microsoft OAuth, terminal failure. However, `lab3` (Scenario 3) and ADR-0007 describe access-token expiry → refresh-token rotation → new tokens; only if refresh fails does the user become a Guest.
- Does the rights-check endpoint attempt a refresh-token rotation before returning `403` (per `lab3`/ADR-0007), or is `403` immediate as literally written in `lab4/spec.md`?
- `lab4/spec.md` is the "absolute single source of truth" for this process, but `lab3`/ADR-0007 are referenced artifacts — so there is a potential tension.

| Option | Description | Trade-offs |
|--------|-------------|------------|
| **A. Immediate `403`** (literal `lab4/spec.md` step 3) | Invalid session → `403` + offer OAuth + terminal failure. No refresh attempt. | Simplest; strictly follows `lab4/spec.md`. Slightly less realistic vs. `lab3`. |
| **B. Refresh first, then `403`** (per `lab3`/ADR-0007) | On invalid access token, attempt refresh; only `403` if refresh fails / user becomes Guest. | More realistic and consistent with `lab3`/ADR-0007, but adds behavior/branches not in `lab4/spec.md` step 3. |

**My provisional lean:** Option A (immediate `403`) because `lab4/spec.md` is the single source of truth for UC09 and explicitly defines this terminal path — but I want your call given `lab3`/ADR-0007.

---

## Constraints I will honor in Phase 2 (once you answer)

- **6 swimlanes exactly:** `Author`, `Frontend`, `Backend`, `Storage (S3)`, `Litematica Parser Service`, `Database`.
- **PlantUML Activity Beta syntax only:** `|Swimlane|`, `start`, `stop`, `if (...) then (...) else (...) endif`, `fork … fork again … end fork`.
- **At least 3 decision nodes:** `isAuthor` check, payload-validation result, parser-integrity result.
- **One `fork`/`join` parallel region** (Branch A storage + Branch B parser).
- **Explicit `stop` terminals** for each failure (access, validation, parser/S3 error) and one success terminal.
- **No invented entities/attributes/rules:** only `slug`, `is_published`, `schematic_url`, `world_zip_url`, dimensions/block count/materials (parser output), `403`/`400`/`422`/`201`, UC05 redirect — all from `lab4/spec.md`, ERD, and ADRs.
- **Deterministic convergence:** every path reaches a `stop`; no orphaned branches at `join`.
- **Output:** `lab4/model/uc-09-activity-diagram.puml` + rendered `.png` (Phase 2 only).

---

Please provide your answers to Q1–Q5 (you can pick an option per question, or propose a variant). Once I have your decisions, I'll proceed to Phase 2: generate the complete PlantUML Beta diagram and render it to PNG.
</details>

---

**Мої відповіді:** 
- **Q1**: A
- **Q2**: B
- **Q3**: Retry 5 times with increasing time between queries (starting with 5 seconds, and x2 each retry), then rollback
- **Q4**: Capped retry with 5 attempts
- **Q5**: B

## Аудит результатів
- **Статус**: Не прийнято
- **Хеш коміту згенерованих артефактів**: `<3d37be466bfba5ae1492fa3dab3ccd8446dd3bd0>`
- **Згенеровані файли**: 
    - `lab4/model/uc-09-activity-diagram.puml`
    - `lab4/model/uc-09-activity-diagram.png`
- **Огляд результатів**:
   - **`uc-09-activity-diagram.puml`**: 
      - Агент лише частково взяв до уваги мої відповіді на його уточнення. 
         - **(Q1)**: Скасування гілок стоїть після `end fork`. Але `end fork` чекає на обидва потоки, тому завислу гілку перервати буде неможливо.
         - **(Q2)**: Немає клієнтської перевірки вибраного архіву, а помилка його завантаження в S3 не зупиняє публікацію.
         - **(Q3)**: 5 спроб відправки в S3 та очікування 5s * 2^n вписані просто коментарем у дію замість повноцінного циклу.
         - **(Q4)**: Замість циклу `repeat ... repeat while` використано звичайний `if`. Через це невдала перевірка не повертає на повторне заповнення, а веде стрілку в самий низ.
         - **(Q5)**: Ротація сесій у таблиці `Session` намальована на бекенді без звернення до свімлейну бази даних.
      - Доріжки не зафіксовані спочатку, тому через запис чернетки свімлейн `Database` став четвертим замість шостого.
   - **`uc-09-activity-diagram.png`**: 
      - Через неправильний порядок і величезну ширину полотна на рендері лишилось лише чотири свімлейни з шести.
      - Шрифт став мікроскрпічним, написи на стрілках неможливо розібрати.
- **Необхідні зміни**:
   1. На початку файлу порожніми блоками оголосити всі 6 доріжок: `Author`, `Frontend`, `Backend`, `Storage (S3)`, `Litematica Parser Service`, `Database`.
   2. **(Q1)**: Завершувати обидві гілки перед `end fork`. Після бару перевіряти спільний статус і запускати відкат з бд і s3, якщо сталася помилка.
   3. **(Q2)**: Валідувати `.zip` на клієнті, якщо файл обрано; збій завантаження архіву в S3 має зупиняти процес і відкочувати транзакцію.
   4. **(Q3)**: Замість тексту побудувати цикл: спроба → пауза → лічильник до 5 спроб → статус збою.
   5. **(Q4)**: Замінити `if` на `repeat ... repeat while` з лічильником до 5 спроб для повернення до полів форми.
   6. **(Q5)**: Показати звернення бекенду до таблиці `Session` окремою стрілкою у свімлейн бази даних.
   7. Для гарного рендеру додати автоперенесення рядків у тексті й налаштувати відступи, щоб діаграма була компактною, шрифт можна було прочитати, стрілки не налізали одна на одну, а також було видно усі 6 свімлейнів.