# User Stories — SchemaPostol

> Lab 2 · Task 2 · Gherkin scenarios.
> Every scenario uses `GIVEN / WHEN / THEN` steps and references only the actors from `lab2/spec.md` (`Guest`, `Registered User`, `Author`) and the ER entities from `lab1/model/er-diagram.mmd`. The `Litematica Parser Service` is not an actor (`lab2/spec.md`): it is a dependent external background service used inside scenarios as a precondition/trigger.

## Index

| ID | Title | Primary actor | Related requirements |
|---|---|---|---|
| US-01 | Search and filter public schematics | Guest | REQ-SYS-001, REQ-STA-001 |
| US-02 | Comment on a schematic | Registered User | REQ-EVN-003, REQ-STA-002, REQ-SYS-003 |
| US-03 | Save schematics to a personal collection | Registered User | REQ-OPT-002, REQ-STA-002 |
| US-04 | Publish a `.litematica` schematic | Author | REQ-EVN-001, REQ-EVN-002, REQ-EVN-004, REQ-UNW-001, REQ-UNW-003, REQ-OPT-001 |
| US-05 | Manage own publications | Author | REQ-SYS-003, REQ-EVN-004, REQ-UNW-002 |
| US-06 | Elevate to Author via Microsoft OAuth | Registered User | REQ-EVN-004, REQ-UNW-003 |

---

## US-01 — Search and filter public schematics

**As a** Guest,
**I want to** search and filter published schematics by `Category`, `Tag` and `Version`,
**so that** I can find a build I need without registering.

| Field | Value |
|---|---|
| Priority | High |
| ER entities | `Schematic`, `Category`, `Tag`, `Version`, `Comment` |
| Related requirements | REQ-SYS-001, REQ-STA-001 |
| Use cases | UC-03, UC-04, UC-05 |

```gherkin
Feature: Public catalog search and filtering
  As a Guest
  I want to search and filter published schematics
  So that I can discover builds without registering

  Scenario: Filter schematics by Category, Tag and Version
    Given I am an unauthenticated visitor on the catalog page
    And the category "Redstone", the tag "Farm" and the version "1.20.1" exist
    When I apply the filters category "Redstone", tag "Farm" and version "1.20.1"
    Then the system shall display only published schematics matching all three filters
    And each result entry shall show the title, slug, Category, Tags and Versions of the Schematic

  Scenario: View schematic details together with comments
    Given I am a Guest
    And a published schematic with the slug "victorian-house" exists
    When I open the page of that schematic by its slug
    Then the system shall show the title, description, litematica_url, Category, Tags, Versions and all Comments
    And I shall not be required to authenticate

  Scenario: Schematics awaiting parsing are not exposed
    Given an Author has submitted a schematic whose .litematica file is still being validated by the Litematica Parser Service
    When I search the catalog
    Then the system shall not list that schematic in the search results
    And the schematic page of that schematic shall not be publicly accessible
```

---

## US-02 — Comment on a schematic

**As a** Registered User,
**I want to** leave comments under published schematics,
**so that** I can give feedback to authors.

| Field | Value |
|---|---|
| Priority | Medium |
| ER entities | `Comment`, `User`, `Schematic` |
| Related requirements | REQ-EVN-003, REQ-STA-002, REQ-SYS-003 |
| Use cases | UC-06 |

```gherkin
Feature: Commenting on schematics
  As a Registered User
  I want to write comments under published schematics
  So that I can give feedback to authors

  Scenario: Post a comment successfully
    Given I am authenticated with the nickname "steve_builder"
    And a published schematic with the slug "victorian-house" exists
    When I submit the comment content "Amazing detail on the roof!"
    Then the system shall persist a new Comment linked to my User and to that Schematic
    And the comment shall appear on the schematic page with my nickname and its created_at timestamp

  Scenario: Unauthenticated comment attempt is rejected
    Given I am not authenticated
    And a published schematic with the slug "victorian-house" exists
    When I try to submit a comment on that schematic
    Then the system shall deny the operation
    And the system shall redirect me to the Log In flow
    And no Comment record shall be created
```

