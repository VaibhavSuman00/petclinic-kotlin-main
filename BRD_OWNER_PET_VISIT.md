# Business Requirements Document (BRD)
## Reverse-Engineered from Source Code — Strict Anti-Hallucination Mode

| Field | Value |
|---|---|
| **Module Name** | Owner / Pet / Visit |
| **Document Version** | 1.0 |
| **Date** | April 14, 2026 |
| **Source Repository** | spring-petclinic-kotlin-main |
| **Basis** | Actual Kotlin source code only — no assumptions |

---

## 1.1 Module Name

**Owner / Pet / Visit**

---

## 1.1.1 Overview

### Purpose

This module provides the complete CRUD lifecycle for three tightly coupled domain entities:

- **Owner** — a person registered at the clinic who owns one or more pets
- **Pet** — an animal belonging to an owner, classified by a predefined pet type
- **Visit** — a veterinary appointment record associated with a specific pet

All three sub-domains are implemented within the same Gradle source package (`org.springframework.samples.petclinic.owner`) with the exception of `Visit`, which resides in `org.springframework.samples.petclinic.visit`.

### System Context

The module is a Spring MVC web application backed by Spring Data JPA. Controllers return Thymeleaf view names or redirect URIs. There is **no explicit service layer** — business logic is embedded in controllers, domain entity methods, validators, and formatters. All repository interactions use Spring Data's `Repository<T, ID>` base interface. The default database is H2 (in-memory); MySQL is supported via a profile override.

**No external APIs, no messaging systems, no security filters, no JWT/OAuth are present in this module's source code.**

---

## 1.1.2 Features

### API Features (derived strictly from controller source code)

| Feature | Endpoint(s) | Source File |
|---|---|---|
| Render owner creation form | `GET /owners/new` | `OwnerController.kt` |
| Create new owner | `POST /owners/new` | `OwnerController.kt` |
| Render owner search form | `GET /owners/find` | `OwnerController.kt` |
| Search owners by last name | `GET /owners` | `OwnerController.kt` |
| Display owner details with pets and visits | `GET /owners/{ownerId}` | `OwnerController.kt` |
| Render owner edit form | `GET /owners/{ownerId}/edit` | `OwnerController.kt` |
| Update owner | `POST /owners/{ownerId}/edit` | `OwnerController.kt` |
| Render new pet form for an owner | `GET /owners/{ownerId}/pets/new` | `PetController.kt` |
| Create new pet under an owner | `POST /owners/{ownerId}/pets/new` | `PetController.kt` |
| Render pet edit form | `GET /owners/{ownerId}/pets/{petId}/edit` | `PetController.kt` |
| Update pet | `POST /owners/{ownerId}/pets/{petId}/edit` | `PetController.kt` |
| Render new visit form for a pet | `GET /owners/*/pets/{petId}/visits/new` | `VisitController.kt` |
| Create new visit for a pet | `POST /owners/{ownerId}/pets/{petId}/visits/new` | `VisitController.kt` |

### Service Capabilities (strictly from code)

| Capability | Source |
|---|---|
| Find owners by last-name prefix | `OwnerRepository.findByLastName()` |
| Load owner eagerly with pets (JOIN FETCH) | `OwnerRepository.findById()` |
| Load all pet types ordered by name | `PetRepository.findPetTypes()` |
| Load visits for a specific pet | `VisitRepository.findByPetId()` |
| Validate pet fields programmatically | `PetValidator.validate()` |
| Convert PetType name to/from entity | `PetTypeFormatter.parse()` / `PetTypeFormatter.print()` |
| Duplicate pet name detection per owner | `PetController.processCreationForm()` |

---

## 1.1.3 Functional Requirements

---

### FR-OWN-001 — Render Owner Creation Form

| Attribute | Value |
|---|---|
| **FR-ID** | FR-OWN-001 |
| **Feature Name** | Render Owner Creation Form |
| **Dimension Tags** | FR24, FR31 |
| **Actors** | Not derivable from source code |
| **Preconditions** | None explicitly defined in source code |
| **Postconditions** | Model contains an empty `Owner` object; view `owners/createOrUpdateOwnerForm` is rendered |

#### Business Flow

1. `GET /owners/new` request is received.
2. A new empty `Owner()` instance is added to the model under the key `"owner"`.
3. The view `owners/createOrUpdateOwnerForm` is returned.

#### Technical Trace

| Element | Detail |
|---|---|
| **File Path** | `src/main/kotlin/…/owner/OwnerController.kt` |
| **Class** | `OwnerController` |
| **Method** | `initCreationForm(model: MutableMap<String, Any>): String` |

#### Conditions
- None explicitly defined.

#### Validations
- `@InitBinder` disallows binding of field `"id"` on all request mappings in this controller.

#### Error Handling
- Not explicitly defined in source code.

---

### FR-OWN-002 — Create New Owner

