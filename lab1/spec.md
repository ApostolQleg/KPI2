# Entities and Attributes

- **User**:
    - `user_id`: UUIDv7 (PK)
    - `nickname`: string (UK)
    - `email`: string (UK)
    - `password_hash`: string
    - `created_at`: timestamp

- **Schematic**:
    - `schematic_id`: UUIDv7 (PK)
    - `author_id`: UUIDv7 (FK)
    - `category_id`: UUIDv7 (FK)
    - `title`: string
    - `description`: string (optional)
    - `litematica_url`: url
    - `world_zip_url`: url (optional)
    - `created_at`: timestamp

- **Comment**:
    - `comment_id`: UUIDv7 (PK)
    - `author_id`: UUIDv7 (FK)
    - `schematic_id`: UUIDv7 (FK)
    - `content`: string
    - `created_at`: timestamp

- **Category**:
    - `category_id`: UUIDv7 (PK)
    - `name`: string (UK)

- **Tag**:
    - `tag_id`: UUIDv7 (PK)
    - `name`: string (UK)

- **Version**:
    - `version_id`: UUIDv7 (PK)
    - `version_num`: string (UK)

# Relations
1. **`User` -creates- `Schematic` ( 1 : 0..N ):**
    * One user can upload zero-to-many schematics ( 1 : 0..M )
    * One schematic must have one and only author ( 1 : 1 )
2. **`User` -saves- `Schematic` ( 0..M : 0..N ):**
    * One user can save zero-to-many schematics ( 1 : 0..M )
    * One schamatic can be saved by zero-to-many users ( 1: 0..M )
3. **`User` -writes- `Comment` ( 1 : 0..N ):**
    * One user can write zero-to-many comments ( 1 : 0..M )
    * One comment must have one and only author ( 1 : 1 )
4. **`Schematic` -has- `Comment` ( 1 : 0..M ):**
    * One schematic can have zero-to-many comments ( 1 : 0..M )
    * One comment must have one and only schematic ( 1 : 1 )
5. **`Schematic` -classified by- `Category` ( 0..M : 1 ):**
    * One schematic must have one and only category ( 1 : 1 )
    * One category can have zero-to-many schematics ( 1 : 0..M )
6. **`Schematic` -has- `Tag` ( 0..M : 1..N ):**
    * One schematic must have at least one tag ( 1 : 1..M )
    * One tag can have zero-to-many schematics ( 1 : 0..M )
7. **`Schematic` -has- `Version` ( 0..M : 1..N ):**
    * One schematic must have at least one version ( 1 : 1..M )
    * One version can have zero-to-many schematics ( 1 : 0..M )

# Acceptance criteria
1. **Artifacts (CRITICAL):**
   - Mermaid model entity relationships diagram, saved in file `model/er-diagram.mmd`.
   - Rendered visual diagram file saved in `model/er-diagram.png`.
2. **Conceptual Model Constraints**:
   - Model business entities and domain relationships only, not physical database tables.
   - No associative entities for many-to-many relationships without own attributes. Many-to-many must be modeled as a direct relation between entities.
   - Associative entities are permitted only if the relationship itself carries domain-specific attributes.
3. **Normalization (3NF):**
   - No non-key attribute depends on another non-key attribute.
   - All non-key attributes must depend directly on the primary key.
   - `Category`, `Tag`, `Version` attributes normalized inside this tables and should not repeat inside `Schematic`.
4. **Unification of identifiers:**
   - All PK and FK is `UUIDv7`.
   - Entity and attributes names should be exactly the same to the `Entities and Attributes` paragraph (snake_case for attributes, PascalCase for entities).