---

## US-03 — Save schematics to a personal collection

**As a** Registered User,
**I want to** save liked schematics and remove them from my collection later,
**so that** I can return to my favourite builds quickly.

| Field | Value |
|---|---|
| Priority | Medium |
| ER entities | `User`–`Schematic` (`saves` M:N), `Schematic` |
| Related requirements | REQ-OPT-002, REQ-STA-002 |
| Use cases | UC-07 |

```gherkin
Feature: Saving schematics to a personal collection
  As a Registered User
  I want to manage my saved schematics
  So that I can return to my favourite builds quickly

  Scenario: Save a schematic
    Given I am authenticated with the nickname "alex_miner"
    And a published schematic with the slug "mega-base-kit" exists
    When I save that schematic
    Then the system shall record the saves relation between my User and that Schematic
    And the schematic shall appear in my saved collection

  Scenario: Remove a schematic from the saved collection
    Given I am authenticated with the nickname "alex_miner"
    And I have previously saved the schematic with the slug "mega-base-kit"
    When I remove that schematic from my saved collection
    Then the system shall remove the saves relation between my User and that Schematic
    And the schematic shall remain published and publicly visible

  Scenario: Saving requires authentication
    Given I am not authenticated
    When I try to save a schematic
    Then the system shall deny the operation and redirect me to the Log In flow
```

---

## US-04 — Publish a `.litematica` schematic

**As an** Author,
**I want to** publish a `.litematica` file together with a `Category`, `Tag`s and `Version`s,
**so that** other players can discover and use my build.

| Field | Value |
|---|---|
| Priority | Critical |
| ER entities | `Schematic`, `User`, `Category`, `Tag`, `Version` |
| Related requirements | REQ-EVN-001, REQ-EVN-002, REQ-EVN-004, REQ-UNW-001, REQ-UNW-003, REQ-OPT-001 |
| Use cases | UC-09, UC-10, UC-11, UC-13 |

```gherkin
Feature: Publishing a schematic
  As an Author
  I want to publish a .litematica file with classification metadata
  So that other players can discover and use my build

  Scenario: Successful publication after parser validation
    Given I am authenticated as an Author with the nickname "oleg_builder"
    And my Author role was granted through the Microsoft OAuth flow after linking my licensed Minecraft account (ADR-0005)
    When I submit the title "Victorian House", a description, the file "victorian-house.litematica", the category "Medieval", the tag "House" and the version "1.20.1"
    Then the system shall create a Schematic linked to my User via author_id
    And the system shall dispatch the .litematica file to the Litematica Parser Service
    When the Litematica Parser Service confirms the file integrity and returns the extracted metadata
    Then the system shall enrich the Schematic with the returned metadata
    And the schematic shall become discoverable through search and filters with its slug

  Scenario: Corrupted file is rejected
    Given I am authenticated as an Author with the nickname "oleg_builder"
    And my Author role was granted through the Microsoft OAuth flow after linking my licensed Minecraft account (ADR-0005)
    When I submit a corrupted .litematica file for publication
    And the Litematica Parser Service reports an integrity failure
    Then the system shall reject the publication attempt
    And the Schematic shall remain unpublished and invisible to other users
    And I shall receive the parser's error details

  Scenario: Optional world archive attachment
    Given I am authenticated as an Author with the nickname "oleg_builder"
    And my Author role was granted through the Microsoft OAuth flow after linking my licensed Minecraft account (ADR-0005)
    And I am publishing the schematic "Victorian House"
    When I additionally attach the world archive "victorian-house-world.zip"
    Then the system shall store its location in world_zip_url of the Schematic
    And the schematic page shall offer the world archive as an optional download
```

---

## US-05 — Manage own publications

**As an** Author,
**I want to** edit the metadata of my own schematics and delete them,
**so that** I can fix mistakes and retire outdated builds.

| Field | Value |
|---|---|
| Priority | High |
| ER entities | `Schematic`, `Category`, `Tag`, `Version`, `User`, `Comment` |
| Related requirements | REQ-SYS-003, REQ-EVN-004, REQ-UNW-002 |
| Use cases | UC-12, UC-15 |