| Attribute | Value |
|---|---|
| **FR-ID** | FR-OWN-002 |
| **Feature Name** | Create New Owner |
| **Dimension Tags** | FR24, FR25, FR26, FR29, FR30, FR31 |
| **Actors** | Not derivable from source code |
| **Preconditions** | None explicitly defined in source code |
| **Postconditions** | If validation passes: owner is saved, HTTP redirect to `/owners/{newId}`. If validation fails: form view is re-rendered with errors. |

#### Business Flow

1. `POST /owners/new` receives a form-bound `Owner` object annotated with `@Valid`.
2. `BindingResult` is checked for validation errors.
3. **Condition — errors present:** return view name `owners/createOrUpdateOwnerForm` (form re-rendered).
4. **Condition — no errors:** call `owners.save(owner)`, then return `"redirect:/owners/" + owner.id`.

#### Technical Trace

| Element | Detail |
|---|---|
| **File Path** | `src/main/kotlin/…/owner/OwnerController.kt` |
| **Class** | `OwnerController` |
| **Method** | `processCreationForm(@Valid owner: Owner, result: BindingResult): String` |

#### Validations (explicit)

Enforced by Jakarta Bean Validation annotations on `Owner` / `Person` superclasses:

| Field | Annotation | Rule | Source File |
|---|---|---|---|
| `firstName` | `@NotEmpty` | Non-blank required | `Person.kt` |
| `lastName` | `@NotEmpty` | Non-blank required | `Person.kt` |
| `address` | `@NotEmpty` | Non-blank required | `Owner.kt` |
| `city` | `@NotEmpty` | Non-blank required | `Owner.kt` |
| `telephone` | `@NotEmpty` | Non-blank required | `Owner.kt` |
| `telephone` | `@Digits(fraction=0, integer=10)` | Numeric only, max 10 digits | `Owner.kt` |

#### Business Rules (explicit)
- Field `id` is explicitly disallowed from form binding via `dataBinder.setDisallowedFields("id")` — prevents mass-assignment of the primary key.

#### Error Handling
- Validation errors are collected in `BindingResult`; no explicit `try-catch` blocks present.

#### Edge Cases
- `owner.id` is used in the redirect string immediately after `owners.save(owner)`. If the save fails (e.g., constraint violation), no explicit catch is coded — exception propagates to Spring's default error handler.

---

### FR-OWN-003 — Render Owner Search Form

| Attribute | Value |
|---|---|
| **FR-ID** | FR-OWN-003 |
| **Feature Name** | Render Owner Search Form |
| **Dimension Tags** | FR24, FR31 |
| **Actors** | Not derivable from source code |
| **Preconditions** | None |
| **Postconditions** | Model contains empty `Owner` object; view `owners/findOwners` is rendered |

#### Business Flow

1. `GET /owners/find` is received.
2. An empty `Owner()` is added to model under key `"owner"`.
3. View `owners/findOwners` is returned.

#### Technical Trace

| Element | Detail |
|---|---|
| **File Path** | `src/main/kotlin/…/owner/OwnerController.kt` |
| **Class** | `OwnerController` |
| **Method** | `initFindForm(model: MutableMap<String, Any>): String` |

#### Error Handling
- Not explicitly defined in source code.

---

### FR-OWN-004 — Search Owners by Last Name

| Attribute | Value |
|---|---|
| **FR-ID** | FR-OWN-004 |
| **Feature Name** | Search Owners by Last Name |
| **Dimension Tags** | FR24, FR25, FR27, FR29, FR30, FR31 |
| **Actors** | Not derivable from source code |
| **Preconditions** | `owner.lastName` is bound from form/query parameter |
| **Postconditions** | One of three outcomes: validation error, single-owner redirect, or multi-owner list view |

#### Business Flow

1. `GET /owners` is received with `owner.lastName` form-bound.
2. `owners.findByLastName(owner.lastName)` is called — returns `Collection<Owner>`.
3. **Condition — results are empty:**
   - `result.rejectValue("lastName", "notFound", "not found")` is called.
   - View `owners/findOwners` is returned.
4. **Condition — results.size == 1:**
   - Return `"redirect:/owners/" + results.first().id`.
5. **Condition — results.size > 1:**
   - Add `results` to model under key `"selections"`.
   - Return view `owners/ownersList`.

#### Technical Trace

| Element | Detail |
|---|---|
| **File Path** | `src/main/kotlin/…/owner/OwnerController.kt` |
| **Class** | `OwnerController` |
| **Method** | `processFindForm(owner: Owner, result: BindingResult, model: MutableMap<String, Any>): String` |
| **Repository Call** | `OwnerRepository.findByLastName(lastName: String): Collection<Owner>` |
| **Repository File** | `src/main/kotlin/…/owner/OwnerRepository.kt` |
| **JPQL Query** | `SELECT DISTINCT owner FROM Owner owner left join fetch owner.pets WHERE owner.lastName LIKE :lastName%` |

