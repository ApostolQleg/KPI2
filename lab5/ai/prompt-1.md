# Ітерація 1:

## Промт
> Act as a software systems architecture assistant. Your task is to analyze `lab5/spec.md` and `lab5/adr/0008-modular-monolith-and-system-boundaries.md` (which serve as the single source of truth for Task 5), verify their alignment with previous artifacts (`lab1` ERD, `lab2` Use Cases & ADRs 0005–0006, `lab3` Sequence diagrams & ADR-0007, `lab4` Activity diagram), and produce a UML Component Diagram in PlantUML syntax.
>
> Execute this task in two strictly separated, sequential phases:
>
> Phase 1: Critical Analysis & Clarifications (STOP & ASK FIRST)
> - Review all components, system boundaries, internal layers, external integrations, and provided/required interfaces defined in `lab5/spec.md`.
> - Check for traceability gaps against Labs 1–4 (e.g., how `Litematica Parser Service` accesses the `.litematica` binary via `Message Queue` vs `S3`, how `UC09` rollback/timeout interacts with the queue, whether `Web UI` needs direct `S3` download links for `UC05`/`UC14`, or how `Microsoft OAuth` browser redirect involves `Web UI`).
> - Do NOT generate any PlantUML code yet. Stop execution, list concise clarifying questions in English with concrete trade-off options, and wait for my answers.
>
> Phase 2: Declarative Model Generation & Render (AFTER CLARIFICATIONS)
> - Once I answer your questions, generate `lab5/model/component-diagram.puml` strictly using PlantUML Component Diagram syntax.
> - Show a clear system boundary (`SchemaPostol`) containing `Web UI (SPA)`, the `Backend` modular monolith (`REST API`, `Auth & Session Service`, `Schematic & Catalog Service`, `Community Service`, `Data Access Layer`), `PostgreSQL Database`, and `Message Queue`.
> - Place external integrations strictly OUTSIDE the system boundary: `Microsoft OAuth`, `Object Storage (S3)`, and `Litematica Parser Service`.
> - Explicitly model UML provided and required interfaces (lollipop/socket notation or named interface nodes) and dependencies between components. Do NOT allow any service or UI to bypass `Data Access Layer` to reach `PostgreSQL Database`.
> - Do not invent out-of-scope components (no Payments, SMS, Email, Redis, or in-game integrations).
> - Optimize layout directives (`skinparam`, direction, package grouping, concise multiline labels) so the auto-rendered diagram has zero overlapping arrows over text and readable typography.
> - Render `lab5/model/component-diagram.png` directly from the `.puml` file.

## Контекст
- Файл: `lab5/spec.md` (commit: `a5dbe358ee058d22be2efc124d22a3af2face5b9`)
- Файл: `lab5/adr/0008-modular-monolith-and-system-boundaries.md` (commit: `8dbbd112811cc43583ec3b97396381e0c8789cd6`)
- Суміжні контексти: `lab1/model/er-diagram.mmd`, `lab2/model/use-case-diagram.puml`, `lab2/adr/0005-oauth-for-authoring.md`, `lab2/adr/0006-litematica-parsing.md`, `lab3/model/01-auth.puml`, `lab3/model/02-oauth.puml`, `lab3/model/03-token-refresh.puml`, `lab3/adr/0007-sessions-storage.md`, `lab4/model/uc-09-activity-diagram.puml`

## Очікуваний результат
- Фаза 1: отримано список уточнювальних питань щодо архітектурних меж, інтерфейсів та взаємодії через чергу й S3
- Фаза 2: після відповідей згенеровано `lab5/model/component-diagram.puml` та якісно відрендерений `lab5/model/component-diagram.png` без накладань ліній на блоки і подібного

## Аудит результатів