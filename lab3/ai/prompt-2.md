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