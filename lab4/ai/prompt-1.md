# Ітерація 1:

## Промт
> Act as a software systems modeling assistant. Your task is to analyze the provided specification `lab4/spec.md` (which serves as the absolute single source of truth) for the business process `UC09: Publish Schematic` and produce a corresponding PlantUML Activity Diagram using the Beta syntax.
>
> Execute this task in two strictly separated, sequential phases:
>
> Phase 1: Critical Analysis & Clarifications (STOP & ASK FIRST)
> - Perform a critical review of the control flows in `lab4/spec.md`.
> - Identify any architectural bottlenecks, potential concurrency deadlocks around the `fork`/`join` synchronization bar (such as S3 upload failures vs. parser failures), validation loops, or optional branches (`world_zip_url`).
> - Do NOT generate the diagram code immediately. Stop execution, formulate precise clarifying questions strictly in English, and present concrete trade-off options for each ambiguity. Await my answers before proceeding.
>
> Phase 2: Declarative Model Generation (AFTER CLARIFICATIONS)
> - Once I provide answers to your clarifying questions, generate the complete PlantUML code strictly incorporating the resolved decisions.
> - Adhere to the 6 swimlanes (`Author`, `Frontend`, `Backend`, `Storage (S3)`, `Litematica Parser Service`, `Database`), conditional decision nodes, parallel execution paths, and terminal points (`stop`).
> - Use new PlantUML Activity Beta syntax (`|Swimlane|`, `start`, `stop`, `if (...) then (...) else (...) endif`, `fork ... fork again ... end fork`). Ensure all control paths converge deterministically without deadlocks.
> - Do not invent domain entities, attributes, or business rules not present in `lab4/spec.md` or its referenced artifacts (`lab2`, `lab3`, ADRs).
> - Render generated file into `.png` format.

## Контекст
- Файл: `lab4/spec.md` (commit: `<input commit hash>`)
- Суміжні контексти: `lab2/model/use-case-diagram.puml`, `lab3/model/02-oauth.puml`, `lab2/adr/0005-oauth-for-authoring.md`, `lab2/adr/0006-external-parser-service.md`

## Очікуваний результат
- Отримано список уточнювальних запитань українською мовою щодо вузьких місць, обробки збоїв та поведінки синхронізації на вузлі `join`
- Після надання відповідей — згенеровано файл `lab4/model/uc-09-activity-diagram.puml` із повною реалізацією діаграми згідно з уточненими вимогами

## Аудит результатів