#### Business Rules (explicit)
- The JPQL query uses a prefix-match pattern (`LIKE :lastName%`). It is **not** a full-string match.
- Query uses `LEFT JOIN FETCH owner.pets` — pets are eagerly loaded in this query.
- H2 schema declares `last_name` as `VARCHAR_IGNORECASE(30)` — case-insensitive matching at DB level for H2.

#### Error Handling
- No results: `result.rejectValue("lastName", "notFound", "not found")` — field-level error rejecting `lastName` with error code `"notFound"` and default message `"not found"`.

---

### FR-OWN-005 — Display Owner Details

| Attribute | Value |
|---|---|
| **FR-ID** | FR-OWN-005 |
| **Feature Name** | Display Owner Details with Pets and Visits |
| **Dimension Tags** | FR24, FR25, FR26, FR31 |
| **Actors** | Not derivable from source code |
| **Preconditions** | Owner with `ownerId` must exist in the database |
| **Postconditions** | Model contains the `Owner` with all pets, each pet populated with its visits; view `owners/ownerDetails` rendered |

#### Business Flow

1. `GET /owners/{ownerId}` is received.
2. `owners.findById(ownerId)` is called — returns `Owner` with pets eagerly loaded via JOIN FETCH.
3. **Loop — for each pet in `owner.getPets()`:**
   - `pet.visits = visits.findByPetId(pet.id!!)` is called.
   - Visits are attached to the `@Transient` `visits` field on `Pet`.
4. `model.addAttribute(owner)` is called.
5. View `owners/ownerDetails` is returned.

#### Technical Trace

| Element | Detail |
|---|---|
| **File Path** | `src/main/kotlin/…/owner/OwnerController.kt` |
| **Class** | `OwnerController` |
| **Method** | `showOwner(@PathVariable ownerId: Int, model: Model): String` |
| **Repository Call 1** | `OwnerRepository.findById(id: Int): Owner` |
| **JPQL — findById** | `SELECT owner FROM Owner owner left join fetch owner.pets WHERE owner.id =:id` |
| **Repository Call 2** | `VisitRepository.findByPetId(petId: Int): MutableSet<Visit>` |
| **Repository File 1** | `src/main/kotlin/…/owner/OwnerRepository.kt` |
| **Repository File 2** | `src/main/kotlin/…/visit/VisitRepository.kt` |

#### Business Rules (explicit)
- `pet.id!!` — a non-null assertion operator is used directly; if `pet.id` is null at this point, a `KotlinNullPointerException` is thrown.
- `owner.getPets()` returns pets sorted by `name` (ascending) — defined in `Owner.getPets()`.

#### Edge Cases
- If `ownerId` does not exist: `OwnerRepository.findById()` is backed by Spring Data's `Repository<Owner, Int>`. No explicit `Optional` unwrapping or null handling is coded in the controller. Runtime exception behavior depends on Spring Data JPA implementation.

---

### FR-OWN-006 — Render Owner Edit Form

| Attribute | Value |
|---|---|
| **FR-ID** | FR-OWN-006 |
| **Feature Name** | Render Owner Edit Form |
| **Dimension Tags** | FR24, FR31 |
| **Actors** | Not derivable from source code |
| **Preconditions** | Owner with `ownerId` must exist |
| **Postconditions** | Model contains existing `Owner` data; view `owners/createOrUpdateOwnerForm` rendered |

#### Business Flow

1. `GET /owners/{ownerId}/edit` is received.
2. `owners.findById(ownerId)` is called.
3. `model.addAttribute(owner)` is called.
4. View `owners/createOrUpdateOwnerForm` is returned.

#### Technical Trace

| Element | Detail |
|---|---|
| **File Path** | `src/main/kotlin/…/owner/OwnerController.kt` |
| **Class** | `OwnerController` |
| **Method** | `initUpdateOwnerForm(@PathVariable ownerId: Int, model: Model): String` |

---

### FR-OWN-007 — Update Owner

| Attribute | Value |
|---|---|
| **FR-ID** | FR-OWN-007 |
| **Feature Name** | Update Owner |
| **Dimension Tags** | FR24, FR25, FR26, FR29, FR30, FR31 |
| **Actors** | Not derivable from source code |
| **Preconditions** | Owner with `ownerId` must exist |
| **Postconditions** | If validation passes: owner ID is set to path variable and owner is saved; redirect to `/owners/{ownerId}`. If errors: form re-rendered. |

#### Business Flow

1. `POST /owners/{ownerId}/edit` is received with form-bound, `@Valid` `Owner` object.
2. `BindingResult` is checked.
3. **Condition — errors present:** return view `owners/createOrUpdateOwnerForm`.
4. **Condition — no errors:**
   - `owner.id = ownerId` is explicitly set from the `@PathVariable`.
   - `owners.save(owner)` is called.
   - Return `"redirect:/owners/{ownerId}"`.

#### Technical Trace

| Element | Detail |
|---|---|
| **File Path** | `src/main/kotlin/…/owner/OwnerController.kt` |
| **Class** | `OwnerController` |
| **Method** | `processUpdateOwnerForm(@Valid owner: Owner, result: BindingResult, @PathVariable ownerId: Int): String` |

