# Comprehensive Summary Document
## Spring Petclinic Kotlin — Owner / Pet / Visit Module

| Field | Value |
|---|---|
| **Document Type** | Consolidated Summary (BRD · BA · HLD · Technical) |
| **Module Scope** | Owner / Pet / Visit |
| **Application** | Spring Petclinic Kotlin |
| **Version** | 1.0 |
| **Date** | April 14, 2026 |
| **Source Documents** | BRD.md · BRD_OWNER_PET_VISIT.md · TECH_WORKBOOK.md |
| **Language** | Kotlin 2.3.20 · Spring Boot 4.0.4 · Java 17 |

---

## Table of Contents

| # | Section | Document Origin |
|---|---|---|
| 1 | [Executive Summary](#1-executive-summary) | BRD |
| 2 | [Business Objectives](#2-business-objectives) | BRD |
| 3 | [Stakeholders](#3-stakeholders) | BRD / BA |
| 4 | [Project Scope](#4-project-scope) | BRD |
| 5 | [Business Analysis (BA)](#5-business-analysis-ba) | BA |
| 6 | [Business Requirements (BRD)](#6-business-requirements-brd) | BRD |
| 7 | [Functional Requirements — Detailed](#7-functional-requirements--detailed) | BRD (Reverse-Engineered) |
| 8 | [Business Rules](#8-business-rules) | BRD / BA |
| 9 | [High-Level Design (HLD)](#9-high-level-design-hld) | HLD |
| 10 | [Data Model](#10-data-model) | HLD / Technical |
| 11 | [Database Design](#11-database-design) | HLD / Technical |
| 12 | [API Design](#12-api-design) | HLD / Technical |
| 13 | [Technical Architecture](#13-technical-architecture) | Technical |
| 14 | [Technology Stack](#14-technology-stack) | Technical |
| 15 | [Component Design](#15-component-design) | Technical |
| 16 | [Validation & Error Handling](#16-validation--error-handling) | Technical / BRD |
| 17 | [Caching Strategy](#17-caching-strategy) | Technical |
| 18 | [Testing Strategy](#18-testing-strategy) | Technical / BA |
| 19 | [Deployment Architecture](#19-deployment-architecture) | Technical / HLD |
| 20 | [Key Design Decisions](#20-key-design-decisions) | Technical / HLD |
| 21 | [Constraints & Assumptions](#21-constraints--assumptions) | BRD |
| 22 | [Acceptance Criteria](#22-acceptance-criteria) | BRD / BA |
| 23 | [Open Questions & Decisions Required](#23-open-questions--decisions-required) | BA |
| 24 | [Glossary](#24-glossary) | BRD |

---

## 1. Executive Summary

The **Spring Petclinic Kotlin** application is a veterinary clinic management system that digitizes the day-to-day administrative operations of a small-to-medium pet clinic.

The **Owner / Pet / Visit module** is the core operational module of the application. It provides a complete CRUD (Create, Read, Update) lifecycle for three tightly coupled domain entities:

| Entity | Responsibility |
|---|---|
| **Owner** | A registered person at the clinic who owns one or more pets |
| **Pet** | An animal belonging to an owner, classified by a predefined pet type |
| **Visit** | A veterinary appointment record linked to a specific pet |

The module is implemented as a **Spring MVC web application** backed by **Spring Data JPA**, serving server-rendered HTML via **Thymeleaf** templates. It contains **13 HTTP endpoints** across **3 controllers**, backed by **3 repositories** and **4 JPA entities**.

This document consolidates all Business Analysis, Business Requirements, High-Level Design, and Technical findings into a single reference artifact.

---

## 2. Business Objectives

| ID | Objective | Priority |
|---|---|---|
| BO-01 | Eliminate paper-based record keeping by providing a centralized digital repository for all clinic data | Must Have |
| BO-02 | Enable staff to quickly locate any owner or pet by name, reducing front-desk lookup time | Must Have |
| BO-03 | Provide a complete, chronological visit history for every pet so veterinarians can make informed clinical decisions | Must Have |
| BO-04 | Deliver a web-based interface accessible from any modern browser without client-side software installation | Must Have |
| BO-05 | Support dual-database deployment (H2 for development, MySQL for production) without code changes | Should Have |
| BO-06 | Serve as a reference implementation demonstrating Spring Boot best practices using Kotlin | Nice to Have |

---

## 3. Stakeholders

| Role | Responsibility | Interaction with Module |
|---|---|---|
| **Clinic Receptionist** | Primary end user — registers owners, adds pets, records visits | All 13 endpoints |
| **Veterinarian** | Reviews pet visit history during consultations | `GET /owners/{ownerId}` |
| **Clinic Manager** | Oversees business operations; ensures data accuracy | Reporting / admin view |
| **System Administrator** | Deploys and configures application; manages database | Configuration, deployment |
| **Development Team** | Implements, tests, and maintains the codebase | All source files |

---

## 4. Project Scope

### 4.1 In Scope

| Area | Detail |
|---|---|
| Owner CRUD | Register, search by last name, view details, edit |
| Pet CRUD | Add pets to owners, edit pet details, assign pet type |
| Visit Recording | Log veterinary visits with date and description |
| Pet Type Catalog | Predefined catalog: cat, dog, lizard, snake, bird, hamster |
| Server-rendered UI | Thymeleaf HTML templates with Bootstrap styling |
| Form Validation | Jakarta Bean Validation + custom `PetValidator` |
| Dual Database | H2 (default, in-memory) + MySQL (profile-activated) |

### 4.2 Out of Scope

| Area | Reason |
|---|---|
| Veterinarian management | Separate `vet` module, not part of this scope |
| Online appointment booking | Not present in source code |
| Billing / payments | Not present in source code |
| Authentication / Authorization | No security layer in this module's source |
| REST JSON API for owners/pets | Not present — only HTML views |
| Role-based access control | Not implemented |
| Multi-clinic support | Single-clinic model only |

---

## 5. Business Analysis (BA)

### 5.1 User Stories

| ID | As a... | I want to... | So that... |
|---|---|---|---|
| US-01 | Clinic Receptionist | Register a new pet owner | The owner has a record in the system before adding their pets |
| US-02 | Clinic Receptionist | Search for an owner by last name | I can quickly pull up their record when they arrive |
| US-03 | Clinic Receptionist | View all pets and recent visits for an owner | The veterinarian can review the pet's medical history |
| US-04 | Clinic Receptionist | Edit an owner's contact details | Records stay current when owners move or change phone |
| US-05 | Clinic Receptionist | Add a new pet to an existing owner | New animals get registered at their first visit |
| US-06 | Clinic Receptionist | Edit a pet's details | Corrections can be made after initial entry |
| US-07 | Clinic Receptionist | Record a vet visit for a pet | There is a permanent log of every visit with date and notes |

### 5.2 Use Case Descriptions

#### UC-01: Find an Owner
```
Actor:   Clinic Receptionist
Trigger: Pet owner arrives at front desk
Steps:
  1. Navigate to Find Owner (GET /owners/find)
  2. Enter partial or full last name
  3a. One match → auto-redirect to owner profile
  3b. Multiple matches → display list; receptionist selects one
  3c. No match → inline error; receptionist creates new owner
```

#### UC-02: Register a New Owner
```
Actor:   Clinic Receptionist
Trigger: Owner not found in search
Steps:
  1. Click "Add Owner" (GET /owners/new)
  2. Enter: first name, last name, address, city, telephone
  3a. Validation passes → owner saved → redirect to owner detail
  3b. Validation fails → form re-rendered with inline error messages
```

#### UC-03: Add a Pet to an Owner
```
Actor:   Clinic Receptionist
Trigger: Existing owner brings a new pet for first time
Pre:     Owner exists in the system
Steps:
  1. Open owner detail → click "Add New Pet" (GET /owners/{ownerId}/pets/new)
  2. Enter: name, birth date, select pet type from dropdown
  3a. Valid, unique name → pet saved → redirect to owner detail
  3b. Duplicate name → inline error "already exists"
  3c. Missing required field → inline field error
```

#### UC-04: Record a Vet Visit
```
Actor:   Clinic Receptionist / Veterinarian
Trigger: Pet completes a vet appointment
Pre:     Pet and owner exist
Steps:
  1. Find owner → click "Add Visit" for the relevant pet
  2. Confirm or change visit date (defaults to today)
  3. Enter visit description
  4a. Valid → visit saved → redirect to owner detail
  4b. Empty description → inline error
```

#### UC-05: Edit Owner Details
```
Actor:   Clinic Receptionist
Trigger: Owner's contact information needs updating
Steps:
  1. Open owner detail → click "Edit Owner" (GET /owners/{ownerId}/edit)
  2. Modify any field
  3a. Valid → saved → redirect to updated owner detail
  3b. Invalid → form re-rendered with errors
```

### 5.3 Process Flow Diagram (Textual)

```
[Receptionist opens browser]
        │
        ▼
[GET /owners/find] ──── enter last name ──── [GET /owners]
        │                                          │
        │                                     ┌───┴───┐
        │                               1 result   multiple
        │                                 │            │
        │                          [redirect]   [ownersList]
        │                                 │            │
        └──────────────────────────────[GET /owners/{id}]
                                              │
                           ┌──────────────────┼──────────────────┐
                           │                  │                   │
                    [Edit Owner]        [Add Pet]          [Add Visit]
                           │                  │                   │
               POST /owners/{id}/edit  POST /pets/new   POST /visits/new
                           │                  │                   │
                    [validation]        [validation]        [validation]
                           │                  │                   │
                        [save]            [save]              [save]
                           └──────────────────┴───────────────────┘
                                              │
                                   [redirect: owner detail]
```

---

## 6. Business Requirements (BRD)

### 6.1 Owner Management Requirements

| ID | Requirement | Priority | Acceptance Criterion |
|---|---|---|---|
| BR-01 | System shall allow registration of a new owner with fields: first name, last name, address, city, telephone (max 10 numeric digits) | Must Have | POST /owners/new with valid data creates record and redirects to new owner's detail |
| BR-02 | System shall allow search for owners by last name using prefix match | Must Have | GET /owners?lastName=Franklin returns all owners whose last name starts with "Franklin" |
| BR-03 | System shall navigate directly to the owner's profile when exactly one match is found | Must Have | Single-match search redirects without showing list |
| BR-04 | System shall display a multi-owner list when more than one match is found | Must Have | Multiple-match search renders owner list view |
| BR-05 | System shall display full owner details including all pets and each pet's visit history | Must Have | GET /owners/{id} shows owner fields, pets array, and visits per pet in chronological order |
| BR-06 | System shall allow staff to edit owner information (all fields except ID) | Must Have | POST /owners/{id}/edit updates record; ID is always forced from URL path, never from form |
| BR-07 | All owner fields are mandatory; validation errors must be shown inline without losing entered data | Must Have | Invalid POST re-renders form with field-level error messages |

### 6.2 Pet Management Requirements

| ID | Requirement | Priority | Acceptance Criterion |
|---|---|---|---|
| BR-08 | System shall allow adding a new pet to an existing owner with name, date of birth, and pet type | Must Have | POST /owners/{id}/pets/new with valid data creates pet under correct owner |
| BR-09 | System shall enforce that pet name is unique per owner | Must Have | Duplicate name returns "already exists" error and does not save |
| BR-10 | Pet types must be selected from the predefined catalog | Must Have | Dropdown populated from `PetRepository.findPetTypes()` sorted alphabetically |
| BR-11 | System shall allow editing of pet details (name, birth date, type) | Must Have | POST /owners/{id}/pets/{petId}/edit updates the pet record |
| BR-12 | Pets must always remain associated with their owner | Must Have | Pet's `owner_id` FK is always set via `Owner.addPet()` domain method |

### 6.3 Visit Management Requirements

| ID | Requirement | Priority | Acceptance Criterion |
|---|---|---|---|
| BR-13 | System shall allow recording a new vet visit for any registered pet with a date and description | Must Have | POST /owners/{id}/pets/{petId}/visits/new creates visit record |
| BR-14 | Visit date shall default to the current date but remain editable | Should Have | `Visit.date` initializes to `LocalDate.now()` in entity class |
| BR-15 | Visit description is mandatory; blank descriptions must be rejected | Must Have | Empty description returns `@NotEmpty` validation error |
| BR-16 | All visits must be displayed in chronological order on the owner detail page | Must Have | `Pet.getVisits()` sorts by date ascending |
| BR-17 | Historical visit records are read-only | Must Have | No edit/delete endpoint exists for visits |

---

## 7. Functional Requirements — Detailed

> Extracted strictly from actual Kotlin source code. No inference or assumptions.

| FR-ID | Feature | HTTP Method | Endpoint | Controller Method | Key Logic |
|---|---|---|---|---|---|
| FR-OWN-001 | Render Owner Creation Form | GET | `/owners/new` | `initCreationForm` | Adds empty `Owner()` to model |
| FR-OWN-002 | Create New Owner | POST | `/owners/new` | `processCreationForm` | `@Valid` → check `BindingResult` → `owners.save()` → redirect |
| FR-OWN-003 | Render Owner Search Form | GET | `/owners/find` | `initFindForm` | Adds empty `Owner()` to model for search form |
| FR-OWN-004 | Search Owners by Last Name | GET | `/owners` | `processFindForm` | JPQL prefix LIKE on `lastName` → 3-branch when expression |
| FR-OWN-005 | Display Owner Details | GET | `/owners/{ownerId}` | `showOwner` | `findById` + loop to load visits per pet |
| FR-OWN-006 | Render Owner Edit Form | GET | `/owners/{ownerId}/edit` | `initUpdateOwnerForm` | Loads existing owner into model |
| FR-OWN-007 | Update Owner | POST | `/owners/{ownerId}/edit` | `processUpdateOwnerForm` | Forces `owner.id = ownerId` from path before save |
| FR-PET-001 | Render New Pet Form | GET | `/owners/{ownerId}/pets/new` | `initCreationForm` | `@ModelAttribute` loads owner + pet types |
| FR-PET-002 | Create New Pet | POST | `/owners/{ownerId}/pets/new` | `processCreationForm` | Duplicate name check → `PetValidator` → `pets.save()` |
| FR-PET-003 | Render Pet Edit Form | GET | `/owners/{ownerId}/pets/{petId}/edit` | `initUpdateForm` | Loads pet by ID into model |
| FR-PET-004 | Update Pet | POST | `/owners/{ownerId}/pets/{petId}/edit` | `processUpdateForm` | `owner.addPet(pet)` → `pets.save()` |
| FR-VIS-001 | Render New Visit Form | GET | `/owners/*/pets/{petId}/visits/new` | `initNewVisitForm` | `loadPetWithVisit` resolves pet and creates Visit |
| FR-VIS-002 | Create New Visit | POST | `/owners/{ownerId}/pets/{petId}/visits/new` | `processNewVisitForm` | `@Valid` → `visits.save()` → redirect |

---

## 8. Business Rules

| ID | Rule | Source Location |
|---|---|---|
| BRL-01 | An owner must exist before a pet can be registered | `PetController` — `@ModelAttribute findOwner()` |
| BRL-02 | A pet must exist before a visit can be recorded | `VisitController` — `loadPetWithVisit()` |
| BRL-03 | Pet name must be unique within the same owner's pet list | `PetController.processCreationForm()` — `owner.getPet(name, true)` |
| BRL-04 | Owner telephone must contain only numeric digits, max 10 | `Owner.kt` — `@Digits(fraction=0, integer=10)` |
| BRL-05 | Visit descriptions may not be blank or whitespace-only | `Visit.kt` — `@NotEmpty` |
| BRL-06 | Pet type is required only when creating a new pet | `PetValidator.validate()` — `if (pet.isNew && pet.type == null)` |
| BRL-07 | Owner `id` cannot be submitted via form; it is always assigned from the URL path variable | `OwnerController` — `setDisallowedFields("id")` + explicit `owner.id = ownerId` |
| BRL-08 | Pet owner back-reference is always set during `addPet()` regardless of new/existing | `Owner.addPet()` — `pet.owner = this` (unconditional) |
| BRL-09 | Visit `petId` is set automatically when `pet.addVisit(visit)` is called | `Pet.addVisit()` — `visit.petId = this.id` |
| BRL-10 | Owner last-name search uses prefix matching (LIKE `value%`), not exact match | `OwnerRepository.findByLastName()` — JPQL LIKE query |
| BRL-11 | Owner search is case-insensitive for H2 backend | H2 schema: `VARCHAR_IGNORECASE(30)` on `last_name` column |
| BRL-12 | Pet types are loaded sorted alphabetically for all form dropdowns | `PetRepository.findPetTypes()` — JPQL `ORDER BY ptype.name` |
| BRL-13 | Pets in owner detail view are displayed sorted alphabetically by name | `Owner.getPets()` — `sortedWith(compareBy { it.name })` |
| BRL-14 | Visits in owner detail view are displayed sorted chronologically by date | `Pet.getVisits()` — `sortedWith(compareBy { it.date })` |

---

## 9. High-Level Design (HLD)

### 9.1 System Context Diagram

```
┌───────────────────────────────────────────────────────────┐
│                     Browser (User)                        │
│          GET/POST HTTP requests with form data            │
└─────────────────────────┬─────────────────────────────────┘
                          │ HTTP/HTML
                          ▼
┌───────────────────────────────────────────────────────────┐
│                Spring MVC DispatcherServlet               │
│                   (Embedded Tomcat)                       │
│  ┌──────────────┐  ┌──────────────┐  ┌────────────────┐  │
│  │OwnerController│  │ PetController│  │VisitController │  │
│  └──────┬───────┘  └──────┬───────┘  └───────┬────────┘  │
└─────────┼────────────────┼──────────────────┼────────────┘
          │                │                  │
          ▼                ▼                  ▼
┌───────────────────────────────────────────────────────────┐
│                  Domain Entities + Logic                  │
│   Owner · Pet · PetType · Visit                          │
│   Owner.addPet() · Pet.addVisit() · Owner.getPet()        │
│   PetValidator · PetTypeFormatter                        │
└─────────────────────────┬─────────────────────────────────┘
                          │ Spring Data JPA
                          ▼
┌───────────────────────────────────────────────────────────┐
│                    Repository Layer                       │
│  OwnerRepository · PetRepository · VisitRepository       │
└─────────────────────────┬─────────────────────────────────┘
                          │ JDBC / Hibernate
                          ▼
┌───────────────────────────────────────────────────────────┐
│              Database (H2 default / MySQL)                │
│  owners · pets · types · visits                          │
└───────────────────────────────────────────────────────────┘
```

### 9.2 Module Component Architecture

```
org.springframework.samples.petclinic
│
├── owner/                          (primary module package)
│   ├── [WEB]         OwnerController       — /owners/** routes
│   ├── [WEB]         PetController         — /owners/{id}/pets/** routes
│   ├── [WEB]         VisitController       — .../visits/** routes
│   ├── [DOMAIN]      Owner                 — @Entity, business methods
│   ├── [DOMAIN]      Pet                   — @Entity, business methods
│   ├── [DOMAIN]      PetType               — @Entity (catalog)
│   ├── [REPO]        OwnerRepository       — Spring Data JPA
│   ├── [REPO]        PetRepository         — Spring Data JPA
│   ├── [UTIL]        PetValidator          — Spring Validator
│   └── [UTIL]        PetTypeFormatter      — Spring Formatter<PetType>
│
├── visit/                          (split package for Visit entity)
│   ├── [DOMAIN]      Visit                 — @Entity
│   └── [REPO]        VisitRepository       — Spring Data JPA
│
└── model/                          (shared base classes)
    ├── BaseEntity                   — @Id, isNew
    ├── NamedEntity                  — name column
    └── Person                       — firstName, lastName
```

### 9.3 Request Processing Flow

```
HTTP Request
     │
     ▼
DispatcherServlet
     │
     ├─── @InitBinder runs first (disallows "id" field binding)
     │
     ├─── @ModelAttribute methods run next
     │         ├── PetController: findOwner(ownerId) → owners.findById()
     │         ├── PetController: populatePetTypes() → pets.findPetTypes()
     │         └── VisitController: loadPetWithVisit(petId) → pets.findById()
     │
     ▼
Handler Method (@GetMapping / @PostMapping)
     │
     ├─── [POST paths] @Valid validation triggers
     │         ├── Jakarta Bean Validation (Owner / Visit fields)
     │         └── PetValidator (Pet fields, registered via @InitBinder)
     │
     ├─── Business Logic
     │         ├── BindingResult error check
     │         ├── Duplicate name check (Pet creation)
     │         ├── Domain method calls (addPet, addVisit)
     │         └── Repository save / query
     │
     └─── Return: view name (String) or "redirect:/..."
               │
               ▼
         Thymeleaf renders HTML → HTTP Response
```

### 9.4 Data Flow — Owner Details Page

```
GET /owners/{ownerId}
     │
     ▼
owners.findById(ownerId)
  → JPQL: SELECT owner LEFT JOIN FETCH owner.pets WHERE owner.id = ?
  → Returns: Owner with pets preloaded
     │
     ▼
for each pet in owner.getPets()  [sorted by name]
  → visits.findByPetId(pet.id!!)
  → Spring Data derived query: SELECT * FROM visits WHERE pet_id = ?
  → pet.visits = result  (@Transient field assignment)
     │
     ▼
model.addAttribute(owner)
     │
     ▼
Thymeleaf: templates/owners/ownerDetails.html
```

---

## 10. Data Model

### 10.1 Entity Inheritance Hierarchy

```
Serializable
  └── BaseEntity  (@MappedSuperclass)
        ├── id: Int?           — PK, IDENTITY
        └── isNew: Boolean     — computed
              │
    ┌─────────┴──────────┐
    │                    │
NamedEntity           Person  (@MappedSuperclass)
(@MappedSuperclass)   ├── firstName: String  @NotEmpty
├── name: String?     └── lastName: String   @NotEmpty
│        │                      │
│   PetType @Entity         ┌───┴───┐
│   (table: types)        Owner   Vet
│                        @Entity  @Entity
Pet @Entity
(table: pets)

BaseEntity
  └── Visit @Entity  (table: visits)
```

### 10.2 Entity Field Reference

#### Owner (`owners` table)

| Field | Kotlin Type | Nullable | Default | Constraint | DB Column |
|---|---|---|---|---|---|
| `id` | `Int?` | Yes | `null` | PK IDENTITY | `id` |
| `firstName` | `String` | No | `""` | `@NotEmpty` | `first_name VARCHAR(30)` |
| `lastName` | `String` | No | `""` | `@NotEmpty` | `last_name VARCHAR_IGNORECASE(30)` |
| `address` | `String` | No | `""` | `@NotEmpty` | `address VARCHAR(255)` |
| `city` | `String` | No | `""` | `@NotEmpty` | `city VARCHAR(80)` |
| `telephone` | `String` | No | `""` | `@NotEmpty, @Digits(0,10)` | `telephone VARCHAR(20)` |
| `pets` | `MutableSet<Pet>` | No | `HashSet()` | `@OneToMany(cascade=ALL)` | — (FK via `owner_id`) |

#### Pet (`pets` table)

| Field | Kotlin Type | Nullable | Default | Constraint | DB Column |
|---|---|---|---|---|---|
| `id` | `Int?` | Yes | `null` | PK IDENTITY | `id` |
| `name` | `String?` | Yes | `null` | PetValidator: required | `name VARCHAR(30)` |
| `birthDate` | `LocalDate?` | Yes | `null` | PetValidator: required | `birth_date DATE` |
| `type` | `PetType?` | Yes | `null` | PetValidator: required if new | `type_id INTEGER FK → types.id` |
| `owner` | `Owner?` | Yes | `null` | `@ManyToOne` | `owner_id INTEGER FK → owners.id` |
| `visits` | `MutableSet<Visit>` | No | `LinkedHashSet()` | `@Transient` — not persisted | — |

#### PetType (`types` table)

| Field | Kotlin Type | Nullable | Default | DB Column |
|---|---|---|---|---|
| `id` | `Int?` | Yes | `null` | `id` PK IDENTITY |
| `name` | `String?` | Yes | `null` | `name VARCHAR(80)` |

**Seeded values:** `bird`, `cat`, `dog`, `hamster`, `lizard`, `snake`

#### Visit (`visits` table)

| Field | Kotlin Type | Nullable | Default | Constraint | DB Column |
|---|---|---|---|---|---|
| `id` | `Int?` | Yes | `null` | PK IDENTITY | `id` |
| `date` | `LocalDate` | No | `LocalDate.now()` | Date format `yyyy-MM-dd` | `visit_date DATE` |
| `description` | `String?` | Yes | `null` | `@NotEmpty` | `description VARCHAR(255)` |
| `petId` | `Int?` | Yes | `null` | Plain FK (not JPA relation) | `pet_id INTEGER FK → pets.id` |

### 10.3 Entity Relationships

```
Owner  ──1──────────────< Pet  (OneToMany, cascade=ALL, mappedBy="owner")
                          Pet  >────────────1── PetType  (ManyToOne)
                          Pet  ──1──────────────< Visit  (@Transient, petId FK)
```

> `Pet.visits` is a `@Transient` field — it is populated manually by the controller, not via a JPA association. `Visit.petId` is a plain `Int?` column (not a `@ManyToOne`).

---

## 11. Database Design

### 11.1 Schema Tables

| Table | Primary Key | Notable Columns | Indexes |
|---|---|---|---|
| `owners` | `id` IDENTITY | `last_name` is `VARCHAR_IGNORECASE` (H2) | `owners_last_name` on `last_name` |
| `pets` | `id` IDENTITY | `type_id` FK, `owner_id` FK | `pets_name` on `name` |
| `types` | `id` IDENTITY | `name` | `types_name` on `name` |
| `visits` | `id` IDENTITY | `pet_id` FK, `visit_date` | `visits_pet_id` on `pet_id` |

### 11.2 Foreign Key Constraints

| Constraint Name | Child Table | Child Column | Parent Table | Parent Column |
|---|---|---|---|---|
| `fk_pets_owners` | `pets` | `owner_id` | `owners` | `id` |
| `fk_pets_types` | `pets` | `type_id` | `types` | `id` |
| `fk_visits_pets` | `visits` | `pet_id` | `pets` | `id` |

### 11.3 Database Profiles

| Profile | Database | Activation | Notes |
|---|---|---|---|
| Default | H2 in-memory | (no profile needed) | Schema and data from `db/h2/schema.sql` + `db/h2/data.sql`; case-insensitive `last_name` |
| `mysql` | MySQL | `--spring.profiles.active=mysql` | `spring.datasource.url=jdbc:mysql://localhost/petclinic`; credentials: `petclinic/petclinic` |

### 11.4 DDL Configuration

```properties
spring.jpa.hibernate.ddl-auto=none         # No auto DDL; SQL scripts manage schema
spring.sql.init.schema-locations=classpath*:db/${database}/schema.sql
spring.sql.init.data-locations=classpath*:db/${database}/data.sql
```

---

## 12. API Design

### 12.1 Full Endpoint Inventory — Owner / Pet / Visit Module

| Method | Endpoint | Controller | Handler Method | Response |
|---|---|---|---|---|
| GET | `/owners/new` | `OwnerController` | `initCreationForm` | View: `owners/createOrUpdateOwnerForm` |
| POST | `/owners/new` | `OwnerController` | `processCreationForm` | Redirect: `/owners/{id}` or form with errors |
| GET | `/owners/find` | `OwnerController` | `initFindForm` | View: `owners/findOwners` |
| GET | `/owners` | `OwnerController` | `processFindForm` | Redirect (1), View: list (many), View: form (0) |
| GET | `/owners/{ownerId}` | `OwnerController` | `showOwner` | View: `owners/ownerDetails` |
| GET | `/owners/{ownerId}/edit` | `OwnerController` | `initUpdateOwnerForm` | View: `owners/createOrUpdateOwnerForm` |
| POST | `/owners/{ownerId}/edit` | `OwnerController` | `processUpdateOwnerForm` | Redirect: `/owners/{ownerId}` or form with errors |
| GET | `/owners/{ownerId}/pets/new` | `PetController` | `initCreationForm` | View: `pets/createOrUpdatePetForm` |
| POST | `/owners/{ownerId}/pets/new` | `PetController` | `processCreationForm` | Redirect: `/owners/{ownerId}` or form with errors |
| GET | `/owners/{ownerId}/pets/{petId}/edit` | `PetController` | `initUpdateForm` | View: `pets/createOrUpdatePetForm` |
| POST | `/owners/{ownerId}/pets/{petId}/edit` | `PetController` | `processUpdateForm` | Redirect: `/owners/{ownerId}` or form with errors |
| GET | `/owners/*/pets/{petId}/visits/new` | `VisitController` | `initNewVisitForm` | View: `pets/createOrUpdateVisitForm` |
| POST | `/owners/{ownerId}/pets/{petId}/visits/new` | `VisitController` | `processNewVisitForm` | Redirect: `/owners/{ownerId}` or form with errors |

### 12.2 Form Fields Per Endpoint (POST)

#### POST `/owners/new` and `POST /owners/{ownerId}/edit`

| Field | Type | Required | Validation |
|---|---|---|---|
| `firstName` | String | Yes | `@NotEmpty` |
| `lastName` | String | Yes | `@NotEmpty` |
| `address` | String | Yes | `@NotEmpty` |
| `city` | String | Yes | `@NotEmpty` |
| `telephone` | String | Yes | `@NotEmpty`, `@Digits(fraction=0, integer=10)` |

#### POST `/owners/{ownerId}/pets/new` and `POST /owners/{ownerId}/pets/{petId}/edit`

| Field | Type | Required | Validation |
|---|---|---|---|
| `name` | String | Yes | `PetValidator` — `StringUtils.hasLength` |
| `birthDate` | LocalDate | Yes | `PetValidator` — non-null; format `yyyy-MM-dd` |
| `type` | PetType | Yes (new only) | `PetValidator` — non-null if `pet.isNew` |

#### POST `/owners/{ownerId}/pets/{petId}/visits/new`

| Field | Type | Required | Validation |
|---|---|---|---|
| `date` | LocalDate | Yes (default: today) | Format `yyyy-MM-dd` |
| `description` | String | Yes | `@NotEmpty` |

### 12.3 Repository Query Reference

| Repository | Method | Query Type | Query Text |
|---|---|---|---|
| `OwnerRepository` | `findByLastName(lastName)` | JPQL custom | `SELECT DISTINCT owner FROM Owner owner left join fetch owner.pets WHERE owner.lastName LIKE :lastName%` |
| `OwnerRepository` | `findById(id)` | JPQL custom | `SELECT owner FROM Owner owner left join fetch owner.pets WHERE owner.id =:id` |
| `OwnerRepository` | `save(owner)` | Spring Data default | INSERT / UPDATE |
| `PetRepository` | `findPetTypes()` | JPQL custom | `SELECT ptype FROM PetType ptype ORDER BY ptype.name` |
| `PetRepository` | `findById(id)` | Spring Data default | SELECT by PK |
| `PetRepository` | `save(pet)` | Spring Data default | INSERT / UPDATE |
| `VisitRepository` | `findByPetId(petId)` | Method name derived | `SELECT * FROM visits WHERE pet_id = ?` |
| `VisitRepository` | `save(visit)` | Spring Data default | INSERT |

---

## 13. Technical Architecture

### 13.1 Layered Architecture

```
┌──────────────────────────────────────────────────────┐
│                  PRESENTATION LAYER                   │
│  Controllers (@Controller) + Thymeleaf Templates      │
│  OwnerController · PetController · VisitController    │
├──────────────────────────────────────────────────────┤
│             BUSINESS LOGIC / DOMAIN LAYER             │
│  Entities: Owner · Pet · PetType · Visit              │
│  Validator: PetValidator                              │
│  Formatter: PetTypeFormatter                         │
│  Model Helpers: BaseEntity · NamedEntity · Person     │
├──────────────────────────────────────────────────────┤
│                DATA ACCESS LAYER                      │
│  OwnerRepository · PetRepository · VisitRepository   │
│  (Spring Data JPA — Repository<T,ID> interface)       │
├──────────────────────────────────────────────────────┤
│                  DATABASE LAYER                       │
│      H2 (in-memory)  ·  MySQL (profile)              │
└──────────────────────────────────────────────────────┘
```

> **No explicit service layer exists.** Business logic is distributed between controllers, domain entities (`Owner.addPet()`, `Pet.addVisit()`), and validators.

### 13.2 Key Architectural Decisions

| Decision | Detail | Rationale |
|---|---|---|
| `proxyBeanMethods = false` | On `@SpringBootApplication` | Faster startup, no CGLIB proxy overhead |
| Constructor injection | All controllers use primary constructor | Idiomatic Kotlin; enables `val` dependencies |
| `-Xjsr305=strict` | Kotlin compiler flag | Bridges Spring/Jakarta `@Nullable`/`@NonNull` to Kotlin null-safety at compile time |
| `spring.data.jpa.repositories.bootstrap-mode=deferred` | JPA repo bootstrap in parallel thread | Reduces startup latency |
| `spring.jpa.open-in-view=false` | OSIV anti-pattern disabled | Forces explicit data loading in controller; no lazy-load in view |
| `@Transient` visits on Pet | Visits loaded manually, not via JPA association | Avoids N+1 queries; keeps Visit entity decoupled from Pet |
| `Visit.petId` as `Int?` | Plain integer FK, not `@ManyToOne` | Keeps Visit independent from Pet entity graph |
| `Repository<T, ID>` base | Minimal Spring Data interface | Only explicitly declared methods are available — no accidental `deleteAll` exposure |

---

## 14. Technology Stack

| Category | Technology | Version | Role in this Module |
|---|---|---|---|
| Language | Kotlin | 2.3.20 | All source code |
| Framework | Spring Boot | 4.0.4 | Application bootstrap, auto-configuration |
| Web Layer | Spring MVC | Managed | `@Controller`, `@GetMapping`, `@PostMapping`, `BindingResult` |
| Templating | Thymeleaf | Managed | HTML views for all 13 endpoints |
| ORM | Spring Data JPA + Hibernate | Managed | `Repository<T, ID>`, JPQL queries, `@Entity` mappings |
| Validation | Jakarta Bean Validation (Hibernate Validator) | Managed | `@NotEmpty`, `@Digits` on Owner/Visit entities |
| Default Database | H2 | Managed | In-memory; schema via SQL scripts |
| Production Database | MySQL | Managed | Profile-activated (`--spring.profiles.active=mysql`) |
| Build Tool | Gradle (Kotlin DSL) | 8+ | `build.gradle.kts`, Kotlin DSL |
| JVM Target | Java | 17 | Compiled by toolchain |
| UI Framework | Bootstrap | 5.3.8 | Styling for all Thymeleaf views (WebJar) |
| Icons | Font Awesome | 4.7.0 | UI icons (WebJar) |
| Containerization | Docker + Google Jib | 3.5.3 | Deployment |
| Observability | Spring Boot Actuator | Managed | Health/metrics endpoints |
| Dev Tooling | Spring Boot DevTools | Managed | Live reload during development |
| Testing | JUnit 5 + Mockito + MockMvc | Managed | All test categories |

---

## 15. Component Design

### 15.1 Controllers

| Controller | Package | Base Path | Dependencies | Template Base |
|---|---|---|---|---|
| `OwnerController` | `owner` | `/owners` | `OwnerRepository`, `VisitRepository` | `owners/` |
| `PetController` | `owner` | `/owners/{ownerId}` | `PetRepository`, `OwnerRepository` | `pets/` |
| `VisitController` | `owner` | `/owners/{ownerId}/pets/{petId}` | `VisitRepository`, `PetRepository` | `pets/` |

### 15.2 Repositories

| Repository | Entity | Base Interface | Explicit Methods |
|---|---|---|---|
| `OwnerRepository` | `Owner` | `Repository<Owner, Int>` | `findByLastName`, `findById`, `save` |
| `PetRepository` | `Pet` | `Repository<Pet, Int>` | `findPetTypes`, `findById`, `save` |
| `VisitRepository` | `Visit` | `Repository<Visit, Int>` | `save`, `findByPetId` |

### 15.3 Domain Entity Methods

| Class | Method | Signature | Purpose |
|---|---|---|---|
| `Owner` | `getPets()` | `(): List<Pet>` | Returns pets sorted alphabetically by name |
| `Owner` | `addPet()` | `(pet: Pet)` | Adds new pet to set; always sets `pet.owner = this` |
| `Owner` | `getPet()` | `(name: String, ignoreNew: Boolean): Pet?` | Finds pet by name; optionally ignores unsaved pets |
| `Pet` | `getVisits()` | `(): List<Visit>` | Returns visits sorted chronologically by date |
| `Pet` | `addVisit()` | `(visit: Visit)` | Adds visit; sets `visit.petId = this.id` |
| `BaseEntity` | `isNew` | `Boolean` (computed) | Returns `true` when `id == null` |

### 15.4 Utilities

| Class | Type | Package | Purpose |
|---|---|---|---|
| `PetValidator` | `Validator` | `owner` | Custom Spring `Validator` — validates pet name, type, birthDate |
| `PetTypeFormatter` | `Formatter<PetType>` | `owner` | Converts between `PetType` entity and its string name in form binding |

---

## 16. Validation & Error Handling

### 16.1 Validation Summary

| Entity | Mechanism | Fields | Rules |
|---|---|---|---|
| `Owner` / `Person` | Jakarta Bean Validation (`@Valid`) | `firstName`, `lastName`, `address`, `city`, `telephone` | All `@NotEmpty`; telephone `@Digits(fraction=0, integer=10)` |
| `Pet` | `PetValidator` (custom Spring `Validator`) | `name`, `type`, `birthDate` | name: `hasLength`; type: non-null if `isNew`; birthDate: non-null |
| `Pet` (duplicate) | Controller conditional check | `name` | `owner.getPet(name, ignoreNew=true) != null` → `rejectValue("name", "duplicate")` |
| `Visit` | Jakarta Bean Validation (`@Valid`) | `description` | `@NotEmpty` |

### 16.2 Security: Mass-Assignment Prevention

All three controllers declare:
```kotlin
@InitBinder
fun setAllowedFields(dataBinder: WebDataBinder) {
    dataBinder.setDisallowedFields("id")
}
```
This prevents the HTTP form from overwriting the entity's primary key.

### 16.3 Error Handling

| Scenario | Mechanism | Behavior |
|---|---|---|
| Form validation failure | `BindingResult.hasErrors()` check | Form re-rendered with error messages; entered data preserved |
| Owner not found via `findById` | No explicit handler in controller | Exception propagates to Spring default error handler (`/error`) |
| Unknown PetType name in formatter | `ParseException` thrown in `PetTypeFormatter.parse()` | Propagates; PetType dropdown prevents invalid input under normal use |
| Non-null assertion `pet.id!!` fails | `KotlinNullPointerException` | Uncaught; propagates to Spring error handler |
| No `try-catch` blocks | None present in this module | Spring Boot's default error page handles uncaught exceptions |

---

## 17. Caching Strategy

> Caching in this module scope: **Not applicable to Owner / Pet / Visit entities.**

The caching configuration (`CacheConfig.kt`) only creates the `vets` JCache region for `VetRepository.findAll()`. No caching is applied to `OwnerRepository`, `PetRepository`, or `VisitRepository` methods.

| Repository | Method | Cached | Cache Region |
|---|---|---|---|
| `VetRepository` | `findAll()` | ✅ Yes | `"vets"` |
| `OwnerRepository` | All methods | ❌ No | — |
| `PetRepository` | All methods | ❌ No | — |
| `VisitRepository` | All methods | ❌ No | — |

---

## 18. Testing Strategy

### 18.1 Test Categories and Coverage

| Test Class | Type | Annotation | Scope | What is Tested |
|---|---|---|---|---|
| `OwnerControllerTest` | Controller | `@WebMvcTest` | Web layer only | All 7 owner endpoints — request/response, model attributes, view names, redirect |
| `PetControllerTest` | Controller | `@WebMvcTest` | Web layer only | All 4 pet endpoints — form rendering, validation errors, redirect |
| `VisitControllerTest` | Controller | `@WebMvcTest` | Web layer only | Visit form rendering, visit save, redirect |
| `OwnerRepositoryTest` | Repository | `@DataJpaTest` | JPA layer only | `findByLastName`, `findById`, `save` |
| `PetRepositoryTest` | Repository | `@DataJpaTest` | JPA layer only | `findPetTypes`, `findById`, `save` |
| `VisitRepositoryTest` | Repository | `@DataJpaTest` | JPA layer only | `findByPetId`, `save` |
| `PetTypeFormatterTest` | Unit | Plain JUnit 5 | No Spring context | `parse()` and `print()` logic |
| `PetclinicIntegrationTests` | Integration | `@SpringBootTest` | Full Spring context | End-to-end cache hit (covers vet module) |

### 18.2 Controller Test Pattern (MockMvc)

```kotlin
// @WebMvcTest loads only web layer; repositories are mocked
@MockitoBean private lateinit var owners: OwnerRepository
@MockitoBean private lateinit var visits: VisitRepository

// Example: Test creation form rendering
mockMvc.perform(get("/owners/new"))
    .andExpect(status().isOk)
    .andExpect(model().attributeExists("owner"))
    .andExpect(view().name("owners/createOrUpdateOwnerForm"))
```

### 18.3 Test Configuration

```properties
# src/test/resources/application-test.properties
spring.web.error.include-message=always   # Enables error message assertions in tests
```

### 18.4 Load Testing

Apache JMeter test plan located at `src/test/jmeter/petclinic_test_plan.jmx` — covers key read and write operations.

---

## 19. Deployment Architecture

### 19.1 Local Development

```bash
# Default — H2 in-memory
./gradlew bootRun

# MySQL profile
./gradlew bootRun --args='--spring.profiles.active=mysql'

# Build JAR
./gradlew build
java -jar build/libs/spring-petclinic-kotlin-4.0.2.jar
```

**Application URL:** `http://localhost:8080`  
**H2 Console (dev):** `http://localhost:8080/h2-console`

### 19.2 Docker Deployment

**Multi-stage Dockerfile:**
```
Stage 1 (build):   gradle:4.7.0-jdk8-alpine  → Performs Gradle build
Stage 2 (runtime): openjdk:8-jre-slim         → Runs the JAR on port 8080
```

```bash
docker build -t spring-petclinic-kotlin .
docker run -p 8080:8080 spring-petclinic-kotlin
```

**Google Jib (no Docker daemon required):**
```bash
./gradlew jibDockerBuild        # build local image
./gradlew jib                   # push to registry
```
Produces: `springcommunity/spring-petclinic-kotlin:4.0.2` and `:latest`

### 19.3 MySQL Deployment (Docker Compose)

```bash
docker-compose up -d            # starts MySQL container
./gradlew bootRun --args='--spring.profiles.active=mysql'
```

MySQL container: image `mysql:5.7`, database `petclinic`, user `petclinic`, password `petclinic`.

### 19.4 Actuator Endpoints

All endpoints exposed via `management.endpoints.web.exposure.include=*`:

| Endpoint | URL |
|---|---|
| Health | `/actuator/health` |
| Metrics | `/actuator/metrics` |
| Caches | `/actuator/caches` |
| Request Mappings | `/actuator/mappings` |
| Environment | `/actuator/env` |

---

## 20. Key Design Decisions

| # | Decision | Detail | Impact |
|---|---|---|---|
| 1 | No explicit service layer | Business logic in entities, controllers, validators | Low complexity acceptable for this scope; would need extraction for production-scale patterns |
| 2 | `Visit.petId` as plain `Int?` | Not a JPA `@ManyToOne` | Visit is decoupled from Pet entity; `pet.addVisit()` manually sets the FK |
| 3 | `Pet.visits` as `@Transient` | Loaded per-pet in `showOwner()` loop | Avoids full JPA cascade on Pet; visits loaded only when needed for the detail view |
| 4 | `Repository<T, ID>` minimal interface | No `JpaRepository` or `CrudRepository` | Only explicitly declared methods are callable — no accidental `deleteAll` exposure |
| 5 | `setDisallowedFields("id")` on all controllers | Mass-assignment protection | Primary key cannot be overwritten via form submission |
| 6 | `owner.id = ownerId` forced in update handler | Path variable always wins over form body | Ensures correct record is targeted on update |
| 7 | JPQL `LEFT JOIN FETCH owner.pets` | Eager pet load in `findById` and `findByLastName` | Prevents N+1 on owner→pet relationship when displaying owner lists |
| 8 | H2 `VARCHAR_IGNORECASE` on `last_name` | Case-insensitive search at DB level | Works transparently with the JPQL LIKE query; MySQL requires separate handling |
| 9 | `PetValidator` over Bean Validation annotations | Custom Spring `Validator` | Enables conditional validation (`type` only required for new pets) that can't be expressed purely via annotations |
| 10 | `proxyBeanMethods = false` | On all `@Configuration` and `@SpringBootApplication` | Faster startup; avoids CGLIB proxy overhead |

---

## 21. Constraints & Assumptions

### Constraints

| Constraint | Detail |
|---|---|
| Java 17 minimum | Gradle Java toolchain requires JDK 17+ |
| H2 default | Data does not persist across restarts without MySQL profile |
| No authentication | This module has no security layer; assumes trusted intranet use |
| No service layer | Business logic ownership is split between controllers and domain objects |
| Pet type catalog is static | Changing pet types (add/remove) requires a database change + redeployment |
| Single time zone | Visit dates use `LocalDate.now()` with no time zone configuration |

### Assumptions

| Assumption | Detail |
|---|---|
| Trusted network | No JWT, OAuth2, or session management present in source code |
| Single-tenant | One clinic per deployment; no multi-tenancy |
| Browser access | UI is web-based; no mobile-native client |
| Staff operated | No self-service pet owner portal |

---

## 22. Acceptance Criteria

| ID | Criterion | FR-ID | Test Coverage |
|---|---|---|---|
| AC-01 | A receptionist can find an existing owner by partial last name in ≤ 3 steps | FR-OWN-003, FR-OWN-004 | `OwnerControllerTest` |
| AC-02 | Creating an owner with a missing required field shows inline error, does not navigate away | FR-OWN-002 | `OwnerControllerTest.testProcessCreationFormHasErrors` |
| AC-03 | Adding a duplicate pet name to the same owner shows "already exists" error and does not save | FR-PET-002 | `PetControllerTest` |
| AC-04 | A newly recorded visit appears on the owner's detail page under the correct pet, sorted by date | FR-VIS-002, FR-OWN-005 | `VisitControllerTest`, `OwnerControllerTest` |
| AC-05 | `GET /owners/{id}` for a non-existent ID does not return a 200 response | FR-OWN-005 | `OwnerControllerTest` |
| AC-06 | Updating an owner via `POST /owners/{id}/edit` cannot change the owner's `id` | FR-OWN-007 | BRL-07; `OwnerControllerTest` |
| AC-07 | Pet type dropdown is populated alphabetically | FR-PET-001 | `PetControllerTest` |
| AC-08 | A visit with an empty description is rejected with a validation error | FR-VIS-002 | `VisitControllerTest` |
| AC-09 | All 10 pre-seeded owners, 13 pets, and visit records are present on first H2 run | General | `PetclinicIntegrationTests` |
| AC-10 | All unit, controller, and repository tests pass via `./gradlew test` | All FR | All test classes |

---

## 23. Open Questions & Decisions Required

> Sourced from modernization brief analysis. These must be resolved before any production migration.

### Blocking Decisions

| ID | Decision | Recommendation |
|---|---|---|
| DEC-KT-ARCH-001 | Target service model and runtime architecture for migration | Define stable API/service boundaries and confirm target runtime before starting implementation |
| DEC-KT-DB-001 | Database contract strategy during migration | Preserve existing contracts initially; modernize data access behind a compatibility layer |
| DEC-KT-SQL-001 | SQL parameterization and query safety | Parameterize all SQL statements; validate against injection risks before target implementation |
| DEC-KT-AUTH-001 | Authentication/authorization migration approach | Model current auth/session behavior first, then define clean target design |
| DEC-IAM-001 | Identity/access model (roles, multi-user assumptions, credential handling) | Define target role model and credential policy before implementation |

### Open Questions

| ID | Question | Owner |
|---|---|---|
| Q-001 | Are server-rendered HTML views required, or can all endpoints return JSON while preserving paths? | Client |
| Q-002 | What are the exact HTTP methods and content types for all 37 routes in the full application? | Client |
| Q-003 | Provide the definitive DB schema for Owner, Pet, PetType, Visit (DDL, indexes, FKs) for production | Client |
| Q-004 | What JWT issuers/audiences/scopes are valid, and is OAuth2 token introspection required? | Client |
| Q-005 | Should CrashController and WelcomeController be retained verbatim or deprecated? | Client |

### Quality Gate Failures (from Modernization Brief)

| Gate | Status | Issue |
|---|---|---|
| `bdd_flow_grounding` | ❌ FAIL | BDD scenarios not linked to extracted golden flows |
| `key_safety_issues_identified` | ❌ FAIL | No explicit SQL/credential safety signals detected in extracted artifacts |
| `compliance_constraints_applied` | ⚠️ WARN | Regulatory/software controls not linked to requirements |
| `identity_access_model` | ⚠️ WARN | Role model or credential handling requires confirmation |
| `database_archaeology_ready` | ⚠️ WARN | DB QA detected blocking issues in schema reconstruction |

---

## 24. Glossary

| Term | Definition |
|---|---|
| **Owner** | A person registered with the clinic who is responsible for one or more pets |
| **Pet** | An animal registered under an owner that receives veterinary care |
| **PetType** | A classification category for a pet (e.g., cat, dog, hamster) — predefined catalog |
| **Visit** | A recorded interaction between a pet and the clinic, identified by date and description |
| **CRUD** | Create, Read, Update, Delete — the four standard database operations |
| **BRD** | Business Requirements Document — formal description of business needs a system must fulfill |
| **BA** | Business Analysis — the practice of identifying business needs and determining solutions |
| **HLD** | High-Level Design — architectural overview of the system describing major components and their interactions |
| **FR** | Functional Requirement — a specific behavior the system must perform |
| **BRL** | Business Rule — an explicit constraint or condition governing business logic |
| **NFR** | Non-Functional Requirement — quality attribute (performance, security, scalability) |
| **Spring MVC** | Spring's web framework — `@Controller`, `@GetMapping`, `BindingResult`, form binding |
| **Thymeleaf** | Server-side HTML templating engine used to generate web UI |
| **JPA** | Jakarta Persistence API — ORM standard for database interaction |
| **Spring Data** | Spring module providing repository abstractions over JPA, JDBC, etc. |
| **JPQL** | Java Persistence Query Language — SQL-like query language operating on JPA entities |
| **H2** | Embedded in-memory relational database for development/testing |
| **@Transient** | JPA annotation marking a field as not mapped to any database column |
| **@MappedSuperclass** | JPA annotation for base classes whose fields are mapped in subclass tables |
| **isNew** | Computed property on `BaseEntity` — `true` when `id == null` (entity not yet persisted) |
| **PetValidator** | Custom Spring `Validator` implementing programmatic validation for `Pet` fields |
| **PetTypeFormatter** | Spring `Formatter<PetType>` converting between form string values and `PetType` entities |
| **Mass Assignment** | Security vulnerability where HTTP form data overwrites unintended entity fields (prevented by `setDisallowedFields`) |
| **Bootstrap Mode Deferred** | Spring Data JPA config that initializes repositories in parallel with rest of app startup |
| **OSIV** | Open Session In View — anti-pattern of keeping JPA session open for the full request (disabled here) |

---

*End of Comprehensive Summary Document — Spring Petclinic Kotlin · Owner / Pet / Visit Module*

| Document | Filename |
|---|---|
| This summary | `SUMMARY_DOCUMENT.md` |
| Business BRD (application-wide) | `BRD.md` |
| Reverse-engineered BRD (source-exact) | `BRD_OWNER_PET_VISIT.md` |
| Technical Workbook | `TECH_WORKBOOK.md` |
