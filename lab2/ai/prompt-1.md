# Iteration 1: Requirements, Use Cases, and Traceability Generation

## Prompt
> Read `lab2/spec.md` and the Task 1 ER model in `lab1/model/er-diagram.mmd`.
> Based on the actors, domain logic, and ER entities, generate the following artifacts:
> 1. `lab2/requirements/ears.md`: Minimum 8 EARS requirements (covering Ubiquitous, Event-driven, State-driven, Optional, Unwanted Behavior, and minimum 1 Out-of-Scope requirement `REQ-OUT-01`).
> 2. `lab2/requirements/user-stories.md`: Minimum 4 User Stories with full Gherkin scenarios (`GIVEN / WHEN / THEN`).
> 3. `lab2/model/use-case-diagram.puml`: PlantUML Use Case diagram with actors `Guest`, `Registered User`, `Author`, and `Litematica Parser Service`. Use `<<include>>` and `<<extend>>` properly.
> 4. `lab2/requirements/traceability-matrix.md`: Markdown table mapping `EARS Requirement ID` <-> `Use Case` in a Many-to-Many ($M:N$) manner, including `REQ-OUT-01` marked as `UNCOVERED`.

## Context Provided
- File: `lab2/spec.md` (commit `<bcf96f1caf76380010bd3c862fa403acb3e8619d>`)
- File: `lab1/model/er-diagram.mmd` (commit `<64396cff7209a6d12f7a8c8b6c06b4a3b5e86f3b>`)

## Expected Outcome
- `lab2/requirements/ears.md` created;
- `lab2/requirements/user-stories.md` created;
- `lab2/model/use-case-diagram.puml` created;
- `lab2/requirements/traceability-matrix.md` created;

## AI Output Audit