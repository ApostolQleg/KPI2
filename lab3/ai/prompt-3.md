# Ітерація 3:

## Промт
> Update `lab3/model/01-auth.puml`, `lab3/model/02-oauth.puml`, and `lab3/model/03-token-refresh.puml` to add missing activation bars for `Frontend` (`FE`):
> 1. In `01-auth.puml`: Add `activate FE` when `HumanUser` triggers Registration/Login and matching `deactivate FE` after `FE` processes backend responses and token storage.
> 2. In `02-oauth.puml`: Add `activate FE` during the initial OAuth URL request (deactivating before redirecting to MS) AND `activate FE` upon receiving the callback authorization code redirect from MS (deactivating after backend exchange and token storage).
> 3. In `03-token-refresh.puml`: Add `activate FE` when `HumanUser` performs an authorized action, keeping `FE` active through the API request and potential token refresh flow until completion.
> 4. Ensure strict symmetry: every `activate FE` MUST have a matching `deactivate FE` at the proper scope level.
> 5. Re-render all 3 PNG images: `lab3/model/01-auth.png`, `lab3/model/02-oauth.png`, and `lab3/model/03-token-refresh.png`.

## Контекст
- Файл: `lab3/spec.md` (commit: `<0fde30b26809a943c24c7f938f04a17aafbf355f>`)
- Файл: `lab3/model/01-auth.puml` (commit: `<1ddbcccf3479ab65bed92f073b23f525874d116c>`)
- Файл: `lab3/model/02-oauth.puml` (commit: `<1ddbcccf3479ab65bed92f073b23f525874d116c>`)
- Файл: `lab3/model/03-token-refresh.puml` (commit: `<1ddbcccf3479ab65bed92f073b23f525874d116c>`)

## Очікуваний результат
- `lab3/model/01-auth.puml` та `01-auth.png` оновлено (додано активації `FE`)
- `lab3/model/02-oauth.puml` та `02-oauth.png` оновлено (додано активації `FE`)
- `lab3/model/03-token-refresh.puml` та `03-token-refresh.png` оновлено (додано активації `FE`)

## Аудит результатів
- **Статус**: Прийнято
- **Хеш коміту згенерованих артефактів**: `<5b6eb8a5da5732c14bbf2df02cdf30c29d268930>`
- **Згенеровані файли**: 
    - `lab3/model/01-auth.puml` та `01-auth.png`
    - `lab3/model/02-oauth.puml` та `02-oauth.png`
    - `lab3/model/03-token-refresh.puml` та `03-token-refresh.png`
- **Огляд результатів**:
    - **`01-auth.puml` / `02-oauth.puml` / `03-token-refresh.puml`**: Усі 3 специфікації оновлено з дотриманням суворої симетрії активацій. Для `Frontend` додано бари активності (`activate FE` / `deactivate FE`) під час обробки клієнтських подій, запитів до API та збереження токенів у пам'яті.
    - **`01-auth.png` / `02-oauth.png` / `03-token-refresh.png`**: Графічні артефакти повністю поновлено. Життєві лінії `Frontend` чітко відображають бари виконання, діаграми читабельні, без візуальних накладань та збігаються з `spec.md`.
- **Необхідні зміни**:
    - Зміни не потрібні. Усі вимоги Ітерацій 1–3 виконано повністю.