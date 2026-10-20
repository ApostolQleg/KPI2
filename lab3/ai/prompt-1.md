# Ітерація 1:

## Промт
> Read `lab3/spec.md`, `lab3/adr/0007-sessions-storage.md`, `lab2/adr/0005-oauth-for-authoring.md`, `lab2/model/use-case-diagram.puml`, and `lab1/model/er-diagram.mmd`.
> Based on the scenarios and strict diagram requirements defined in `lab3/spec.md`, generate the following artifacts:
> 1. `lab3/model/sequence-diagram.puml`:
>    - PlantUML Sequence Diagram covering Scenario 1 (Classic Registration & Login), Scenario 2 (Microsoft OAuth Session Creation), and Scenario 3 (Session Verification & Token Rotation).
>    - Use strict actor naming from Task 2: `Guest`, `Registered User`, `Author` (as human actors), `Microsoft OAuth Service` (external system), and `User` & `Session` (database entities matching Task 1 ERD and ADR-0007).
>    - Use `alt / else` blocks for invalid input / authentication failure / expired token branches.
>    - Strictly ensure symmetric activation bars: every `activate` MUST have a matching `deactivate` at the same scope level (do not leave activations hanging across `alt/else` boundaries).
>    - Use realistic REST endpoints (e.g. `POST /auth/register`, `POST /auth/login`, `POST /auth/refresh`) and internal backend methods (`hashPassword()`, `verifyJwt()`, `createSession()`).
> 2. Render `lab3/model/sequence-diagram.png` from the generated `.puml` file.

## Контекст
- Файл: `lab3/spec.md` (commit: `<5ada7d40fd5dae0bdc3a44086dea869f2436af0b>`)
- Файл: `lab2/requirements/ears.md` (commit: `<00139ec507eaab5c51824ad4e185260278e9398e>`)
- Файл: `lab2/model/use-case-diagram.puml` (commit: `<00139ec507eaab5c51824ad4e185260278e9398e>`)
- Файл: `lab2/requirements/user-stories.md` (commit: `<00139ec507eaab5c51824ad4e185260278e9398e>`)
- Файл: `lab1/model/er-diagram.mmd` (commit: `<64396cff7209a6d12f7a8c8b6c06b4a3b5e86f3b>`)

## Очікуваний результат
- `lab3/model/sequence-diagram.puml` створено
- `lab3/model/sequence-diagram.png` створено

## Аудит результатів
- **Статус**: Не прийнято
- **Хеш коміту згенерованих артефактів**: `<506f4c564714d41bf07d699a4ef3a7e4afddd0dc>`
- **Згенеровані файли**: 
    - `lab3/model/sequence-diagram.puml`
    - `lab3/model/sequence-diagram.png`
- **Огляд результатів**:
    - **`sequence-diagram.puml`**: Усі 3 складні сценарії (класична автентифікація, Microsoft OAuth та ротація сесій) були об'єднані в один монолітний файл. Одночасне оголошення трьох людських акторів (`Guest`, `Registered User`, `Author`) утворило неінформативні порожні вертикальні лінії. Використання `autonumber` у поєднанні з розгалуженнями `alt/else` заплутало кроки дій. Проблеми з активаційними барами фронтенду в усіх трьох послідовностях. Допущено технічні неточності: некоректне формулювання перевірки солей замість порівняння хешів пароля та помилкове відображення збереження `HttpOnly` кукі скриптами фронтенду (JS не має до них доступу, браузер обробляє `HttpOnly` автоматично).
    - **`sequence-diagram.png`**: Рендер виявився критично перевантаженим і важким для читання.
- **Необхідні зміни**:
    1. **Декомпозиція на 3 окремі діаграми**: Розділити один монолітний файл на 3 окремі PlantUML- специфікації (`01-classic-auth.puml`, `02-oauth.puml`, `03-token-refresh.puml`) із відповідними `.png` рендерами для кращої читабельності.
    2. **Об'єднання акторів**: На Sequence-діаграмах використовувати одного основного актора-людину (`User`), а його контекстну роль (`Guest` / `Author`) позначати за допомогою `note`, щоб прибрати зайві вертикальні лінії.
    3. **Оптимізація верстки**: Вилучити скрізну нумерацію `autonumber` та обмежувачі переносу рядків, щоб додати місця між повідомленнями та прибрати візуальний шум.
    4. **Виправлення технічної логіки**: Уточнити опис перевірки пароля (звіряння хешів через Argon2id/bcrypt) та скоригувати дію фронтенду щодо токенів (JS зберігає в пам'яті лише `accessToken`, а `HttpOnly` cookie керується браузером на рівні мережі).
    5. **Оновлення `spec.md`**: Змінити вимоги до структури артефактів у `spec.md`, зафіксувавши вище згадані необхідні зміни, а також вимогу збереження ролі `Author` у базі даних.