#### Business Rules (explicit)
- The `id` field is explicitly disallowed from form binding. The controller forcibly assigns `owner.id = ownerId` from the path variable — this is the mechanism ensuring the correct record is updated regardless of form submission.

#### Validations
- Same Jakarta Bean Validation rules as FR-OWN-002.

---

### FR-PET-001 — Render New Pet Form

| Attribute | Value |
|---|---|
| **FR-ID** | FR-PET-001 |
| **Feature Name** | Render New Pet Form |
| **Dimension Tags** | FR24, FR26, FR31 |
| **Actors** | Not derivable from source code |
| **Preconditions** | Owner with `ownerId` must exist |
| **Postconditions** | Model contains `owner`, `types` (pet type list), and a new unsaved `Pet`; view `pets/createOrUpdatePetForm` rendered |

#### Business Flow

1. `GET /owners/{ownerId}/pets/new` is received.
2. `@ModelAttribute("owner")` is resolved via `findOwner(ownerId)` — calls `owners.findById(ownerId)`.
3. `@ModelAttribute("types")` is resolved via `populatePetTypes()` — calls `pets.findPetTypes()`.
4. New `Pet()` is created; `owner.addPet(pet)` is called.
5. `model["pet"] = pet` is set.
6. View `pets/createOrUpdatePetForm` is returned.

#### Technical Trace

| Element | Detail |
|---|---|
| **File Path** | `src/main/kotlin/…/owner/PetController.kt` |
| **Class** | `PetController` |
| **Method** | `initCreationForm(owner: Owner, model: Model): String` |
| **Model Attr Method** | `populatePetTypes(): Collection<PetType>` |
| **Model Attr Method** | `findOwner(@PathVariable ownerId: Int): Owner` |
| **Repository Call** | `PetRepository.findPetTypes(): List<PetType>` |
| **JPQL** | `SELECT ptype FROM PetType ptype ORDER BY ptype.name` |

---

### FR-PET-002 — Create New Pet

| Attribute | Value |
|---|---|
| **FR-ID** | FR-PET-002 |
| **Feature Name** | Create New Pet Under Owner |
| **Dimension Tags** | FR24, FR25, FR26, FR29, FR30, FR31 |
| **Actors** | Not derivable from source code |
| **Preconditions** | Owner with `ownerId` must exist |
| **Postconditions** | If validation passes: pet is saved; redirect to `/owners/{ownerId}`. If errors: form re-rendered. |

#### Business Flow

1. `POST /owners/{ownerId}/pets/new` received; `owner` resolved from `@ModelAttribute`.
2. `@Valid pet` and `BindingResult` are present (PetValidator is set via `@InitBinder`).
3. **Condition — duplicate name check:**
   - IF `StringUtils.hasLength(pet.name)` AND `pet.isNew` AND `owner.getPet(pet.name!!, true) != null`
   - THEN `result.rejectValue("name", "duplicate", "already exists")`
4. `owner.addPet(pet)` is called unconditionally.
5. **Condition — errors in BindingResult:**
   - `model["pet"] = pet`
   - Return view `pets/createOrUpdatePetForm`.
6. **Condition — no errors:**
   - `pets.save(pet)`
   - Return `"redirect:/owners/{ownerId}"`.

#### Technical Trace

| Element | Detail |
|---|---|
| **File Path** | `src/main/kotlin/…/owner/PetController.kt` |
| **Class** | `PetController` |
| **Method** | `processCreationForm(owner: Owner, @Valid pet: Pet, result: BindingResult, model: Model): String` |
| **Validator** | `PetValidator` (registered via `@InitBinder("pet")`) |
| **Domain Method** | `Owner.getPet(name: String, ignoreNew: Boolean)` |
| **Domain Method** | `Owner.addPet(pet: Pet)` |

#### Business Rules (explicit)
- Duplicate pet name is checked with `ignoreNew = true`, meaning only persisted pets (with non-null `id`) are considered for the duplicate check.
- `owner.addPet(pet)` adds the pet to the set only if `pet.isNew`, then always sets `pet.owner = this`.

#### Validations (PetValidator — explicit)

| Field | Rule | Source File |
|---|---|---|
| `name` | `StringUtils.hasLength(name)` must be true | `PetValidator.kt` |
| `type` | Non-null required **only** when `pet.isNew == true` | `PetValidator.kt` |
| `birthDate` | Non-null required | `PetValidator.kt` |

Error codes used: `"required"` (static companion constant `REQUIRED`).

#### Error Handling
- Field-level errors set via `errors.rejectValue(field, code, message)`. No `try-catch` blocks.

---

### FR-PET-003 — Render Pet Edit Form

