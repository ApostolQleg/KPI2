# Ітерація 3:

## Промт
> Update `lab4/model/uc-09-activity-diagram.puml` and re-render `lab4/model/uc-09-activity-diagram.png` addressing all remaining action items from the Iteration 2 audit:
>
> 1. **Fix Form Retry Loop Placement (Q4)**:
>    - Move the opening `repeat` keyword before the Author's form interaction (`|Author| : Fill metadata...`).
>    - When client validation fails, ensure the flow returns control back to the Author so they can actually edit inputs and re-submit, rather than looping idly on the Frontend.
>    - Preserve the bounded retry condition (`validation_attempts < 5`) and terminal `stop` upon reaching the limit.
>
> 2. **Eliminate Arrow & Action Collisions**:
>    - Resolve layout overlaps where return arrows or swimlane transitions cross through action blocks or intersect other connector lines.
>    - Ensure incoming and outgoing connectors at decision diamonds and loop-back branches route cleanly through designated swimlanes.
>
> 3. **Compact Vertical Spacing & Enhance Font Readability**:
>    - Reduce vertical separation by tuning skinparams (e.g. decrease `skinparam ranksep` to ~22–26, adjust `nodesep` and `padding`).
>    - Keep multiline text labels compact using `\n` to reduce total diagram height, allowing the rendered `.png` image to scale at a comfortable, readable font size.
>
> 4. **Preserve Validated Architecture & Swimlane Order**:
>    - Maintain the explicit initial declaration of all 6 swimlanes in exact sequence: `|Author|`, `|Frontend|`, `|Backend|`, `|Storage (S3)|`, `|Litematica Parser Service|`, `|Database|`.
>    - Keep the verified business flows intact: Session token rotation querying `|Database|` (ADR-0007), S3 5-attempt retry loop with exponential backoff ($5s \times 2^n$), All-or-Nothing handling for optional `.zip`, and post-join compensation rollback (cleanup in S3 and Database).
>
> 5. Re-render `lab4/model/uc-09-activity-diagram.png` ensuring all 6 swimlanes are fully visible, no elements overlap, and all labels are distinct and legible.

## Контекст
- Файл: `lab4/spec.md` (commit: `<6f1d3aa93cebfbfd3903c523914f08cd8bc1f009>`)
- Файл: `lab4/model/uc-09-activity-diagram.puml` (commit: `<97a79e67fb7bbce3233b06b35a1ee7401914943d>`)
- Суміжні контексти: `lab3/adr/0007-sessions-storage.md`, `lab2/adr/0005-oauth-for-authoring.md`, `lab2/adr/0006-external-parser-service.md`, `lab1/model/er-diagram.mmd`

## Очікуваний результат
- `lab4/model/uc-09-activity-diagram.puml` оновлено (виправлено охоплення циклу форми, оптимізовано переходи стрілок та параметри відступів)
- `lab4/model/uc-09-activity-diagram.png` оновлено (всі 6 свімлейнів на місці, відсутні накладання ліній на блоки дій, масштаб збалансований і текст легко читається)

## Аудит результатів
- **Статус**: Прийнято частково
- **Хеш коміту згенерованих артефактів**: `<43dd45552afe69fa4bde2a1cc296bb9010f6a05d>`
- **Згенеровані файли**: 
    - `lab4/model/uc-09-activity-diagram.puml`
    - `lab4/model/uc-09-activity-diagram.png`
- **Огляд результатів**:
    - **`uc-09-activity-diagram.puml`**:
        - **(Q4)**: Ключове слово `repeat` тепер стоїть перед діями автора, тому користувач дійсно повертається до полів і може виправити помилки.
        - Одразу після `repeat while` стоїть блок `if (Client validation failed?)`. Через це з'явилися два ромби поспіль без жодної дії між ними, а стрілка виходу з циклу не має підпису гілки.
        - Усі 6 свімлейнів оголошені правильно, ротація сесії йде через базу даних, для S3 працює retry-цикл, а після join налаштовано відкат.
    - **`uc-09-activity-diagram.png`**:
        - Усі 6 доріжок присутні на зображенні, текст у блоках став розбірливим.
        - Стрілки налізають одна на одну та на блоки дій по всій діаграмі.
- **Необхідні зміни**:
    1. Спростити вихід із циклу валідації в `.puml`, позбувшись зайвого другого `if` після `repeat while`, та підписати гілки переходу.
    2. Перегрупувати дії перевірки сесії та S3 так, щоб зворотні стрілки не різали вертикальні лінії основного потоку.
    3. Налаштувати переходи для блоків відкату в S3/БД та успішного збереження, щоб лінії між свімлейнами не налізали одна на одну.
    4. Додати мітки переходів `(yes)/(no)` до умовних вузлів для контролю напрямку виходу стрілок та випробувати стиль ортогональних ліній (`skinparam linetype ortho`), щоб усунути накладання ліній на рамки блоків.
    5. Підібрати `ranksep` для достатнього вертикального кліренсу стрілок, а ширину свімлейнів оптимізувати через жорстке обмеження довжини рядків тексту переносами `\n`.