# Ітерація 1:

## Промт
> Read `lab2/spec.md` and the Task 1 ER model in `lab1/model/er-diagram.mmd`.
> Based on the actors, domain logic, and ER entities, generate the following artifacts:
> 1. `lab2/requirements/ears.md`: Minimum 8 EARS requirements (covering Ubiquitous, Event-driven, State-driven, Optional, Unwanted Behavior, and minimum 1 Out-of-Scope requirement `REQ-OUT-01`).
> 2. `lab2/requirements/user-stories.md`: Minimum 4 User Stories with full Gherkin scenarios (`GIVEN / WHEN / THEN`).
> 3. `lab2/model/use-case-diagram.puml`: PlantUML Use Case diagram with actors `Guest`, `Registered User`, `Author`, and `Litematica Parser Service`. Use `<<include>>` and `<<extend>>` properly.
> 4. `lab2/requirements/traceability-matrix.md`: Markdown table mapping `EARS Requirement ID` <-> `Use Case` in a Many-to-Many ($M:N$) manner, including `REQ-OUT-01` marked as `UNCOVERED`.

## Контекст
- Файл: `lab2/spec.md` (commit: `<bcf96f1caf76380010bd3c862fa403acb3e8619d>`)
- Файл: `lab1/model/er-diagram.mmd` (commit: `<64396cff7209a6d12f7a8c8b6c06b4a3b5e86f3b>`)

## Очікуваний результат
- `lab2/requirements/ears.md` створено
- `lab2/requirements/user-stories.md` створено
- `lab2/model/use-case-diagram.puml` створено
- `lab2/model/use-case-diagram.png` створено
- `lab2/requirements/traceability-matrix.md` створено

## Аудит результатів
- **Статус**: Не прийнято
- **Хеш коміту згенерованих артефактів**: `cf6a3da2fd5928761405822bccabd1fa52610da6`
- **Згенеровані файли**: 
    - `lab2/requirements/ears.md`     
    - `lab2/requirements/user-stories.md`
    - `lab2/model/use-case-diagram.puml`
    - `lab2/model/use-case-diagram.png`    
    - `lab2/requirements/traceability-matrix.md`
- **Огляд результатів**:
    - **`ears.md`**: Усі вимоги написані правильно відносно наданого контексту, логіку також не порушено. Описано більшість необхідних вимог. За охопленям вимоги покрили досить мало параметрів, але це вже проблема наданого контексту.
    - **`user-stories.md`**: Усі історії написані коректно відносно наданого контексту. Історії охопили більшість функціоналу та вписуються в загальну семантику проекту.
    - **`use-case-diagram.puml`**: Здебільшого схема правильна, однак парсер є актором, але не є людиною, тож його слід було позначити як систему. Прецедент Log In включено через `<<include>>` у більшість інших прецедентів. Це є помилкою, адже аутентифікованість є передумовою, а не підпрограмою для більшості прецедентів.
    - **`use-case-diagram.png`**: Поганий рендер, який дуже важко зрозуміти. Багато зв'язків поналазили один на одного, коментарії щодо типу зв'язків літають десь збоку і не зрозуміло якому зв'язку належать. Гість знаходить з протилежного боку від користувача та автора, а система парсингу позначена як людина. 
    - **`traceability-matrix.md`**: Немає однієї загальної матриці трасування з галочками про зв'язок юз-кейсів з вимогами, натомість це розділено на дві таблиці потребу в яких вказано в `spec.md` не було.
- **Рішення, що було прийнято під час аудиту (ADR)**:
    - `adr/0005-oauth-for-authoring.md`
    - `adr/0006-litematica-parsing.md`
- **Необхідні зміни**:
    1. **Виправити антипатерн `Log In <<include>>` у PlantUML**: Вилучити зв'язки `<<include>> Log In` від функціональних прецендентів (`Write Comment`, `Publish Schematic`, `Save Schematic` тощо). Вхід у систему є передумовою сесії (`Precondition`), що гарантується зв'язком прецедентів із ролями `Registered User` та `Author`.
    2. **Зафіксувати стереотип `<<System>>` для парсера**: Додати атрибут `<<System>>` до актора `Litematica Parser Service` у PlantUML, щоб він рендерився як системний компонент, а не як людина.
    3. **Оптимізувати лейаут діаграми в PlantUML**: Додати директиви розташування (`top to bottom direction`, групування акторів-людей з одного боку, прив'язка нотаток `note top/bottom of`), щоб усунути перетинання стрілок на `.png` рендері.
    4. **Консолідувати матрицю трасування**: Об'єднати дві окремі таблиці в єдину зручну матрицю $M:N$ у `traceability-matrix.md`, де вказано прямі зв'язки `EARS REQ-ID` <-> `Use Case ID`.
    5. **Інтегрувати рішення ADR-0005 (OAuth) у вимоги та сценарії**: Додати вимогу щодо обов'язкового розширення прав до `Author` через Microsoft OAuth у `spec.md`, `ears.md` та оновити відповідний Gherkin-сценарій у `user-stories.md`.