| Attribute | Value |
|---|---|
| **FR-ID** | FR-PET-003 |
| **Feature Name** | Render Pet Edit Form |
| **Dimension Tags** | FR24, FR26, FR31 |
| **Actors** | Not derivable from source code |
| **Preconditions** | Pet with `petId` must exist; owner with `ownerId` must exist |
| **Postconditions** | Model contains loaded `Pet`; view `pets/createOrUpdatePetForm` rendered |

#### Business Flow

1. `GET /owners/{ownerId}/pets/{petId}/edit` received.
2. `pets.findById(petId)` called; result placed in `model["pet"]`.
3. View `pets/createOrUpdatePetForm` returned.

#### Technical Trace

| Element | Detail |
|---|---|
| **File Path** | `src/main/kotlin/…/owner/PetController.kt` |
| **Class** | `PetController` |
| **Method** | `initUpdateForm(@PathVariable petId: Int, model: Model): String` |

---

### FR-PET-004 — Update Pet

| Attribute | Value |
|---|---|
| **FR-ID** | FR-PET-004 |
| **Feature Name** | Update Pet |
| **Dimension Tags** | FR24, FR25, FR26, FR29, FR30, FR31 |
| **Actors** | Not derivable from source code |
| **Preconditions** | Pet and owner must exist |
| **Postconditions** | If no errors: pet updated and redirect. If errors: form re-rendered. |

#### Business Flow

1. `POST /owners/{ownerId}/pets/{petId}/edit` received.
2. **Condition — errors in BindingResult:**
   - `pet.owner = owner` explicitly set.
   - `model["pet"] = pet`.
   - Return view `pets/createOrUpdatePetForm`.
3. **Condition — no errors:**
   - `owner.addPet(pet)` called.
   - `pets.save(pet)` called.
   - Return `"redirect:/owners/{ownerId}"`.

#### Technical Trace

| Element | Detail |
|---|---|
| **File Path** | `src/main/kotlin/…/owner/PetController.kt` |
| **Class** | `PetController` |
| **Method** | `processUpdateForm(@Valid pet: Pet, result: BindingResult, owner: Owner, model: Model): String` |

---

### FR-VIS-001 — Render New Visit Form

| Attribute | Value |
|---|---|
| **FR-ID** | FR-VIS-001 |
| **Feature Name** | Render New Visit Form |
| **Dimension Tags** | FR24, FR26, FR31 |
| **Actors** | Not derivable from source code |
| **Preconditions** | Pet with `petId` must exist |
| **Postconditions** | Model contains `pet` and a new `Visit` instance attached to the pet; view `pets/createOrUpdateVisitForm` rendered |

#### Business Flow

1. `GET /owners/*/pets/{petId}/visits/new` received.
2. `@ModelAttribute("visit")` — `loadPetWithVisit(petId, model)` is invoked:
   - `pets.findById(petId)` called.
   - `model["pet"] = pet`.
   - New `Visit()` created.
   - `pet.addVisit(visit)` called — this sets `visit.petId = pet.id`.
   - Visit returned as model attribute.
3. View `pets/createOrUpdateVisitForm` returned.

#### Technical Trace

| Element | Detail |
|---|---|
| **File Path** | `src/main/kotlin/…/owner/VisitController.kt` |
| **Class** | `VisitController` |
| **Method** | `initNewVisitForm(@PathVariable petId: Int, model: Map<String, Any>): String` |
| **Model Attr Method** | `loadPetWithVisit(@PathVariable petId: Int, model: MutableMap<String, Any>): Visit` |
| **Domain Method** | `Pet.addVisit(visit: Visit)` |

#### Business Rules (explicit)
- `Pet.addVisit(visit)` sets `visit.petId = this.id`. This is the mechanism that links the visit to the pet via a plain integer FK (no JPA association on Visit side).

---

### FR-VIS-002 — Create New Visit

| Attribute | Value |
|---|---|
| **FR-ID** | FR-VIS-002 |
| **Feature Name** | Create New Visit for Pet |
| **Dimension Tags** | FR24, FR25, FR26, FR29, FR30, FR31 |
| **Actors** | Not derivable from source code |
| **Preconditions** | Pet with `petId` must exist; `loadPetWithVisit` runs before this method |
| **Postconditions** | If no errors: visit is saved; redirect to `/owners/{ownerId}`. If errors: form re-rendered. |

#### Business Flow

1. `POST /owners/{ownerId}/pets/{petId}/visits/new` received.
2. `@ModelAttribute("visit")` — `loadPetWithVisit` runs first (see FR-VIS-001 flow).
3. `@Valid visit` and `BindingResult` evaluated.
4. **Condition — errors in BindingResult:**
   - Return view `pets/createOrUpdateVisitForm`.
5. **Condition — no errors:**
   - `visits.save(visit)` called.
   - Return `"redirect:/owners/{ownerId}"`.

#### Technical Trace

| Element | Detail |
|---|---|
| **File Path** | `src/main/kotlin/…/owner/VisitController.kt` |
| **Class** | `VisitController` |
| **Method** | `processNewVisitForm(@Valid visit: Visit, result: BindingResult): String` |
| **Repository Call** | `VisitRepository.save(visit: Visit)` |

