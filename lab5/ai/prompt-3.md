# Ітерація 3: ререндер рендеру

## Промт
> Update `lab5/model/component-diagram.puml` and re-render `lab5/model/component-diagram.png` to fix the visual layout and label collisions identified in the Iteration 2 audit while keeping all components, boundaries, and UML ball-and-socket interfaces unchanged:
>
> 1. **Enforce Natural Top-to-Bottom Architectural Layering**:
>    - Ensure `Web UI (SPA)` is positioned at the **top** of `SchemaPostol`, followed downward by `REST API`, the domain services (`Auth Service`, `Schematic Service`, `Comment Service`), `Data Access Layer`, and `PostgreSQL Database` at the **bottom**.
>    - Use directional hints on connectors (e.g. `-down-`, `-down-(`) and `together` blocks so Graphviz does not invert the diagram upside down.
>
> 2. **Eliminate Interface Label Collisions**:
>    - Remove `skinparam linetype ortho` (use default polyline/spline routing) so connector lines do not merge into overlapping horizontal bundles or slice through interface labels like `IParseJobQueue`.
>    - Increase `skinparam nodesep 80` and `skinparam ranksep 65` so adjacent interface labels (`ICommentService`, `ISchematicService`, `IAuthService`, `IRestApi`, `IMicrosoftOAuthRedirect`) have plenty of horizontal breathing room and never overlap.
>
> 3. **Fix `Message Queue` & Note Placement**:
>    - Position `Message Queue` on the right side inside `SchemaPostol` (adjacent to `Schematic Service` and closer to the external `Litematica Parser Service`) so their connection does not cross the entire width of the diagram.
>    - Change `note right of MQ` to `note bottom of MQ` (or `note left of MQ` inside the boundary) so the note stays compact near `Message Queue` instead of stretching a long dashed line across external systems.
>    - Keep `Microsoft OAuth` on the left side (near `Web UI` and `Auth Service`) and `Object Storage (S3)` + `Litematica Parser Service` on the right side (near `Web UI` and `Schematic Service`) to minimize cross-diagram line crossings.
>
> 4. **Verify & Render**:
>    - Keep all 8 components, system boundary, external clouds, and 11 interfaces intact.
>    - Re-render `lab5/model/component-diagram.png` directly from `lab5/model/component-diagram.puml` and verify that zero labels overlap and the vertical flow reads cleanly from top to bottom.

## Контекст
- Файл: `lab5/spec.md` (commit: `0d497aad0c36970eadee9f7faadb724be04e0de5`)
- Файл: `lab5/adr/0008-modular-monolith-and-system-boundaries.md` (commit: `8dbbd112811cc43583ec3b97396381e0c8789cd6`)
- Файл: `lab5/model/component-diagram.puml` (commit: `85a95395616ab9e0dc05895ea2959179cfc14d18`)

## Очікуваний результат
- Оновлено `lab5/model/component-diagram.puml` із правильним порядком шарів зверху вниз, оптимальним розміщенням `Message Queue` і нотатки та достатніми інтервалами між інтерфейсами
- Автоматично відрендерено чистий і читабельний `lab5/model/component-diagram.png` без накладання написів та перехрещення ліній через усю діаграму

## Аудит результатів