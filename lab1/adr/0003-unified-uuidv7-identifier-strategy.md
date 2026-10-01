# Unified UUIDv7 Identifier Strategy Across All Entities

## Context and Problem Statement

Entities across the SchemaPostol domain require robust, unique primary and foreign key identification. Using mixed or primitive identifier types (such as auto-incrementing integers for some tables and random UUIDv4 for others) leads to type inconsistency, security vulnerabilities (ID enumeration), and database index fragmentation. What unified identifier strategy should be adopted?

## Considered Options

* Auto-incrementing integers / BigInt
* Random UUID (UUIDv4)
* Time-ordered UUID (UUIDv7)

## Decision Outcome

Chosen option: "Time-ordered UUID (UUIDv7)", because it provides global uniqueness suitable for distributed systems while maintaining monotonic chronological sortability (K-sortable), which prevents B-tree index fragmentation and establishes a strict, uniform typing standard across all entities.

### Consequences

* Good, because all Primary Keys (`PK`) and Foreign Keys (`FK`) share an identical, strongly typed standard (`UUIDv7`) across the entire schema.
* Good, because creation timestamps are natively encoded within the identifier, enabling natural time-based sorting.
* Good, because it prevents external enumeration attacks on user-facing IDs.
* Bad, because 128-bit UUIDs consume more storage and memory than standard 32-bit or 64-bit integers and are less human-readable in URLs.