#### Validations (explicit)

| Field | Annotation | Rule | Source File |
|---|---|---|---|
| `description` | `@NotEmpty` | Non-blank required | `Visit.kt` |

#### Business Rules (explicit)
- `Visit.date` defaults to `LocalDate.now()` at instantiation. This default is set in the entity class, not in the controller.
- `@InitBinder` disallows binding of field `"id"` in `VisitController`.

#### Error Handling
- No `try-catch` present. `BindingResult` is checked; errors route back to form.

---

## 1.1.4 State Transitions

### Owner

```
[New Owner form displayed]
    → [POST with valid data] → Owner saved → Redirect to Owner details
    → [POST with invalid data] → Form re-rendered with errors
```

### Pet

```
[New Pet form displayed for owner]
    → [POST with valid data, no duplicate name] → Pet saved → Redirect to Owner details
    → [POST with valid data, duplicate name] → Form re-rendered with "already exists" error
    → [POST with invalid data] → Form re-rendered with errors
```

### Visit

```
[New Visit form displayed for pet]
    → [POST with valid data] → Visit saved → Redirect to Owner details
    → [POST with invalid data] → Form re-rendered with errors
```

---

## 1.1.5 Calculations / Algorithms

### Algorithm: Owner.getPets() — Sorted Pet List

```kotlin
// File: src/main/kotlin/…/owner/Owner.kt
// Class: Owner
// Method: getPets()
fun getPets(): List<Pet> = pets.sortedWith(compareBy({ it.name }))
```

**Behavior:** Returns all pets in ascending alphabetical order by `name`. Applied in `showOwner` before visits are loaded.

---

### Algorithm: Pet.getVisits() — Sorted Visit List

```kotlin
// File: src/main/kotlin/…/owner/Pet.kt
// Class: Pet
// Method: getVisits()
fun getVisits(): List<Visit> = visits.sortedWith(compareBy { it.date })
```

**Behavior:** Returns all visits in ascending chronological order by `date`.

---

### Algorithm: Owner.addPet() — Add Pet to Owner

```kotlin
// File: src/main/kotlin/…/owner/Owner.kt
// Class: Owner
// Method: addPet(pet: Pet)
fun addPet(pet: Pet) {
    if (pet.isNew) {
        pets.add(pet)
    }
    pet.owner = this
}
```

**Behavior:**
- If `pet.id == null` (new, not yet persisted): pet is added to the `pets` set.
- Always: `pet.owner` back-reference is set to `this` owner.

---

### Algorithm: Owner.getPet() — Find Pet by Name

```kotlin
// File: src/main/kotlin/…/owner/Owner.kt
// Class: Owner — full method body not present in the portion read
// Used with: getPet(name: String, ignoreNew: Boolean)
```

**Behavior:** Finds a pet by name in the owner's pet set. When `ignoreNew = true`, unsaved pets (where `pet.isNew == true`) are excluded from the match (used for duplicate detection during creation).

---

### Algorithm: Pet.addVisit() — Link Visit to Pet

```kotlin
// File: src/main/kotlin/…/owner/Pet.kt
// Class: Pet
// Method: addVisit(visit: Visit)
fun addVisit(visit: Visit) {
    visits.add(visit)
    visit.petId = this.id
}
```

**Behavior:** Adds visit to transient set AND propagates `pet.id` to `visit.petId`.

---

### Algorithm: PetTypeFormatter — Parse / Print

```kotlin
// File: src/main/kotlin/…/owner/PetTypeFormatter.kt
// Class: PetTypeFormatter

override fun print(petType: PetType, locale: Locale): String = petType.name ?: ""

override fun parse(text: String, locale: Locale): PetType {
    val findPetTypes = this.pets.findPetTypes()
    return findPetTypes.find { it.name == text } ?:
                throw ParseException("type not found: " + text, 0)
}
```

**Behavior:**
- `print`: Returns `name` of `PetType`, or empty string if null.
- `parse`: Does an exact name match against all pet types loaded from DB. Throws `java.text.ParseException` with message `"type not found: {text}"` at position 0 if not found.

---

## 1.1.6 Data Model Summary

### Entity: `Owner`

| Field | Kotlin Type | Nullable | Default | DB Column | Source File |
|---|---|---|---|---|---|
| `id` (inherited) | `Int?` | Nullable | `null` | `id` (PK, IDENTITY) | `BaseEntity.kt` |
| `firstName` (inherited) | `String` | Non-nullable | `""` | `first_name` | `Person.kt` |
| `lastName` (inherited) | `String` | Non-nullable | `""` | `last_name` | `Person.kt` |
| `address` | `String` | Non-nullable | `""` | `address` | `Owner.kt` |
| `city` | `String` | Non-nullable | `""` | `city` | `Owner.kt` |
| `telephone` | `String` | Non-nullable | `""` | `telephone` | `Owner.kt` |
| `pets` | `MutableSet<Pet>` | Non-nullable | `HashSet()` | — (via FK `owner_id`) | `Owner.kt` |

