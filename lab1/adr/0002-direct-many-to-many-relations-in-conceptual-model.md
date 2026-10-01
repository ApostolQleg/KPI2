# Model Conceptual Many-to-Many Relationships Directly Without Synthetic Junction Tables

## Context and Problem Statement

Several relationships in the domain exhibit many-to-many cardinality: a schematic can have multiple tags (`Schematic }o--|{ Tag`), support multiple game versions (`Schematic }o--|{ Version`), and be saved by multiple users (`User }o--o{ Schematic`). Should these relationships be modeled as explicit junction entities (e.g., `SchematicTag`, `UserSavedSchematic`) or as direct conceptual relationships?

## Considered Options

* Explicit junction/bridge entities carrying composite keys
* Direct Many-to-Many relationships between core domain entities

## Decision Outcome

Chosen option: "Direct Many-to-Many relationships between core domain entities", because none of these relationships currently carry distinct domain attributes (e.g., user bookmarks only express the link itself), and introducing intermediate tables premature to physical database implementation violates the conceptual abstraction level of an ER model.

### Consequences

* Good, because the ER diagram remains focused on genuine business entities rather than physical relational database artifacts.
* Good, because it strictly fulfills the course requirement prohibiting synthetic junction tables without own attributes.
* Bad, because if a relationship later requires audit data (such as a `saved_at` timestamp on bookmarking), it will require refactoring into an associative entity.