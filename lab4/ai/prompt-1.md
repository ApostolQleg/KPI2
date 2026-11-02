# Ітерація 1:

## Промт
> Act as a software systems modeling assistant. Your task is to analyze the provided specification `lab4/spec.md` (which serves as the absolute single source of truth) for the business process `UC09: Publish Schematic` and produce a corresponding PlantUML Activity Diagram using the Beta syntax.
>
> Follow these strict operational constraints:
> 1. Spec as the Sole Authority: Do not invent domain entities, attributes, or business rules not present in `lab4/spec.md` or its referenced artifacts (`lab2`, `lab3`, ADRs).
> 2. Gap Analysis and Clarifications: Perform a critical review of the control flows. Identify any architectural bottlenecks, potential concurrency deadlocks around the `fork`/`join` synchronization bar (such as S3 upload failures vs. parser failures), loopbacks, or optional branches (`world_zip_url`). Formulate precise clarifying questions and present trade-off options for any ambiguities rather than making silent assumptions. The questions should be in ukrainian.
> 3. Declarative Model Generation: Produce the complete PlantUML code strictly adhering to the 6 swimlanes (`Author`, `Frontend`, `Backend`, `Storage (S3)`, `Litematica Parser Service`, `Database`), conditional nodes, parallel execution paths, and terminal points (`stop`).
> 4. Layout and Syntax Rules: Use new PlantUML Activity Beta syntax (`|Swimlane|`, `start`, `stop`, `if (...) then (...) else (...) endif`, `fork ... fork again ... end fork`). Ensure all control paths converge correctly and that transitions are fully deterministic.

## Контекст
- Файл: `lab4/spec.md` (commit: `<input commit hash>`)
- Суміжні контексти: `lab2/model/use-case-diagram.puml`, `lab3/model/02-oauth.puml`, `lab2/adr/0005-oauth-for-authoring.md`, `lab2/adr/0006-external-parser-service.md`

## Очікуваний результат
- `lab4/model/uc-09-activity-diagram.puml` створено (первинна версія коду діаграми)
- Перелік знайдених прогалин, потенційних дедлоків на вузлі `join` та уточнювальних питань для інженерного вибору

## Аудит результатів