**JPA:** `@Entity`, `@Table(name = "owners")`, `@OneToMany(cascade = [CascadeType.ALL], mappedBy = "owner")`

---

### Entity: `Pet`

| Field | Kotlin Type | Nullable | Default | DB Column | Source File |
|---|---|---|---|---|---|
| `id` (inherited) | `Int?` | Nullable | `null` | `id` (PK, IDENTITY) | `BaseEntity.kt` |
| `name` (inherited) | `String?` | Nullable | `null` | `name` | `NamedEntity.kt` |
| `birthDate` | `LocalDate?` | Nullable | `null` | `birth_date` | `Pet.kt` |
| `type` | `PetType?` | Nullable | `null` | `type_id` (FK) | `Pet.kt` |
| `owner` | `Owner?` | Nullable | `null` | `owner_id` (FK) | `Pet.kt` |
| `visits` | `MutableSet<Visit>` | Non-nullable | `LinkedHashSet()` | — (`@Transient`) | `Pet.kt` |

**JPA:** `@Entity`, `@Table(name = "pets")`, `@ManyToOne` on `type` and `owner`, `@Transient` on `visits`

---

### Entity: `PetType`

| Field | Kotlin Type | Nullable | Default | DB Column | Source File |
|---|---|---|---|---|---|
| `id` (inherited) | `Int?` | Nullable | `null` | `id` (PK, IDENTITY) | `BaseEntity.kt` |
| `name` (inherited) | `String?` | Nullable | `null` | `name` | `NamedEntity.kt` |

**JPA:** `@Entity`, `@Table(name = "types")`  
**Note:** Class body is empty — all fields inherited from `NamedEntity` → `BaseEntity`.

---

### Entity: `Visit`

| Field | Kotlin Type | Nullable | Default | DB Column | Source File |
|---|---|---|---|---|---|
| `id` (inherited) | `Int?` | Nullable | `null` | `id` (PK, IDENTITY) | `BaseEntity.kt` |
| `date` | `LocalDate` | Non-nullable | `LocalDate.now()` | `visit_date` | `Visit.kt` |
| `description` | `String?` | Nullable | `null` | `description` | `Visit.kt` |
| `petId` | `Int?` | Nullable | `null` | `pet_id` (FK) | `Visit.kt` |

**JPA:** `@Entity`, `@Table(name = "visits")`  
**Note:** `petId` is a plain `Int?` column, NOT a `@ManyToOne` JPA association.

---

### Abstract Base: `BaseEntity`

| Field | Kotlin Type | Nullable | Default | Source File |
|---|---|---|---|---|
| `id` | `Int?` | Nullable | `null` | `BaseEntity.kt` |
| `isNew` (computed) | `Boolean` | Non-nullable | — | `BaseEntity.kt` |

---

### Abstract Base: `NamedEntity`

| Field | Kotlin Type | Nullable | Default | Source File |
|---|---|---|---|---|
| `name` | `String?` | Nullable | `null` | `NamedEntity.kt` |

---

### Abstract Base: `Person`

| Field | Kotlin Type | Nullable | Default | Source File |
|---|---|---|---|---|
| `firstName` | `String` | Non-nullable | `""` | `Person.kt` |
| `lastName` | `String` | Non-nullable | `""` | `Person.kt` |

---

## 1.1.7 API Mapping Table

| Endpoint | HTTP Method | FR-ID | Description | File | Class | Method |
|---|---|---|---|---|---|---|
| `/owners/new` | GET | FR-OWN-001 | Render owner creation form | `OwnerController.kt` | `OwnerController` | `initCreationForm` |
| `/owners/new` | POST | FR-OWN-002 | Save new owner | `OwnerController.kt` | `OwnerController` | `processCreationForm` |
| `/owners/find` | GET | FR-OWN-003 | Render owner search form | `OwnerController.kt` | `OwnerController` | `initFindForm` |
| `/owners` | GET | FR-OWN-004 | Search owners by last name | `OwnerController.kt` | `OwnerController` | `processFindForm` |
| `/owners/{ownerId}` | GET | FR-OWN-005 | Display owner details with pets and visits | `OwnerController.kt` | `OwnerController` | `showOwner` |
| `/owners/{ownerId}/edit` | GET | FR-OWN-006 | Render owner edit form | `OwnerController.kt` | `OwnerController` | `initUpdateOwnerForm` |
| `/owners/{ownerId}/edit` | POST | FR-OWN-007 | Update owner | `OwnerController.kt` | `OwnerController` | `processUpdateOwnerForm` |
| `/owners/{ownerId}/pets/new` | GET | FR-PET-001 | Render new pet form | `PetController.kt` | `PetController` | `initCreationForm` |
| `/owners/{ownerId}/pets/new` | POST | FR-PET-002 | Create new pet | `PetController.kt` | `PetController` | `processCreationForm` |
| `/owners/{ownerId}/pets/{petId}/edit` | GET | FR-PET-003 | Render pet edit form | `PetController.kt` | `PetController` | `initUpdateForm` |
| `/owners/{ownerId}/pets/{petId}/edit` | POST | FR-PET-004 | Update pet | `PetController.kt` | `PetController` | `processUpdateForm` |
| `/owners/*/pets/{petId}/visits/new` | GET | FR-VIS-001 | Render new visit form | `VisitController.kt` | `VisitController` | `initNewVisitForm` |
| `/owners/{ownerId}/pets/{petId}/visits/new` | POST | FR-VIS-002 | Save new visit | `VisitController.kt` | `VisitController` | `processNewVisitForm` |

