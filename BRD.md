# Business Requirements Document (BRD)
## Spring Petclinic — Kotlin Edition

| Field | Value |
|---|---|
| **Document Version** | 1.0 |
| **Date** | April 10, 2026 |
| **Status** | Draft |
| **Application** | Spring Petclinic Kotlin |
| **Technology Stack** | Kotlin 2.3.20 · Spring Boot 4.0.4 · Spring MVC · Thymeleaf · JPA / H2 / MySQL |

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Business Objectives](#2-business-objectives)
3. [Project Scope](#3-project-scope)
4. [Stakeholders](#4-stakeholders)
5. [Business Requirements](#5-business-requirements)
   - 5.1 [Owner Management](#51-owner-management)
   - 5.2 [Pet Management](#52-pet-management)
   - 5.3 [Visit Management](#53-visit-management)
   - 5.4 [Veterinarian Directory](#54-veterinarian-directory)
   - 5.5 [System & Non-Functional Requirements](#55-system--non-functional-requirements)
6. [Use Cases](#6-use-cases)
7. [Data Requirements](#7-data-requirements)
8. [Business Rules](#8-business-rules)
9. [Constraints and Assumptions](#9-constraints-and-assumptions)
10. [Acceptance Criteria](#10-acceptance-criteria)
11. [Glossary](#11-glossary)

---

## 1. Executive Summary

The **Spring Petclinic Kotlin** application is a veterinary clinic management system designed to digitize the day-to-day administrative operations of a small-to-medium pet clinic. The system enables clinic staff to maintain records for pet owners, their animals, scheduled veterinary visits, and the clinic's veterinary professionals.

This document defines the business requirements that the system must satisfy. It serves as the authoritative reference for development, testing, and acceptance of the solution.

---

## 2. Business Objectives

| ID | Objective |
|----|-----------|
| BO-01 | Eliminate paper-based record keeping by providing a centralized digital repository for all clinic data. |
| BO-02 | Enable staff to quickly locate any owner or pet by name, reducing front-desk lookup time. |
| BO-03 | Provide a complete, chronological visit history for every pet so veterinarians can make informed clinical decisions. |
| BO-04 | Maintain an up-to-date directory of veterinarians and their medical specialties for patient scheduling and referral purposes. |
| BO-05 | Deliver a web-based interface accessible from any modern browser without requiring software installation on client machines. |
| BO-06 | Serve as a reference implementation demonstrating Spring Boot best practices on the JVM using Kotlin. |

---

## 3. Project Scope

### 3.1 In Scope

- Registration, search, and update of **pet owners**
- Registration, update, and type classification of **pets** owned by registered owners
- Recording and display of **veterinary visit** history per pet
- Read-only **veterinarian directory** with specialty information
- Web UI served by Spring MVC with Thymeleaf templates
- REST/JSON endpoint for the veterinarian list (`GET /vets`)
- In-memory H2 database for development; MySQL support for production deployment
- Response caching for the veterinarian list
- Error handling page for unhandled server exceptions

### 3.2 Out of Scope

- Online appointment booking or scheduling workflows
- Billing, invoicing, or payment processing
- Medical records beyond descriptive visit notes
- Inventory or pharmacy management
- Role-based access control or authentication/authorization
- Mobile native applications
- Multi-clinic / multi-tenant support

---

## 4. Stakeholders

| Role | Responsibilities |
|------|-----------------|
| **Clinic Receptionist** | Primary end user; registers owners, adds pets, records visits. |
| **Veterinarian** | Reviews pet visit history during consultations. |
| **Clinic Manager** | Maintains vet directory and oversees business operations. |
| **System Administrator** | Deploys and configures the application; manages database. |
| **Development Team** | Implements, tests, and maintains the codebase. |

---

## 5. Business Requirements

### 5.1 Owner Management

| ID | Requirement | Priority |
|----|-------------|----------|
| BR-01 | The system shall allow staff to **register a new owner** with the following required fields: first name, last name, address, city, and telephone number (up to 10 digits). | Must Have |
| BR-02 | The system shall allow staff to **search for owners by last name** using a partial or full name match. | Must Have |
| BR-03 | When exactly one owner matches a search, the system shall navigate directly to that owner's detail page. | Must Have |
| BR-04 | When multiple owners match a search, the system shall display a list of matching owners. | Must Have |
| BR-05 | The system shall allow staff to **view full owner details** including all registered pets and the visit history for each pet. | Must Have |
| BR-06 | The system shall allow staff to **edit owner information** (all fields except system-generated ID). | Must Have |
| BR-07 | All owner fields are mandatory; the system shall validate input and surface descriptive error messages without losing entered data. | Must Have |

### 5.2 Pet Management

| ID | Requirement | Priority |
|----|-------------|----------|
| BR-08 | The system shall allow staff to **add a new pet** to an existing owner record. Each pet requires a name, date of birth, and pet type. | Must Have |
| BR-09 | The system shall enforce that a **pet name is unique per owner** (duplicate names not allowed for the same owner). | Must Have |
| BR-10 | Pet types shall be selected from a **predefined catalog** (cat, dog, lizard, snake, bird, hamster). | Must Have |
| BR-11 | The system shall allow staff to **edit pet details** (name, birth date, type). | Must Have |
| BR-12 | Pets shall always remain **associated with their owner** and displayed within the owner's detail view. | Must Have |

### 5.3 Visit Management

| ID | Requirement | Priority |
|----|-------------|----------|
| BR-13 | The system shall allow staff to **record a new veterinary visit** for any registered pet. Each visit requires a date and a description. | Must Have |
| BR-14 | Visit date shall **default to the current date** but be editable. | Should Have |
| BR-15 | Visit description is a **mandatory free-text field**; empty descriptions shall be rejected. | Must Have |
| BR-16 | The system shall display **all visits in chronological order** on the pet owner's detail page. | Must Have |
| BR-17 | Historical visit records shall be **read-only** (no editing or deletion of past visits). | Must Have |

### 5.4 Veterinarian Directory

| ID | Requirement | Priority |
|----|-------------|----------|
| BR-18 | The system shall display a **directory of all veterinarians** with their first name, last name, and list of specialties. | Must Have |
| BR-19 | Specialties shall be sourced from a **predefined catalog** (radiology, surgery, dentistry). | Must Have |
| BR-20 | A veterinarian with **no assigned specialties** shall display "none" in the specialties column. | Must Have |
| BR-21 | The veterinarian list shall be accessible as a **JSON REST endpoint** (`GET /vets`) in addition to the HTML view. | Should Have |
| BR-22 | The veterinarian list shall be **cached** to minimize repeated database reads; the cache shall be invalidated on restart. | Should Have |

### 5.5 System & Non-Functional Requirements

| ID | Requirement | Priority |
|----|-------------|----------|
| NFR-01 | The application shall be deployable as a **self-contained JAR** using Spring Boot's embedded Tomcat server. | Must Have |
| NFR-02 | The application shall support **Docker-based deployment** via a provided Dockerfile and Google Jib plugin. | Should Have |
| NFR-03 | All web pages shall be **responsive** and compatible with modern desktop browsers (Chrome, Firefox, Edge, Safari). | Must Have |
| NFR-04 | The system shall use **form validation** on both client-facing forms and server-side, providing inline error feedback without full page reloads where possible. | Must Have |
| NFR-05 | **Unhandled exceptions** shall result in a user-friendly 5xx error page rather than raw stack traces. | Must Have |
| NFR-06 | The application shall expose **Spring Boot Actuator** health and metrics endpoints for operational monitoring. | Should Have |
| NFR-07 | **H2 in-memory database** shall be available by default for development and testing; MySQL shall be supported via a profile. | Must Have |
| NFR-08 | Application page response time under normal load shall not exceed **2 seconds** for any standard CRUD operation. | Should Have |
| NFR-09 | The UI shall support **internationalization (i18n)** with English and German message bundles provided. | Nice to Have |

---

## 6. Use Cases

### UC-01: Find an Owner

**Actor:** Clinic Receptionist  
**Trigger:** A pet owner arrives at the front desk.  
**Flow:**
1. Receptionist navigates to the Find Owner screen.
2. Enters the owner's last name (partial or full).
3. System searches and either redirects to the owner's profile (one match) or displays a list of matches (multiple).
4. Receptionist selects the correct owner.

**Alternate Flow:** No results found → system displays validation message; receptionist may create a new owner record.

---

### UC-02: Register a New Owner

**Actor:** Clinic Receptionist  
**Trigger:** An owner is not found in the system.  
**Flow:**
1. Receptionist clicks "Add Owner."
2. Fills in first name, last name, address, city, and telephone.
3. System validates all fields and saves the record.
4. System redirects to the new owner's detail page.

**Alternate Flow:** Validation failure → system redisplays the form with error messages; data is preserved.

---

### UC-03: Add a Pet to an Owner

**Actor:** Clinic Receptionist  
**Trigger:** An owner brings a new pet to the clinic for the first time.  
**Flow:**
1. Receptionist opens the owner's detail page.
2. Clicks "Add New Pet."
3. Enters pet name, birth date, and selects a pet type.
4. System validates and saves; redirects back to the owner's detail page.

**Business Rule Applied:** BR-09 (unique pet name per owner).

---

### UC-04: Record a Veterinary Visit

**Actor:** Clinic Receptionist or Veterinarian  
**Trigger:** A pet completes a vet visit.  
**Flow:**
1. Navigate to the pet's owner and locate the pet.
2. Click "Add Visit" for the relevant pet.
3. Enter or confirm the visit date; enter a description of the visit.
4. System saves the visit; it appears immediately in the pet's visit history.

---

### UC-05: View Veterinarian Directory

**Actor:** Clinic Receptionist  
**Trigger:** Receptionist needs to identify an available specialist.  
**Flow:**
1. Receptionist navigates to the Veterinarians page.
2. System displays all vets with their names and specialties.

---

## 7. Data Requirements

### 7.1 Entity Definitions

#### Owner
| Field | Type | Constraints |
|-------|------|-------------|
| id | Integer | Auto-generated PK |
| firstName | String | Required |
| lastName | String | Required |
| address | String | Required |
| city | String | Required |
| telephone | String | Required, numeric, max 10 digits |

#### Pet
| Field | Type | Constraints |
|-------|------|-------------|
| id | Integer | Auto-generated PK |
| name | String | Required, unique per owner |
| birthDate | LocalDate | Required, format `yyyy-MM-dd` |
| type | PetType | Required, from catalog |
| owner | Owner | Required FK |

#### Visit
| Field | Type | Constraints |
|-------|------|-------------|
| id | Integer | Auto-generated PK |
| petId | Integer | Required FK |
| date | LocalDate | Required, defaults to today |
| description | String | Required, non-empty |

#### Vet
| Field | Type | Constraints |
|-------|------|-------------|
| id | Integer | Auto-generated PK |
| firstName | String | Required |
| lastName | String | Required |
| specialties | Set\<Specialty\> | Optional, many-to-many |

### 7.2 Reference Data (Seeded)

| Catalog | Values |
|---------|--------|
| **Pet Types** | cat, dog, lizard, snake, bird, hamster |
| **Specialties** | radiology, surgery, dentistry |

---

## 8. Business Rules

| ID | Rule |
|----|------|
| BRL-01 | An owner must exist before a pet can be registered. |
| BRL-02 | A pet must exist before a visit can be recorded. |
| BRL-03 | A pet's name must be unique within the same owner's pet list. |
| BRL-04 | The owner's telephone must contain only numeric digits (max 10). |
| BRL-05 | Visit descriptions may not be blank or whitespace-only. |
| BRL-06 | Pet types and vet specialties are managed as system data and are not editable through the UI. |
| BRL-07 | Past visit records are immutable; only new visits can be added. |

---

## 9. Constraints and Assumptions

### Constraints

- The application runs on **Java 17** or later.
- The default database is **H2 in-memory**; data does not persist across restarts unless MySQL profile is activated.
- No authentication or authorization mechanism is implemented; the application assumes a trusted internal network or future integration with an identity provider.

### Assumptions

- All users operate the application on a local or corporate intranet.
- Clinic staff have basic web-browser proficiency.
- Pet type and specialty catalogs are stable and change infrequently, requiring a code deployment to update.
- A single time zone is sufficient for visit date recording.

---

## 10. Acceptance Criteria

| ID | Criterion |
|----|-----------|
| AC-01 | A receptionist can find an existing owner by partial last name in ≤ 3 navigation steps. |
| AC-02 | Creating an owner with a missing required field displays an inline validation error and does not navigate away from the form. |
| AC-03 | Attempting to add a duplicate pet name to the same owner displays an error and prevents saving. |
| AC-04 | A newly recorded visit appears immediately in the owner's detail view under the correct pet, sorted by date. |
| AC-05 | The veterinarian list page loads and displays every vet with their correct specialties (or "none"). |
| AC-06 | `GET /vets` returns a valid JSON response listing all veterinarians and their specialties. |
| AC-07 | Navigating to `/oups` renders a user-friendly 5xx error page (no raw stack trace visible). |
| AC-08 | The application starts successfully with `./gradlew bootRun` using default H2 configuration with no additional setup. |
| AC-09 | All 10 pre-seeded owners, 13 pets, and associated visit records are present on first run. |
| AC-10 | The application passes all unit, controller, and repository tests via `./gradlew test`. |

---

## 11. Glossary

| Term | Definition |
|------|------------|
| **BRD** | Business Requirements Document — formal description of business needs a system must fulfill. |
| **Owner** | A person registered with the clinic who is responsible for one or more pets. |
| **Pet** | An animal registered under an owner that receives veterinary care at the clinic. |
| **Pet Type** | A classification category for a pet (e.g., cat, dog, hamster). |
| **Visit** | A recorded interaction between a pet and the clinic, identified by date and a descriptive note. |
| **Vet / Veterinarian** | A licensed clinic professional capable of examining and treating animals. |
| **Specialty** | A medical area of expertise associated with a veterinarian (e.g., surgery, radiology). |
| **Spring Boot** | An opinionated Java/Kotlin framework for building production-ready web applications. |
| **Thymeleaf** | A server-side Java/Kotlin HTML templating engine used for generating the web UI. |
| **JPA** | Java Persistence API — standard interface for ORM-based database interaction. |
| **H2** | An embedded, in-memory relational database used in development and testing. |
| **Actuator** | Spring Boot module exposing operational endpoints (health, metrics, etc.) for monitoring. |
