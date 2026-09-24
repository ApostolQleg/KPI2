# Human-Readable Resource Addressing and URL Routing Strategy

## Context and Problem Statement

SchemaPostol is a community-oriented web platform where players share links to schematics and creator profiles across external platforms (e.g., Discord, Reddit, forums). While UUIDv7 is adopted as the uniform internal primary key across all entities (ADR-0003), exposing raw 128-bit UUID strings in public URLs degrades user experience, harms search engine optimization (SEO), and makes links unwieldy. How should public resources be addressed in web routes without compromising internal primary key integrity?

## Considered Options

* Expose raw UUIDv7 in all public URLs (e.g., `/schematics/018f2e45-...`, `/users/018f2e47-...`)
* Base62 / Short-hash encoding of internal UUIDs (e.g., `/s/7bXk9P`)
* Semantic `slug` for Schematics, unique `nickname` for Users, and raw UUIDv7 for internal/fine-grained resources

## Decision Outcome

Chosen option: "Semantic `slug` for Schematics, unique `nickname` for Users, and raw UUIDv7 for internal/fine-grained resources", because it optimizes sharing ergonomics, readability, and SEO for high-value public assets (schematics and author profiles), while avoiding unnecessary overhead for entities that are never accessed as standalone pages (comments, categories, tags).

### Consequences

* Good, because schematic links are descriptive and recognizable for community members (e.g., `/schematics/iron-golem-farm-v2` instead of an opaque UUID string).
* Good, because user profiles leverage the existing unique `nickname` attribute (`/users/{nickname}`) without requiring additional surrogate fields.
* Good, because comments do not incur slug generation overhead, as they are anchored directly to their parent schematic page and referenced internally or via fragment identifiers (`#comment-{uuidv7}`).
* Bad, because `Schematic` requires an additional unique attribute (`slug: string UK`), necessitating collision resolution logic during publication.