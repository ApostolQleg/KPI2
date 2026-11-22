# Архітектура (Component Diagram)

## Намір
Побудувати UML Component діаграму платформи **SchemaPostol**, яка зводить в єдину архітектурну модель усі попередні артефакти: дані (`lab1`), вимоги та юз-кейси (`lab2`), автентифікацію (`lab3`) і процес публікації (`lab4`). Діаграма має чітко розділяти внутрішні компоненти системи та зовнішні інтеграції через provided/required інтерфейси.

## Межа системи `SchemaPostol`

### Клієнт
- **`Web UI (SPA)`** - єдиний фронтенд-компонент у браузері. Відповідає за каталог, фільтрацію (`UC03`–`UC05`), коментарі та збереження (`UC06`, `UC07`), форму публікації з клієнтською валідацією (`UC09`, `UC10`, `UC13`), зберігання `accessToken` у пам'яті.

### Бекенд
- **`REST API`** - вхідна точка для `Web UI`. Роутинг ендпоінтів (`/auth/*`, `/api/schematics/*`, `/api/comments/*`), перевірка JWT, серверна валідація MIME-типів і лімітів розміру.
- **`Auth Service`** - реєстрація, вхід через `Argon2id`, випуск JWT, ротація `refreshToken` через таблицю `Session` (`ADR-0007`), підвищення ролі `User.isAuthor` через Microsoft OAuth (`ADR-0005`).
- **`Schematic Service`** - пошук і фільтрація за `Category`, `Tag`, `Version`, CRUD власних схематик (`UC12`, `UC15`), генерація `slug` (`ADR-0004`), оркестрація публікації `UC09` (створення чернетки `is_published = false`, завантаження в S3 з retry-політикою, постановка задачі в чергу парсингу, фінальний коміт або компенсаційний відкат).
- **`Comment Service`** - додавання коментарів (`UC06`) та керування збереженими схематиками (`UC07`).
- **`Data Access Layer`** - єдиний рівень доступу до БД з допомогою репозиторіїв чи ORM. Інкапсулює запити до сутностей `User`, `Session`, `Schematic`, `Comment`, `Category`, `Tag`, `Version`.

### Внутрішня інфраструктура
- **`PostgreSQL Database`** - реляційна БД платформи. Доступ дозволено лише з `Data Access Layer`.
- **`Message Queue`** - внутрішня система черг для асинхронної передачі задач з парсингу `.litematica` та отримання результатів/метаданих (`ADR-0006`). На рівні процесу в `lab4` шину черг було пропущено для спрощення `fork/join`, але в архітектурній моделі він явно зафіксована за `ADR-0006`.

## Зовнішні системи та сервіси
- **`Microsoft OAuth`** - зовнішній провайдер авторизації для перевірки ліцензії Minecraft та видачі прав `Author` (`ADR-0005`).
- **`Object Storage (S3)`** - зовнішнє об'єктне сховище для бінарних `.litematica` файлів та опціональних `.zip` архівів світу (`litematica_url`, `world_zip_url`).
- **`Litematica Parser Service`** - зовнішній залежний від моєї системи фоновий сервіс (`lab2`, `ADR-0006`). Забирає задачі з `Message Queue`, розбирає NBT-структуру, перевіряє контрольні суми й повертає метадані або статус помилки.

## Основні інтерфейси (Provided / Required)
- `Web UI (SPA)` → requires `IRestApi` ← provided by `REST API`.
- `REST API` → requires `IAuthService`, `ISchematicService`, `ICommunityService` ← provided by відповідні сервіси бекенду.
- Сервіси бекенду → require `IDataAccess` ← provided by `Data Access Layer`.
- `Data Access Layer` → requires `ISqlDriver` ← provided by `PostgreSQL Database`.
- `Auth & Session Service` → requires `IMicrosoftOAuthApi` ← provided by `Microsoft OAuth`.
- `Schematic & Catalog Service` → requires `IS3StorageApi` ← provided by `Object Storage (S3)`.
- `Schematic & Catalog Service` → requires `IParseJobQueue` ← provided by `Message Queue`.
- `Litematica Parser Service` → requires `IParseJobQueue` (публікація результатів) та `IS3StorageApi` (за потреби читання файлу за посиланням або отримання пейлоада з черги).

## Out of Scope
- `REQ-OUT-01`: пряма інтеграція з ігровим клієнтом чи серверами Minecraft.
- Платіжні шлюзи, SMS та Email-сервіси.
- Підписки на авторів.

## Вимоги до діаграми та критерії прийому
1. **Артефакти**: `lab5/model/component-diagram.puml` та автоматично відрендерений `lab5/model/component-diagram.png`.
2. **Нотація**: явна межа системи `SchemaPostol`, зовнішні інтеграції за її межами, коректні UML provided/required інтерфейси, відсутність прямого доступу сервісів чи UI до БД в обхід `Data Access Layer`.
3. **Читабельність**: чітке групування, без перехрещення ліній поверх тексту блоків, розбірливий шрифт.
4. **Узгодженість**: кожен компонент і зв'язок трасується до сутностей `lab1`, юз-кейсів `lab2`, сценаріїв `lab3`, процесу `lab4` та ADR-0001..0008, проте іноді співпадає саме з їх комбінацією.