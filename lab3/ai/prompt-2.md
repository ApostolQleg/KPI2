# Ітерація 2:

## Промт
> Read `lab3/spec.md`, `lab3/adr/0007-sessions-storage.md`, `adr/0005-oauth-for-authoring.md`, `lab2/model/use-case-diagram.puml`, and `lab1/model/er-diagram.mmd`.
> Re-generate all Task 3 artifacts addressing all Action Items from Iteration 1:
> 1. Delete the obsolete monolithic files: `lab3/model/sequence-diagram.puml` and `lab3/model/sequence-diagram.png`.
> 2. Generate 3 separate PlantUML sequence diagrams matching the updated `lab3/spec.md`:
>    - `lab3/model/01-auth.puml`: Scenario 1 (Classic Registration & Login).
>    - `lab3/model/02-oauth.puml`: Scenario 2 (Microsoft OAuth Session Creation & Author role elevation via `isAuthor` flag in User table).
>    - `lab3/model/03-token-refresh.puml`: Scenario 3 (Session Verification & Token Rotation).
> 3. Strict Diagram Rules & Technical Corrections:
>    - Use a single unified human actor `User` on the lifeline; indicate context roles (`Guest` / `Registered User` / `Author`) via `note`.
>    - Database entities MUST match Task 1 ERD and ADR-0007 (`User` has `isAuthor` flag, `Session`).
>    - Strictly ensure symmetric activation bars: every `activate` MUST have a matching `deactivate` at the same scope level.
>    - DO NOT use `autonumber` and DO NOT use restrictive `wrapWidth` / `maxMessageSize` parameters.
>    - Password check: compare submitted password against stored `password_hash` using Argon2id.
>    - Token storage: JS stores `accessToken` in memory, while browser automatically manages the `HttpOnly` refresh-token cookie.
> 4. Render PNG files for all 3 generated diagrams: `01-auth.png`, `02-oauth.png`, `03-token-refresh.png`.

## Контекст
- Файл: `lab3/spec.md` (commit: `<0fde30b26809a943c24c7f938f04a17aafbf355f>`)
- Файл: `lab2/requirements/ears.md` (commit: `<00139ec507eaab5c51824ad4e185260278e9398e>`)
- Файл: `lab2/model/use-case-diagram.puml` (commit: `<00139ec507eaab5c51824ad4e185260278e9398e>`)
- Файл: `lab2/requirements/user-stories.md` (commit: `<00139ec507eaab5c51824ad4e185260278e9398e>`)
- Файл: `lab1/model/er-diagram.mmd` (commit: `<64396cff7209a6d12f7a8c8b6c06b4a3b5e86f3b>`)

## Очікуваний результат
   - `lab3/model/01-auth.puml` та `.png` рендер створено
   - `lab3/model/02-oauth.puml` та `.png` рендер створено
   - `lab3/model/03-token-refresh.puml` та `.png` рендер створено
   - `lab3/model/sequence-diagram.puml` видалено
   - `lab3/model/sequence-diagram.png` видалено

## Аудит результатів
- **Статус**: Прийнято частково
- **Хеш коміту згенерованих артефактів**: `<1ddbcccf3479ab65bed92f073b23f525874d116c>`
- **Згенеровані файли**: 
    - `lab3/model/01-auth.puml` та `01-auth.png`
    - `lab3/model/02-oauth.puml` та `02-oauth.png`
    - `lab3/model/03-token-refresh.puml` та `03-token-refresh.png`
    - `lab3/model/sequence-diagram.puml` (видалено)
    - `lab3/model/sequence-diagram.png` (видалено)
- **Огляд результатів**:
    - **`01-auth.puml` / `02-oauth.puml` / `03-token-refresh.puml`**: Декомпозиція на 3 окремі файли виконана успішно, застарілі монолітні файли видалено. Використано єдиного актора `User` з нотатками ролей, додано прапор `isAuthor` для підтвердження ролі `Author` у базі даних, прибрано плутанину з `autonumber`. Однак виявлено системний недолік: `Frontend` повністю позбавлений активаційних барів (`activate FE` / `deactivate FE`), через що клієнт виглядає пасивною лінією зв'язку, а не активним компонентом, який виконує запити, обробляє відповіді та зберігає `accessToken` у пам'яті.
    - **`01-auth.png` / `02-oauth.png` / `03-token-refresh.png`**: Читабельність схем суттєво зросла порівняно з монолітним варіантом, текст не розривається. Проте візуально лінія `Frontend` не відображає блоків виконання під час обробки клієнтських подій.
- **Необхідні зміни**:
    1. **Додати активаційні бари для `Frontend`**: У всіх трьох `.puml` файлах розставити `activate FE` при отриманні події від `User` та парні `deactivate FE` після завершення обробки відповіді/збереження токенів.
    2. **Перегенерувати PNG-рендери**: Оновити всі 3 графічні файли діаграм після додавання активацій `Frontend`.