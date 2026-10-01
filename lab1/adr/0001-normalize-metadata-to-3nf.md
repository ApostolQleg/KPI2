# Normalize Metadata to Third Normal Form (3NF)

## Context and Problem Statement

In the SchemaPostol platform, schematics require classification by category, community tagging, and tracking of supported Minecraft versions. How should categorical and versioning metadata be structured within the domain model to avoid data duplication and update anomalies while supporting efficient search and filtering?

## Considered Options

* Denormalized string and array attributes directly inside `Schematic`
* Semi-structured JSON document attribute inside `Schematic`
* Third Normal Form (3NF) decomposition with independent `Category`, `Tag`, and `Version` entities

## Decision Outcome

Chosen option: "Third Normal Form (3NF) decomposition with independent `Category`, `Tag`, and `Version` entities", because it ensures complete entity independence, prevents data anomalies (for example inconsistent version naming or orphaned categories), and enables cross-schematic aggregation and discovery.

### Consequences

* Good, because renaming a category or version number requires updating a single record without mutating individual schematic entries.
* Good, because it prevents inconsistent user input for standardized versions (e.g., avoids mixed values like "1.20", "v1.20", "1.20.0").
* Good, because it strictly satisfies 3NF requirements by eliminating non-key transitive dependencies.
* Bad, because retrieving full schematic metadata will require relational joins or multi-entity lookups during physical implementation.