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