```gherkin
Feature: Managing own publications
  As an Author
  I want to edit or delete my own schematics
  So that I can fix mistakes and retire outdated builds

  Scenario: Edit own publication metadata
    Given I am authenticated as an Author with the nickname "oleg_builder"
    And my Author role was granted through the Microsoft OAuth flow after linking my licensed Minecraft account (ADR-0005)
    And the schematic with the slug "victorian-house" is authored by me
    When I change its title to "Victorian Townhouse" and reassign its category
    Then the system shall update the Schematic accordingly
    And the change shall be visible on the public schematic page

  Scenario: Delete own publication
    Given I am authenticated as an Author with the nickname "oleg_builder"
    And my Author role was granted through the Microsoft OAuth flow after linking my licensed Minecraft account (ADR-0005)
    And the schematic with the slug "old-beta-build" is authored by me
    When I delete that schematic
    Then the system shall remove it from the public catalog
    And other users shall no longer be able to open its schematic page

  Scenario: Editing a foreign publication is denied
    Given I am authenticated as an Author with the nickname "oleg_builder"
    And my Author role was granted through the Microsoft OAuth flow after linking my licensed Minecraft account (ADR-0005)
    And the schematic with the slug "rival-mega-base" is authored by another user
    When I attempt to edit or delete that schematic
    Then the system shall deny the operation
    And no Schematic, Category, Tag or Version data shall be mutated
    And I shall not receive any non-public data of that publication
```

---

## US-06 — Elevate to Author via Microsoft OAuth

**As a** Registered User,
**I want to** link my licensed Minecraft account through Microsoft OAuth so that my account is elevated to the `Author` role,
**so that** I can publish my own schematics.

| Field | Value |
|---|---|
| Priority | High |
| ER entities | `User` (`user_id`) |
| Related requirements | REQ-EVN-004, REQ-UNW-003 |
| Use cases | UC-08, UC-09 |

```gherkin
Feature: Elevating a Registered User to Author via Microsoft OAuth
  As a Registered User
  I want to link my licensed Minecraft account through Microsoft OAuth
  So that I can publish my own schematics

  Scenario: Successful elevation to Author
    Given I am authenticated with the nickname "oleg_builder"
    And my account currently has the Registered User role
    When I start the Microsoft OAuth flow from my profile
    And Microsoft confirms that the linked account owns a licensed copy of Minecraft
    Then the system shall elevate my account to the Author role
    And I shall be allowed to publish schematics

  Scenario: Elevation is not granted without a licensed game account
    Given I am authenticated with the nickname "oleg_builder"
    When I complete the Microsoft OAuth flow without confirming a licensed Minecraft account
    Then the system shall not elevate my account to the Author role
    And I shall remain a Registered User

  Scenario: Publishing is blocked before elevation
    Given I am authenticated with the nickname "oleg_builder"
    And my account has not been elevated to the Author role
    When I attempt to submit a schematic for publication
    Then the system shall deny the operation
    And no Schematic record shall be created
    And the system shall route me to the Microsoft OAuth elevation flow
```

---

## Notes

* `Litematica Parser Service` is a dependent external background service, **not an actor** (`lab2/spec.md`); its behaviour inside scenarios is expressed as preconditions/triggers (`the Litematica Parser Service confirms/reports ...`), which map to REQ-EVN-001, REQ-EVN-002 and REQ-UNW-001.
* Every `Author` scenario (US-04, US-05) starts from an account whose role was elevated through the Microsoft OAuth flow (REQ-EVN-004, ADR-0005): `Registered User` → `Author`. The ER model keeps no `role` attribute, so the elevation introduces no new entities or attributes — it is authorization data of the `User` account.
* All identifiers used above (`slug`, `author_id`, `world_zip_url`, entity names) match the ER model of `lab1/model/er-diagram.mmd` exactly.
* Use case IDs (UC-xx) are defined in `lab2/model/use-case-diagram.puml` and mapped to requirements in `lab2/requirements/traceability-matrix.md`.



