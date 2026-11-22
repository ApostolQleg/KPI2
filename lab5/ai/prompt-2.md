# Ітерація 2:

## Промт
> Update `lab5/model/component-diagram.puml` and re-render `lab5/model/component-diagram.png` addressing all defects identified in the Iteration 1 audit:
>
> 1. **Fix UML Provided / Required Interface Notation (No Dangling Interfaces)**:
>    - Remove all direct component-to-component arrows (`WEBUI --> RESTAPI`, `RESTAPI --> AUTHSVC`, etc.) and dead-end `provides` arrows (`RESTAPI --> IREST`).
>    - Connect components strictly **through** the interface nodes using standard PlantUML ball-and-socket (lollipop/socket) notation:
>      - Provider exposes interface with a solid line: `Provider -- Interface`
>      - Consumer requires interface with a socket connector: `Consumer -( Interface`
>
> 2. **Keep Internal Interfaces Inside the System Boundary**:
>    - Declare internal interfaces inside `rectangle "SchemaPostol"`:
>      - Inside `package "Backend (modular monolith)"`: `IAuthService`, `ISchematicService`, `ICommentService`, `IDataAccess`.
>      - Inside `rectangle "SchemaPostol"` (outside Backend package): `IRestApi`, `ISqlDriver`, `IParseJobQueue`.
>    - Keep only external system interfaces outside the `SchemaPostol` boundary: `IMicrosoftOAuthApi`, `IMicrosoftOAuthRedirect`, `IS3StorageApi`, `IS3DownloadApi`.
>
> 3. **Fix Parser S3 Interface (`Q1` / `Q3`)**:
>    - Connect `Litematica Parser Service` to the read-only `IS3DownloadApi` (`PARSER -( IS3R`), NOT `IS3StorageApi`. Keep `IS3StorageApi` exclusively for `Schematic Service` (upload and rollback deletion).
>
> 4. **Add Request/Reply Timeout & Rollback Note (`Q2`)**:
>    - Attach a concise `note` to `Message Queue` (or `Schematic Service`) documenting request/reply semantics (`correlation_id`, reply channel, timeout) and compensation rollback (deleting S3 objects via `IS3StorageApi` and deleting the DB draft via `IDataAccess` on parser failure or timeout). Keep the diagram strictly structural without procedural flow arrows.
>
> 5. **Fix Title Encoding & Visual Layout**:
>    - Replace the em-dash `—` in the title with a plain ASCII hyphen (`title SchemaPostol - Component Diagram (Lab 5)`) to prevent character encoding corruption.
>    - Adjust spacing (`nodesep`, `ranksep`) and layout grouping so no labels overlap each other or container borders, and all arrows route cleanly.
>    - Render `lab5/model/component-diagram.png` directly from `lab5/model/component-diagram.puml`.

## Контекст
- Файл: `lab5/spec.md` (commit: `0d497aad0c36970eadee9f7faadb724be04e0de5`)
- Файл: `lab5/adr/0008-modular-monolith-and-system-boundaries.md` (commit: `8dbbd112811cc43583ec3b97396381e0c8789cd6`)
- Файл: `lab5/model/component-diagram.puml` (commit: `0017cea8bb7279039c17685695fade0c9537f623`)
- Суміжні контексти: `lab1/model/er-diagram.mmd`, `lab2/model/use-case-diagram.puml`, `lab2/adr/0005-oauth-for-authoring.md`, `lab2/adr/0006-litematica-parsing.md`, `lab3/model/01-auth.puml`, `lab3/model/02-oauth.puml`, `lab3/model/03-token-refresh.puml`, `lab3/adr/0007-sessions-storage.md`, `lab4/model/uc-09-activity-diagram.puml`

## Очікуваний результат
- Оновлено `lab5/model/component-diagram.puml` із коректними UML ball-and-socket інтерфейсами всередині та зовні межі системи, правильним інтерфейсом `IS3DownloadApi` для парсера та нотаткою про таймаут і відкат
- Відрендерено читабельний `lab5/model/component-diagram.png` без битих символів у заголовку та без накладання підписів

## Аудит результатів
## Аудит результатів
- **Статус**: Прийнято частково
- **Хеш коміту згенерованих артефактів**: `85a95395616ab9e0dc05895ea2959179cfc14d18`
- **Згенеровані файли**: 
  - `lab5/model/component-diagram.puml`
  - `lab5/model/component-diagram.png`
- **Огляд результатів**:
  - **`component-diagram.puml`**:
    - Усі змістовні та нотаційні зауваження Ітерації 1 виправлено: використано коректні UML ball-and-socket інтерфейси (`--` та `-(`), внутрішні інтерфейси повернуто в межу `SchemaPostol`, для `Litematica Parser Service` вказано `IS3DownloadApi`, додано нотатку про request/reply та компенсаційний відкат, виправлено заголовок.
  - **`component-diagram.png`**:
    - Порушено природний порядок шарів зверху вниз: `PostgreSQL Database` опинилася нагорі, а `Web UI (SPA)` - унизу.
    - Написи сусідніх інтерфейсів накладаються один на одного (`ICommentService` + `ISchematicService`, `IRestApi` + `IMicrosoftOAuthRedirect`), а лінія перетинає текст `IParseJobQueue`.
    - `Message Queue` розміщена зліва біля БД, а `Litematica Parser Service` - справа зовні, через що лінія перетинає всю систему; `note right of MQ` винесло за межі S3 довгим пунктиром через усю діаграму.
- **Необхідні зміни**:
  1. **Відновити архітектурний порядок зверху вниз**: розмістити шари у природній послідовності: `Web UI (SPA)` нагорі → `REST API` → сервіси (`Auth`, `Schematic`, `Comment`) → `Data Access Layer` → `PostgreSQL Database` унизу.
  2. **Прибрати накладання написів інтерфейсів**: відмовитися від `linetype ortho` на користь стандартного трасування ліній PlantUML, збільшити горизонтальний інтервал `nodesep` (до 75–85) і згрупувати внутрішні сервіси через `together`, щоб кружечки та підписи інтерфейсів не злипалися.
  3. **Оптимізувати позицію `Message Queue` та нотатки**: розмістити `Message Queue` ближче до правого краю `SchemaPostol` (поруч зі `Schematic Service` та зовнішнім `Litematica Parser Service`), а нотатку прив'язати знизу (`note bottom of MQ`), щоб прибрати довгу пунктирну лінію через усю схему.