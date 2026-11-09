# Ітерація 4:

## Промт
> Update `lab4/model/uc-09-activity-diagram.puml` and re-render `lab4/model/uc-09-activity-diagram.png` addressing the structural syntax defect and visual collisions identified in the Iteration 3 audit:
>
> 1. **Eliminate Redundant Decision Diamond in Validation Loop**:
>    - Remove the redundant `if (Client validation failed?)` decision diamond immediately following `repeat while`.
>    - Embed the bounded limit check directly inside the validation failure branch:
>      ```plantuml
>      if (Client validation failed?) then (yes)
>        :Display form errors;\nreturn control to Author;
>        :Increment validation_attempts;
>        if (validation_attempts >= 5?) then (yes)
>          stop
>        endif
>      endif
>      repeat while (Client validation failed?) is (yes) not (no)
>      :Send POST /api/schematics\n(multipart/form-data) to Backend;
>      ```
>    - Ensure the exit arrow labeled `(no)` transitions directly to `Send POST /api/schematics` without stacked diamonds.
>
> 2. **Eliminate Arrow Crossings in Session Verification & S3 Loop**:
>    - **Session check**: Restructure the refresh flow so that returning from `|Database|` to `|Backend|` does not cross through the vertical line of the already-valid session.
>    - **S3 Retry loop**: Keep the upload status evaluation (`if (Upload failed?)`) inside `|Storage (S3)|` or streamline it on `|Backend|` so the loop-back arrow does not slice through the attempt incrementing path.
>
> 3. **Disentangle Post-Join Routing (Rollback vs. Commit)**:
>    - Cleanly separate the failure rollback path (`|Storage (S3)|` delete objects -> `|Database|` delete draft -> `422`) from the success commit path (`|Database|` commit -> `201`).
>    - Ensure transitions spanning multiple swimlanes have distinct vertical offsets so horizontal lines do not merge, overlap, or clip action boundaries.
>
> 4. **Visual Geometry & Orthogonal Routing**:
>    - Add explicit branch labels `(yes)` and `(no)` to all decision diamonds to guide PlantUML's branch placement.
>    - Set `skinparam linetype ortho` to force clean 90-degree connector routing, preventing diagonal lines from slicing through boxes.
>    - Set `skinparam ranksep 28` and `skinparam nodesep 24` to provide enough clearance for loop return arrows.
>    - Keep label widths strictly constrained with `\n` (max 25–30 characters per line) to keep swimlane widths compact and uniform.
>
> 5. **Preserve Verified Architecture**:
>    - Keep the 6 declared swimlanes in exact order: `|Author|`, `|Frontend|`, `|Backend|`, `|Storage (S3)|`, `|Litematica Parser Service|`, `|Database|`.
>    - Retain all business rules: ADR-0007 token rotation, S3 retry policy (5 attempts, $5s \times 2^n$), All-or-Nothing handling for `.zip`, and post-join compensation.
>    - Re-render `lab4/model/uc-09-activity-diagram.png` ensuring all 6 swimlanes are fully visible, readable, and free of collisions.

## Контекст
- Файл: `lab4/spec.md` (commit: `<6f1d3aa93cebfbfd3903c523914f08cd8bc1f009>`)
- Файл: `lab4/model/uc-09-activity-diagram.puml` (commit: `<43dd45552afe69fa4bde2a1cc296bb9010f6a05d>`)
- Суміжні контексти: `lab3/adr/0007-sessions-storage.md`, `lab2/adr/0005-oauth-for-authoring.md`, `lab2/adr/0006-external-parser-service.md`, `lab1/model/er-diagram.mmd`

## Очікуваний результат
- `lab4/model/uc-09-activity-diagram.puml` оновлено (прибрано спарені ромби валідації, оптимізовано переходи стрілок та налаштовано ортогональне трасування)
- `lab4/model/uc-09-activity-diagram.png` оновлено (відсутні перетинання та накладання ліній на блоки дій, усі 6 свімлейнів повністю видимі, схема компактна та легко читається)

## Аудит результатів
- **Статус**: Прийнято частково
- **Хеш коміту згенерованих артефактів**: `<26d89b111dd2a010ad867de173161a9214f40df3>`
- **Згенеровані файли**: 
    - `lab4/model/uc-09-activity-diagram.puml`
    - `lab4/model/uc-09-activity-diagram.png`
- **Огляд результатів**:
    - **`uc-09-activity-diagram.puml`**:
        - Повністю виконано всі вимоги специфікації `spec.md` та попередніх промтів.
        - Прибрано спарені ромби, перевірку ліміту спроб вбудовано в гілку помилки, а вихід із циклу одразу веде до відправки запиту на бекенд.
        - Розставлено мітки переходів `(yes)` та `(no)` на всіх розгалуженнях умов.
    - **`uc-09-activity-diagram.png`**:
        - Агент не справляється з накладаннями стрілок одна на одну.
- **Необхідні зміни**:
    1. Виконати ручне коригування рендеру, щоб усунути артефакти автоматичного генерування та отримати більш охайне фінальне зображення `uc-09-activity-diagram.jpg`.