---

## 1.1.8 — 8-Dimensional BRD Coverage

| Dimension | Status | Notes |
|---|---|---|
| **FR24 – Functional Requirements** | ✅ Covered | 9 FRs extracted from 3 controllers across 13 endpoints |
| **FR25 – Business Rules** | ✅ Covered | Prefix-match query, duplicate pet name check, ID disallow, petId propagation, forced id assignment on update |
| **FR26 – Data Models** | ✅ Covered | 4 entities + 3 base superclasses fully documented |
| **FR27 – Integration Points** | ➡ Not explicitly derivable | No external APIs, messaging systems, or third-party services present in source code |
| **FR28 – Security & Compliance** | ➡ Partially derivable | `dataBinder.setDisallowedFields("id")` present in all three controllers — prevents mass-assignment of PK. No JWT, OAuth2, roles, or security filters present in this module's source code |
| **FR29 – Error Handling** | ✅ Covered | `result.rejectValue()` calls documented per FR. `ParseException` in `PetTypeFormatter`. No try-catch blocks present. |
| **FR30 – Edge Cases & Boundary Conditions** | ✅ Covered | Null assertion `pet.id!!`, missing owner behavior, `pet.isNew` conditional, `ignoreNew` flag in duplicate check, `ParseException` on unknown pet type |
| **FR31 – User Journeys & Workflows** | ✅ Covered | All three CRUD flows documented with branching conditions |

---

## 1.1.9 Repository / Database Reference

| Repository | Method | Query / Derivation | Transactional | Cacheable | Source File |
|---|---|---|---|---|---|
| `OwnerRepository` | `findByLastName(lastName: String)` | `SELECT DISTINCT owner FROM Owner owner left join fetch owner.pets WHERE owner.lastName LIKE :lastName%` | `readOnly = true` | No | `OwnerRepository.kt` |
| `OwnerRepository` | `findById(id: Int)` | `SELECT owner FROM Owner owner left join fetch owner.pets WHERE owner.id =:id` | `readOnly = true` | No | `OwnerRepository.kt` |
| `OwnerRepository` | `save(owner: Owner)` | Spring Data default | No | No | `OwnerRepository.kt` |
| `PetRepository` | `findPetTypes()` | `SELECT ptype FROM PetType ptype ORDER BY ptype.name` | `readOnly = true` | No | `PetRepository.kt` |
| `PetRepository` | `findById(id: Int)` | Spring Data default | `readOnly = true` | No | `PetRepository.kt` |
| `PetRepository` | `save(pet: Pet)` | Spring Data default | No | No | `PetRepository.kt` |
| `VisitRepository` | `save(visit: Visit)` | Spring Data default | No | No | `VisitRepository.kt` |
| `VisitRepository` | `findByPetId(petId: Int)` | Spring Data method name derivation (`pet_id = ?`) | No | No | `VisitRepository.kt` |

---

## 1.1.10 Security Notes

| Mechanism | Detail | Source |
|---|---|---|
| `setDisallowedFields("id")` | Prevents form binding of `id` field in `OwnerController` | `OwnerController.kt` — `setAllowedFields(@InitBinder)` |
| `setDisallowedFields("id")` on `owner` | Prevents form binding of `id` field for `Owner` model attribute in `PetController` | `PetController.kt` — `initOwnerBinder(@InitBinder("owner"))` |
| `setDisallowedFields("id")` | Prevents form binding of `id` field in `VisitController` | `VisitController.kt` — `setAllowedFields(@InitBinder)` |

**No authentication, no authorization, no JWT, no OAuth2, no security filters are present in this module's source code.**

---

## Self-Validation Checklist

| Check | Status |
|---|---|
| No assumptions made beyond source code | ✅ |
| All 13 endpoints mapped | ✅ |
| All 3 controllers covered | ✅ |
| All 3 repositories covered | ✅ |
| All 4 entities + 3 base classes documented | ✅ |
| Validator and Formatter covered | ✅ |
| All FR24–FR31 dimensions present | ✅ |
| Missing items clearly marked | ✅ (FR27, FR28 — no integration/auth in code) |
| Full traceability (file, class, method) on all FRs | ✅ |
| No inferred business logic — only explicit code | ✅ |
