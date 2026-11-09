# Ітерація 2:

## Промт
> Act as a software systems modeling assistant. Your previous generation of the UC09 Activity Diagram failed the audit due to broken control flows, missing swimlanes in the render, and pseudo-logic inside text blocks.
>
> Re-generate `lab4/model/uc-09-activity-diagram.puml` and re-render `lab4/model/uc-09-activity-diagram.png` addressing all action items from the audit:
>
> 1. **Strict Swimlane Declaration & Order**:
>    - At the very beginning of the `.puml` script, declare all 6 swimlanes sequentially using empty labels to fix their order from left to right:
>      `|Author|`, `|Frontend|`, `|Backend|`, `|Storage (S3)|`, `|Litematica Parser Service|`, `|Database|`.
>    - Ensure `Database` remains the 6th swimlane on the far right and does not shift to the middle.
>
> 2. **Proper Form Validation Loop (Q4)**:
>    - Replace the broken `if` block with a valid PlantUML `repeat ... repeat while` loop for metadata entry and client validation.
>    - Implement a bounded retry (up to 5 attempts): if validation fails and `attempts < 5`, loop back to form editing; if attempts reach 5, terminate with `stop`.
>
> 3. **Database Responsibility for Session Refresh (Q5)**:
>    - During the publish rights verification, if the session is invalid, route the refresh-token lookup/rotation explicitly into the `|Database|` swimlane (querying the `Session` table per ADR-0007) and return the result to `|Backend|`.
>
> 4. **Optional .zip All-or-Nothing Handling (Q2)**:
>    - Frontend: Validate `.zip` extension only if a file was selected; if invalid, trigger the form validation retry.
>    - Storage: Upload `.zip` if present. If the upload of either `.litematica` or `.zip` fails, mark Branch A as failed.
>
> 5. **Explicit S3 Retry Loop (Q3)**:
>    - Do not use raw text summaries inside action boxes.
>    - Model a concrete retry loop for S3 upload: attempt upload -> on failure, check attempt counter (< 5 attempts with backoff) -> retry; upon exhaustion, mark Branch A as failed.
>
> 6. **Deterministic Fork/Join & Compensation Flow (Q1)**:
>    - Both Branch A (Storage) and Branch B (Parser) must complete and report their status before converging at `end fork`. Remove any misleading "cancel in-flight" notes placed after the join bar.
>    - Immediately after `end fork`, evaluate the combined outcome: if either branch failed, trigger the compensation rollback (delete S3 objects in `|Storage (S3)|`, delete draft record in `|Database|`, return `422 Unprocessable Entity` to `|Frontend|`, and `stop`).
>
> 7. **Visual Layout & Formatting**:
>    - Use appropriate line breaks (`\n`) or formatting parameters to keep labels compact.
>    - Ensure the diagram renders cleanly without excessive horizontal stretching, all 6 swimlanes are fully visible (no cropped lanes), text remains legible, and transition arrows do not overlap awkwardly.

## Контекст
- Файл: `lab4/spec.md` (commit: `<10213906db6475c5a02677d90e5f8e128b1994fc>`)
- Суміжні контексти: `lab3/adr/0007-sessions-storage.md`, `lab2/adr/0005-oauth-for-authoring.md`, `lab2/adr/0006-external-parser-service.md`, `lab1/model/er-diagram.mmd`

## Очікуваний результат
- Оновлено `lab4/model/uc-09-activity-diagram.puml` із виправленою логікою циклів, коректною синхронізацією fork/join та 6 зафіксованими свімлейнами
- Згенеровано читабельний `lab4/model/uc-09-activity-diagram.png`, де присутні всі 6 свімлейнів без обрізань та з розбірливим текстом

## Аудит результатів
- **Статус**: Прийнято частково
- **Хеш коміту згенерованих артефактів**: `<97a79e67fb7bbce3233b06b35a1ee7401914943d>`
- **Згенеровані файли**: 
    - `lab4/model/uc-09-activity-diagram.puml`
    - `lab4/model/uc-09-activity-diagram.png`
- **Огляд результатів**:
    - **`uc-09-activity-diagram.puml`**:
        - **(Q1)**: Прибрано псевдо-скасування, обидва потоки доходять до `end fork`, після чого перевіряється загальний статус та викликається відкат з S3 і БД. 
        - **(Q3)**: Текстову заглушку замінено на цикл `repeat ... repeat while` з лічильником спроб та експоненційною паузою.
        - **(Q2)**: Виправлено.
        - **(Q4)**: Оператор `repeat` стоїть на фронтенді вже після дій автора. Через це при помилці валідації форма не дає користувачеві виправити поля, а 5 разів підряд ганяє перевірку тих самих даних у циклі.
        - **(Q5)**: Звернення до таблиці `Session` тепер правильно йде у свімлейн базт даних.
        - Усі 6 доріжок оголошені на початку.
    - **`uc-09-activity-diagram.png`**: 
        - Доріжки більше не обрізаються, парсер і база даних присутні на схемі.
        - Через переноси ширина стала кращою, але діаграма занадто витягнулася в висоту, тому загальний масштаб зображення все одно малий.
        - В багатьох місцях лінії стрілок налізають одна на одну, а то і на блоки дій.
- **Необхідні зміни**:
    1. **(Q4)**: Поставити оператор `repeat` перед блоком заповнення форми, щоб при невдалій валідації користувач дійсно міг ввести виправлені дані, а не зациклював фронтенд на одній помилці.
    2. Зменшити значення `skinparam ranksep`, щоб прибрати зайву висоту полотна і зробити шрифт при перегляді більшим та зручнішим для читання.
    3. Позбутись налізання стрілок одна на одну та на блоки дій.