# Technical Workbook
## Spring Petclinic Kotlin

| Field | Value |
|---|---|
| **Document Version** | 1.0 |
| **Date** | April 10, 2026 |
| **Application** | Spring Petclinic Kotlin |
| **Language** | Kotlin 2.3.20 |
| **Framework** | Spring Boot 4.0.4 |
| **JVM Target** | Java 17 |

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [Technology Stack](#2-technology-stack)
3. [Project Structure](#3-project-structure)
4. [Build System](#4-build-system)
5. [Application Entry Point](#5-application-entry-point)
6. [Domain Model](#6-domain-model)
7. [Data Access Layer](#7-data-access-layer)
8. [Service / Business Logic Layer](#8-service--business-logic-layer)
9. [Web / Controller Layer](#9-web--controller-layer)
10. [Thymeleaf Templates](#10-thymeleaf-templates)
11. [Database Schema](#11-database-schema)
12. [Configuration Reference](#12-configuration-reference)
13. [Caching](#13-caching)
14. [Validation](#14-validation)
15. [Internationalization](#15-internationalization)
16. [Testing Strategy](#16-testing-strategy)
17. [Deployment](#17-deployment)
18. [API Reference](#18-api-reference)
19. [Key Design Decisions](#19-key-design-decisions)
20. [Dependency Reference](#20-dependency-reference)

---

## 1. Project Overview

Spring Petclinic Kotlin is a full-stack, CRUD-oriented web application modelling a veterinary clinic management system. It is the **Kotlin-language companion** to the canonical Spring Petclinic Java sample application maintained by the Spring community.

### Purpose

- Serve as a real-world reference application demonstrating Spring Boot with Kotlin.
- Illustrate idiomatic usage of Kotlin in a Spring context (constructor injection, data classes, null-safety, extension functions).
- Demonstrate Spring MVC, Spring Data JPA, Thymeleaf, JCache, and Spring Boot Actuator integration.

### Key Capabilities

| Capability | Description |
|---|---|
| Owner CRUD | Register, find, edit pet owners |
| Pet CRUD | Add and edit pets within an owner |
| Visit Recording | Log and view veterinary visits per pet |
| Vet Directory | Browse veterinarians and their specialties |
| Dual Database | H2 (in-memory, default) and MySQL (profile-activated) |
| REST API | JSON/XML endpoints for veterinarian data |
| Caching | JCache-backed vet list caching |
| Observability | Full Spring Boot Actuator exposure |
| Containerization | Docker and Google Jib support |

---

## 2. Technology Stack

| Category | Technology | Version |
|---|---|---|
| Language | Kotlin | 2.3.20 |
| Framework | Spring Boot | 4.0.4 |
| Core Framework | Spring Framework | 6.x (managed by Boot) |
| Build Tool | Gradle (Kotlin DSL) | 8+ |
| JVM Target | Java | 17 |
| Web Layer | Spring MVC | Managed |
| Templating | Thymeleaf | Managed |
| ORM | Spring Data JPA + Hibernate | Managed |
| Validation | Jakarta Bean Validation (Hibernate Validator) | Managed |
| Caching | JCache (JSR-107) + Spring `@EnableCaching` | Managed |
| Default Database | H2 (in-memory) | Managed |
| Optional Database | MySQL | Managed |
| JSON | Jackson + `jackson-module-kotlin` | Managed |
| XML Binding | JAXB (`jaxb-runtime`) | Managed |
| UI Framework | Bootstrap | 5.3.8 |
| Icons | Font Awesome | 4.7.0 |
| WebJars Locator | webjars-locator-lite | 1.1.2 |
| Observability | Spring Boot Actuator | Managed |
| Dev Tooling | Spring Boot DevTools | Managed |
| Containerization | Google Jib Plugin | 3.5.3 |
| Testing | JUnit 5, Mockito, AssertJ, MockMvc | Managed |
| Load Testing | Apache JMeter | External |

---

## 3. Project Structure

```
spring-petclinic-kotlin/
│
├── build.gradle.kts                    # Gradle build script (Kotlin DSL)
├── settings.gradle.kts                 # Project name declaration
├── gradlew / gradlew.bat               # Gradle wrapper scripts
├── gradle/wrapper/
│   └── gradle-wrapper.properties       # Wrapper version config
│
├── Dockerfile                          # Multi-stage Docker build
├── docker-compose.yml                  # MySQL container setup
│
├── AGENTS.md                           # Project description for AI agents
├── BRD.md                              # Business Requirements Document
├── TECH_WORKBOOK.md                    # This file
│
└── src/
    ├── main/
    │   ├── kotlin/org/springframework/samples/petclinic/
    │   │   ├── PetClinicApplication.kt         # @SpringBootApplication entry point
    │   │   │
    │   │   ├── model/                          # Shared base entities
    │   │   │   ├── BaseEntity.kt               # @MappedSuperclass with @Id
    │   │   │   ├── NamedEntity.kt              # Adds name column
    │   │   │   └── Person.kt                   # Adds firstName/lastName
    │   │   │
    │   │   ├── owner/                          # Owner, Pet, PetType domain
    │   │   │   ├── Owner.kt
    │   │   │   ├── Pet.kt
    │   │   │   ├── PetType.kt
    │   │   │   ├── OwnerRepository.kt
    │   │   │   ├── PetRepository.kt
    │   │   │   ├── OwnerController.kt
    │   │   │   ├── PetController.kt
    │   │   │   ├── VisitController.kt
    │   │   │   ├── PetValidator.kt
    │   │   │   └── PetTypeFormatter.kt
    │   │   │
    │   │   ├── vet/                            # Vet and Specialty domain
    │   │   │   ├── Vet.kt
    │   │   │   ├── Specialty.kt
    │   │   │   ├── VetController.kt
    │   │   │   ├── VetRepository.kt
    │   │   │   └── Vets.kt
    │   │   │
    │   │   ├── visit/                          # Visit domain
    │   │   │   ├── Visit.kt
    │   │   │   └── VisitRepository.kt
    │   │   │
    │   │   └── system/                         # Cross-cutting concerns
    │   │       ├── CacheConfig.kt
    │   │       ├── CrashController.kt
    │   │       └── WelcomeController.kt
    │   │
    │   ├── resources/
    │   │   ├── application.properties          # Main configuration
    │   │   ├── application-mysql.properties    # MySQL profile overrides
    │   │   ├── banner.txt                      # Spring Boot ASCII banner
    │   │   ├── db/
    │   │   │   ├── h2/
    │   │   │   │   ├── schema.sql              # H2 DDL
    │   │   │   │   └── data.sql                # H2 seed data
    │   │   │   └── mysql/
    │   │   │       ├── schema.sql              # MySQL DDL
    │   │   │       ├── data.sql                # MySQL seed data
    │   │   │       ├── user.sql                # MySQL user creation
    │   │   │       └── petclinic_db_setup_mysql.txt
    │   │   ├── messages/
    │   │   │   ├── messages.properties         # Default messages
    │   │   │   ├── messages_en.properties      # English
    │   │   │   └── messages_de.properties      # German
    │   │   ├── static/resources/               # Static web assets (CSS, fonts, images)
    │   │   └── templates/                      # Thymeleaf templates
    │   │       ├── welcome.html
    │   │       ├── error.html
    │   │       ├── error/5xx.html
    │   │       ├── fragments/
    │   │       │   ├── layout.html
    │   │       │   ├── inputField.html
    │   │       │   └── selectField.html
    │   │       ├── owners/
    │   │       │   ├── createOrUpdateOwnerForm.html
    │   │       │   ├── findOwners.html
    │   │       │   ├── ownerDetails.html
    │   │       │   └── ownersList.html
    │   │       ├── pets/
    │   │       │   ├── createOrUpdatePetForm.html
    │   │       │   └── createOrUpdateVisitForm.html
    │   │       └── vets/
    │   │           └── vetList.html
    │   │
    │   ├── less/                               # LESS stylesheets (compiled to CSS)
    │   │   ├── petclinic.less
    │   │   ├── header.less
    │   │   ├── responsive.less
    │   │   └── typography.less
    │   └── wro/
    │       ├── wro.xml                         # WRO4J resource groups
    │       └── wro.properties                  # WRO4J configuration
    │
    └── test/
        ├── kotlin/org/springframework/samples/petclinic/
        │   ├── PetclinicIntegrationTests.kt    # Full context integration test
        │   ├── model/                          # Model validation tests
        │   ├── owner/                          # Owner/Pet/Visit controller and repo tests
        │   ├── vet/                            # Vet controller, repo, and unit tests
        │   ├── visit/                          # Visit repository tests
        │   └── system/                         # CrashController tests
        ├── jmeter/
        │   └── petclinic_test_plan.jmx         # JMeter load test plan
        └── resources/
            └── application-test.properties     # Test-specific overrides
```

---

## 4. Build System

### Gradle Kotlin DSL (`build.gradle.kts`)

```kotlin
description = "Kotlin version of the Spring Petclinic application"
group = "org.springframework.samples"
version = "4.0.2"

plugins {
    val kotlinVersion = "2.3.20"
    id("org.springframework.boot") version "4.0.4"
    id("io.spring.dependency-management") version "1.1.7"
    id("com.google.cloud.tools.jib") version "3.5.3"
    kotlin("jvm") version kotlinVersion
    kotlin("plugin.spring") version kotlinVersion
}
```

### Java Toolchain

```kotlin
java {
    toolchain {
        languageVersion = JavaLanguageVersion.of(17)
    }
}
```

Declares Java 17 as the compilation target via Gradle's toolchain API, ensuring consistent builds regardless of locally installed JDKs.

### Kotlin Compiler Options

```kotlin
kotlin {
    compilerOptions {
        freeCompilerArgs.addAll("-Xjsr305=strict")
    }
}
```

`-Xjsr305=strict` — Treats Jakarta/Spring nullability annotations (`@NonNull`, `@Nullable`) as hard Kotlin null-safety constraints, enabling the compiler to enforce null checks at compile time rather than runtime.

### Test Runner

```kotlin
tasks.withType<Test> {
    useJUnitPlatform()
}
```

Configures all test tasks to use JUnit Platform (JUnit 5).

### Repositories

```kotlin
repositories {
    mavenCentral()
    maven { url = uri("https://repo.spring.io/snapshot") }
    maven { url = uri("https://repo.spring.io/milestone") }
}
```

Spring snapshot and milestone repositories are included because Spring Boot 4.0.4 may resolve pre-release artifacts.

### Common Gradle Commands

| Command | Description |
|---|---|
| `./gradlew build` | Compile, test, and package the application |
| `./gradlew bootRun` | Run the application with DevTools hot reload |
| `./gradlew test` | Run all tests only |
| `./gradlew jibDockerBuild` | Build Docker image locally with Jib |
| `./gradlew jib` | Push Docker image to a registry with Jib |
| `./gradlew clean` | Delete build outputs |
| `./gradlew dependencies` | Print the full dependency tree |

### Jib Configuration

```kotlin
jib {
    to {
        image = "springcommunity/spring-petclinic-kotlin"
        tags = setOf(project.version.toString(), "latest")
    }
}
```

Google Jib builds OCI-compliant container images without requiring a local Docker daemon. It tags the image with both the project version (`4.0.2`) and `latest`.

---

## 5. Application Entry Point

**File:** `PetClinicApplication.kt`

```kotlin
@SpringBootApplication(proxyBeanMethods = false)
class PetClinicApplication

fun main(args: Array<String>) {
    runApplication<PetClinicApplication>(*args)
}
```

### Key Points

| Point | Detail |
|---|---|
| `@SpringBootApplication` | Enables component scanning, auto-configuration, and `@EnableAutoConfiguration` |
| `proxyBeanMethods = false` | Disables CGLIB proxying of `@Bean` methods for faster startup and lower memory footprint (lite mode) |
| `runApplication<T>()` | Kotlin spring-boot extension function; equivalent to `SpringApplication.run(PetClinicApplication::class.java, *args)` |

---

## 6. Domain Model

### 6.1 Inheritance Hierarchy

```
BaseEntity (abstract @MappedSuperclass)
 ├── NamedEntity (@MappedSuperclass) — adds `name`
 │   ├── PetType (@Entity, table: types)
 │   └── Specialty (@Entity, table: specialties)
 └── Person (@MappedSuperclass) — adds `firstName`, `lastName`
     ├── Owner (@Entity, table: owners)
     └── Vet (@Entity, table: vets)

BaseEntity
 └── Pet (@Entity, table: pets)

BaseEntity
 └── Visit (@Entity, table: visits)
```

---

### 6.2 `BaseEntity`

```kotlin
@MappedSuperclass
open class BaseEntity : Serializable {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    open var id: Int? = null

    val isNew: Boolean
        get() = this.id == null
}
```

| Member | Purpose |
|---|---|
| `id: Int?` | Auto-generated database primary key (nullable before persist) |
| `isNew: Boolean` | Computed property; `true` when `id == null` (not yet persisted) |
| `Serializable` | Allows entity instances to be stored in HTTP sessions or distributed caches |

---

### 6.3 `NamedEntity`

```kotlin
@MappedSuperclass
open class NamedEntity : BaseEntity() {
    @Column(name = "name")
    open var name: String? = null

    override fun toString(): String = this.name ?: ""
}
```

Extends `BaseEntity` with a single `name` column. Used by `PetType` and `Specialty`.

---

### 6.4 `Person`

```kotlin
@MappedSuperclass
open class Person : BaseEntity() {
    @Column(name = "first_name")
    @NotEmpty
    var firstName = ""

    @Column(name = "last_name")
    @NotEmpty
    var lastName = ""
}
```

Extends `BaseEntity` with `firstName` and `lastName`. Both fields are validated as non-empty via Jakarta Bean Validation. Used by `Owner` and `Vet`.

---

### 6.5 `Owner`

**Table:** `owners`

```kotlin
@Entity
@Table(name = "owners")
class Owner : Person() {
    @Column(name = "address")  @NotEmpty  var address = ""
    @Column(name = "city")     @NotEmpty  var city = ""
    @Column(name = "telephone") @NotEmpty @Digits(fraction=0, integer=10)
    var telephone = ""

    @OneToMany(cascade = [CascadeType.ALL], mappedBy = "owner")
    var pets: MutableSet<Pet> = HashSet()

    fun getPets(): List<Pet> = pets.sortedWith(compareBy({ it.name }))
    fun addPet(pet: Pet) { if (pet.isNew) pets.add(pet); pet.owner = this }
    fun getPet(name: String, ignoreNew: Boolean): Pet? { ... }
}
```

| Field | Type | Constraint | DB Column |
|---|---|---|---|
| `address` | String | `@NotEmpty` | `address` |
| `city` | String | `@NotEmpty` | `city` |
| `telephone` | String | `@NotEmpty`, `@Digits(integer=10)` | `telephone` |
| `pets` | `MutableSet<Pet>` | `@OneToMany(cascade=ALL)` | via `owner_id` FK |

**Key Methods:**

| Method | Description |
|---|---|
| `getPets()` | Returns pets sorted by name (deterministic display order) |
| `addPet(pet)` | Adds non-persisted pets to set; always sets the back-reference `pet.owner` |
| `getPet(name, ignoreNew)` | Finds a pet by name; `ignoreNew=true` skips unsaved pets (used for duplicate detection) |

---

### 6.6 `Pet`

**Table:** `pets`

```kotlin
@Entity
@Table(name = "pets")
class Pet : NamedEntity() {
    @Column(name = "birth_date")
    @DateTimeFormat(pattern = "yyyy-MM-dd")
    var birthDate: LocalDate? = null

    @ManyToOne @JoinColumn(name = "type_id")
    var type: PetType? = null

    @ManyToOne @JoinColumn(name = "owner_id")
    var owner: Owner? = null

    @Transient
    var visits: MutableSet<Visit> = LinkedHashSet()

    fun getVisits(): List<Visit> = visits.sortedWith(compareBy { it.date })
    fun addVisit(visit: Visit) { visits.add(visit); visit.petId = this.id }
}
```

| Field | Type | Constraint | DB Column |
|---|---|---|---|
| `name` (inherited) | String? | via `PetValidator` | `name` |
| `birthDate` | LocalDate? | Required by `PetValidator` | `birth_date` |
| `type` | PetType? | Required for new pets; `@ManyToOne` | `type_id` (FK) |
| `owner` | Owner? | `@ManyToOne` | `owner_id` (FK) |
| `visits` | `MutableSet<Visit>` | `@Transient` — loaded separately | — |

> `visits` is marked `@Transient` and populated manually by `OwnerController` via `VisitRepository.findByPetId()`. This avoids an N+1 query issue that would occur if mapped as a JPA association on the `Pet` entity.

---

### 6.7 `PetType`

```kotlin
@Entity
@Table(name = "types")
class PetType : NamedEntity()
```

Simple entity extending `NamedEntity`. Seeded values: `cat`, `dog`, `lizard`, `snake`, `bird`, `hamster`.

---

### 6.8 `Visit`

**Table:** `visits`

```kotlin
@Entity
@Table(name = "visits")
class Visit : BaseEntity() {
    @Column(name = "visit_date")
    @DateTimeFormat(pattern = "yyyy-MM-dd")
    var date: LocalDate = LocalDate.now()

    @NotEmpty
    @Column(name = "description")
    var description: String? = null

    @Column(name = "pet_id")
    var petId: Int? = null
}
```

| Field | Type | Constraint | DB Column |
|---|---|---|---|
| `date` | LocalDate | Defaults to today | `visit_date` |
| `description` | String? | `@NotEmpty` | `description` |
| `petId` | Int? | FK to `pets.id` (not a JPA `@ManyToOne`) | `pet_id` |

> The `petId` is stored as a plain `Int?` rather than a `@ManyToOne` relationship. This is an intentional design choice to keep `Visit` independent of the `Pet` entity and avoid loading the full pet graph when fetching visits.

---

### 6.9 `Vet`

**Table:** `vets`

```kotlin
@Entity
@Table(name = "vets")
class Vet : Person() {
    @ManyToMany(fetch = FetchType.EAGER)
    @JoinTable(name = "vet_specialties",
        joinColumns = [JoinColumn(name = "vet_id")],
        inverseJoinColumns = [JoinColumn(name = "specialty_id")])
    var specialties: MutableSet<Specialty> = HashSet()

    @XmlElement
    fun getSpecialties(): List<Specialty> = specialties.sortedWith(compareBy { it.name })
    fun getNrOfSpecialties(): Int = specialties.size
    fun addSpecialty(specialty: Specialty) = specialties.add(specialty)
}
```

Specialties are fetched `EAGERLY` because the vet list always displays them and the data is small. The join table `vet_specialties` is managed by JPA.

---

### 6.10 `Specialty`

```kotlin
@Entity
@Table(name = "specialties")
class Specialty : NamedEntity()
```

Seeded values: `radiology`, `surgery`, `dentistry`.

---

### 6.11 `Vets` (DTO)

```kotlin
@XmlRootElement
class Vets(val vetList: List<Vet>)
```

A thin wrapper class used for XML and JSON serialization of the vet list. The `@XmlRootElement` annotation enables JAXB marshalling for XML responses.

---

## 7. Data Access Layer

All repositories extend Spring Data's `Repository<T, ID>` interface directly (the most minimal base interface), meaning only explicitly declared methods are available — no accidental exposure of `delete` or `deleteAll` methods.

### 7.1 `OwnerRepository`

```kotlin
interface OwnerRepository : Repository<Owner, Int> {
    @Query("SELECT DISTINCT owner FROM Owner owner left join fetch owner.pets WHERE owner.lastName LIKE :lastName%")
    @Transactional(readOnly = true)
    fun findByLastName(lastName: String): Collection<Owner>

    @Query("SELECT owner FROM Owner owner left join fetch owner.pets WHERE owner.id =:id")
    @Transactional(readOnly = true)
    fun findById(id: Int): Owner

    fun save(owner: Owner)
}
```

| Method | Description |
|---|---|
| `findByLastName(lastName)` | Case-insensitive prefix-match search with eager pet fetch (avoids N+1) using `LEFT JOIN FETCH` |
| `findById(id)` | Loads a single owner with pets eagerly |
| `save(owner)` | Insert or update |

---

### 7.2 `PetRepository`

```kotlin
interface PetRepository : Repository<Pet, Int> {
    @Query("SELECT ptype FROM PetType ptype ORDER BY ptype.name")
    @Transactional(readOnly = true)
    fun findPetTypes(): List<PetType>

    @Transactional(readOnly = true)
    fun findById(id: Int): Pet

    fun save(pet: Pet)
}
```

| Method | Description |
|---|---|
| `findPetTypes()` | Returns all pet types sorted alphabetically (used to populate form dropdowns) |
| `findById(id)` | Load a pet by ID |
| `save(pet)` | Insert or update |

---

### 7.3 `VisitRepository`

```kotlin
interface VisitRepository : Repository<Visit, Int> {
    fun save(visit: Visit)
    fun findByPetId(petId: Int): MutableSet<Visit>
}
```

| Method | Description |
|---|---|
| `save(visit)` | Insert |
| `findByPetId(petId)` | Load all visits for a pet (Spring Data derives the query from the method name) |

---

### 7.4 `VetRepository`

```kotlin
interface VetRepository : Repository<Vet, Int> {
    @Transactional(readOnly = true)
    @Cacheable("vets")
    fun findAll(): Collection<Vet>
}
```

| Method | Description |
|---|---|
| `findAll()` | Loads all vets and caches the result in the `vets` JCache region |

---

## 8. Service / Business Logic Layer

This application has **no explicit service layer**. Business logic is embedded either in:

- **Domain entities** (`Owner.addPet()`, `Owner.getPet()`, `Pet.addVisit()`)
- **Controllers** (orchestration logic, e.g., duplicate-pet-name detection in `PetController`)
- **Validators** (`PetValidator`)
- **Formatters** (`PetTypeFormatter`)

This is acceptable for a reference application of this complexity. In a production system, a dedicated service layer would isolate transaction boundaries and business rules from HTTP concerns.

### 8.1 `PetValidator`

Custom Spring `Validator` — bypasses Jakarta Bean Validation annotations for richer programmatic rules.

```kotlin
class PetValidator : Validator {
    override fun validate(obj: Any, errors: Errors) {
        val pet = obj as Pet
        if (!StringUtils.hasLength(pet.name))   errors.rejectValue("name", REQUIRED, REQUIRED)
        if (pet.isNew && pet.type == null)       errors.rejectValue("type", REQUIRED, REQUIRED)
        if (pet.birthDate == null)               errors.rejectValue("birthDate", REQUIRED, REQUIRED)
    }
    override fun supports(clazz: Class<*>) = Pet::class.java.isAssignableFrom(clazz)
    companion object { const val REQUIRED = "required" }
}
```

Registered via `@InitBinder("pet")` in `PetController`:
```kotlin
@InitBinder("pet")
fun initPetBinder(dataBinder: WebDataBinder) {
    dataBinder.validator = PetValidator()
}
```

### 8.2 `PetTypeFormatter`

Spring `Formatter<PetType>` — converts between the string form binding value (type name) and the `PetType` entity.

```kotlin
@Component
class PetTypeFormatter(val pets: PetRepository) : Formatter<PetType> {
    override fun print(petType: PetType, locale: Locale): String = petType.name ?: ""
    override fun parse(text: String, locale: Locale): PetType {
        return pets.findPetTypes().find { it.name == text }
               ?: throw ParseException("type not found: $text", 0)
    }
}
```

Registered automatically as a `@Component` and consumed by Spring MVC's `ConversionService`.

---

## 9. Web / Controller Layer

All controllers are annotated with `@Controller` and use constructor injection of repository dependencies.

### 9.1 `WelcomeController`

```kotlin
@Controller
class WelcomeController {
    @GetMapping("/")
    fun welcome(): String = "welcome"
}
```

Serves the landing page at `/`.

---

### 9.2 `OwnerController`

**Injected:** `OwnerRepository`, `VisitRepository`

| Mapping | Method | Description |
|---|---|---|
| `GET /owners/new` | `initCreationForm` | Render blank owner form |
| `POST /owners/new` | `processCreationForm` | Validate and save new owner |
| `GET /owners/find` | `initFindForm` | Render search form |
| `GET /owners` | `processFindForm` | Execute search; redirect or list |
| `GET /owners/{ownerId}` | `showOwner` | Owner detail with pets and visits |
| `GET /owners/{ownerId}/edit` | `initUpdateOwnerForm` | Render pre-filled edit form |
| `POST /owners/{ownerId}/edit` | `processUpdateOwnerForm` | Validate and update owner |

**`@InitBinder`** disallows binding of the `id` field for security (prevents mass-assignment of the primary key from form data).

**`showOwner` flow** — manually loads visits to avoid lazy-load issues:
```kotlin
for (pet in owner.getPets()) {
    pet.visits = visits.findByPetId(pet.id!!)
}
```

---

### 9.3 `PetController`

**Base path:** `/owners/{ownerId}`  
**Injected:** `PetRepository`, `OwnerRepository`

| Mapping | Method | Description |
|---|---|---|
| `GET /pets/new` | `initCreationForm` | Render blank pet form |
| `POST /pets/new` | `processCreationForm` | Validate, check duplicate name, save |
| `GET /pets/{petId}/edit` | `initUpdateForm` | Render pre-filled pet edit form |
| `POST /pets/{petId}/edit` | `processUpdateForm` | Validate and update pet |

**`@ModelAttribute` methods** run before every request handler:
- `populatePetTypes()` — injects pet type list into model for dropdown rendering
- `findOwner(ownerId)` — loads the owner so it's available in every handler

**Duplicate name check:**
```kotlin
if (StringUtils.hasLength(pet.name) && pet.isNew && owner.getPet(pet.name!!, true) != null) {
    result.rejectValue("name", "duplicate", "already exists")
}
```

---

### 9.4 `VisitController`

**Injected:** `VisitRepository`, `PetRepository`

| Mapping | Method | Description |
|---|---|---|
| `GET /owners/*/pets/{petId}/visits/new` | `initNewVisitForm` | Render visit form |
| `POST /owners/{ownerId}/pets/{petId}/visits/new` | `processNewVisitForm` | Validate and save visit |

**`@ModelAttribute("visit")`** — `loadPetWithVisit()` runs before every request, loads the pet, creates a new `Visit`, and attaches it via `pet.addVisit(visit)`.

---

### 9.5 `VetController`

**Injected:** `VetRepository`

| Mapping | Produces | Method | Description |
|---|---|---|---|
| `GET /vets.html` | `text/html` | `showHtmlVetList` | Thymeleaf vet list |
| `GET /vets.json` | `application/json` | `showJsonVetList` | JSON vet list |
| `GET /vets.xml` | `application/xml` | `showXmlVetList` | XML vet list |

---

### 9.6 `CrashController`

```kotlin
@Controller
class CrashController {
    @GetMapping("/oups")
    fun triggerException(): String {
        throw RuntimeException("Expected: controller used to showcase what happens when an exception is thrown")
    }
}
```

Intentionally throws a `RuntimeException` to demonstrate Spring Boot's global error handling and `error/5xx.html` error view.

---

## 10. Thymeleaf Templates

All templates use **Thymeleaf 3** with `th:*` attributes and are stored under `src/main/resources/templates/`.

### Layout System

`fragments/layout.html` provides the site-wide shell (Bootstrap navbar, footer, CSS, JS includes). All page templates insert their content via the Thymeleaf `th:fragment` / `th:insert` mechanism.

### Template Map

| Template | Route(s) | Description |
|---|---|---|
| `welcome.html` | `GET /` | Welcome / home page |
| `error.html` | Error routing | Generic error page |
| `error/5xx.html` | 5xx errors | Server error page (shown on `/oups`) |
| `owners/findOwners.html` | `GET /owners/find` | Owner search form |
| `owners/ownersList.html` | `GET /owners` (multiple results) | Search results list |
| `owners/createOrUpdateOwnerForm.html` | `GET/POST /owners/new`, `GET/POST /owners/{id}/edit` | Owner create/edit form |
| `owners/ownerDetails.html` | `GET /owners/{id}` | Owner profile with pets and visits |
| `pets/createOrUpdatePetForm.html` | `GET/POST .../pets/new`, `GET/POST .../pets/{id}/edit` | Pet create/edit form |
| `pets/createOrUpdateVisitForm.html` | `GET/POST .../visits/new` | Visit create form |
| `vets/vetList.html` | `GET /vets.html` | Veterinarian directory |

### Fragment Helpers

| Fragment | Purpose |
|---|---|
| `fragments/inputField.html` | Reusable text input with label and validation error display |
| `fragments/selectField.html` | Reusable select/dropdown with label and validation error display |

---

## 11. Database Schema

### H2 Schema (`db/h2/schema.sql`)

```sql
-- Drop order respects FK constraints
DROP TABLE vet_specialties IF EXISTS;
DROP TABLE vets IF EXISTS;
DROP TABLE specialties IF EXISTS;
DROP TABLE visits IF EXISTS;
DROP TABLE pets IF EXISTS;
DROP TABLE types IF EXISTS;
DROP TABLE owners IF EXISTS;

CREATE TABLE vets (
  id         INTEGER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
  first_name VARCHAR(30),
  last_name  VARCHAR(30)
);
CREATE INDEX vets_last_name ON vets (last_name);

CREATE TABLE specialties (
  id   INTEGER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
  name VARCHAR(80)
);
CREATE INDEX specialties_name ON specialties (name);

CREATE TABLE vet_specialties (
  vet_id       INTEGER NOT NULL,
  specialty_id INTEGER NOT NULL
);
ALTER TABLE vet_specialties ADD CONSTRAINT fk_vet_specialties_vets
    FOREIGN KEY (vet_id) REFERENCES vets (id);
ALTER TABLE vet_specialties ADD CONSTRAINT fk_vet_specialties_specialties
    FOREIGN KEY (specialty_id) REFERENCES specialties (id);

CREATE TABLE types (
  id   INTEGER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
  name VARCHAR(80)
);
CREATE INDEX types_name ON types (name);

CREATE TABLE owners (
  id         INTEGER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
  first_name VARCHAR(30),
  last_name  VARCHAR_IGNORECASE(30),   -- H2-specific: case-insensitive search
  address    VARCHAR(255),
  city       VARCHAR(80),
  telephone  VARCHAR(20)
);
CREATE INDEX owners_last_name ON owners (last_name);

CREATE TABLE pets (
  id         INTEGER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
  name       VARCHAR(30),
  birth_date DATE,
  type_id    INTEGER NOT NULL,
  owner_id   INTEGER NOT NULL
);
ALTER TABLE pets ADD CONSTRAINT fk_pets_owners FOREIGN KEY (owner_id) REFERENCES owners (id);
ALTER TABLE pets ADD CONSTRAINT fk_pets_types  FOREIGN KEY (type_id)  REFERENCES types (id);
CREATE INDEX pets_name ON pets (name);

CREATE TABLE visits (
  id          INTEGER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
  pet_id      INTEGER NOT NULL,
  visit_date  DATE,
  description VARCHAR(255)
);
ALTER TABLE visits ADD CONSTRAINT fk_visits_pets FOREIGN KEY (pet_id) REFERENCES pets (id);
CREATE INDEX visits_pet_id ON visits (pet_id);
```

### Seed Data (`db/h2/data.sql`)

| Entity | Count | Examples |
|---|---|---|
| Vets | 6 | James Carter, Helen Leary, Linda Douglas, Rafael Ortega, Henry Stevens, Sharon Jenkins |
| Specialties | 3 | radiology, surgery, dentistry |
| Pet Types | 6 | cat, dog, lizard, snake, bird, hamster |
| Owners | 10 | George Franklin, Betty Davis, … |
| Pets | 13 | Leo (cat), Basil (hamster), Rosy (dog), … |
| Visits | 2+ | Rabies shot, stitches removed |

### Entity-Relationship Diagram

```
OWNERS ──────< PETS >──────── TYPES
                 │
                 └──────< VISITS

VETS >──────────────────────< SPECIALTIES
         (via vet_specialties)
```

---

## 12. Configuration Reference

### `application.properties`

```properties
# Database selection: h2 or mysql
database=h2
spring.sql.init.schema-locations=classpath*:db/${database}/schema.sql
spring.sql.init.data-locations=classpath*:db/${database}/data.sql

# Thymeleaf
spring.thymeleaf.mode=HTML

# JPA
spring.jpa.hibernate.ddl-auto=none        # Schema managed by SQL scripts only
spring.jpa.open-in-view=false             # Disable Open-Session-In-View anti-pattern

# Parallel repository bootstrapping
spring.data.jpa.repositories.bootstrap-mode=deferred

# i18n
spring.messages.basename=messages/messages

# Actuator — expose all endpoints
management.endpoints.web.exposure.include=*

# Logging
logging.level.org.springframework=INFO

# Static resource cache — 12-hour browser cache
spring.web.resources.cache.cachecontrol.max-age=12h
```

### `application-mysql.properties` (activated via `--spring.profiles.active=mysql`)

```properties
database=mysql
spring.datasource.url=jdbc:mysql://localhost/petclinic
spring.datasource.username=petclinic
spring.datasource.password=petclinic
spring.sql.init.mode=always              # Re-run init scripts on startup
```

### `application-test.properties`

```properties
spring.web.error.include-message=always  # Exposes error messages in test assertions
```

### Selecting the MySQL Profile

```bash
# Via Gradle
./gradlew bootRun --args='--spring.profiles.active=mysql'

# Via Java
java -jar spring-petclinic-kotlin.jar --spring.profiles.active=mysql

# Via environment variable
SPRING_PROFILES_ACTIVE=mysql java -jar spring-petclinic-kotlin.jar
```

---

## 13. Caching

### Configuration (`CacheConfig.kt`)

```kotlin
@Configuration(proxyBeanMethods = false)
@EnableCaching
class CacheConfig {
    @Bean
    fun cacheManagerCustomizer(): JCacheManagerCustomizer {
        return JCacheManagerCustomizer {
            it.createCache("vets", createCacheConfiguration())
        }
    }

    private fun createCacheConfiguration(): Configuration<Any, Any> =
        MutableConfiguration<Any, Any>().setStatisticsEnabled(true)
}
```

### How It Works

| Aspect | Detail |
|---|---|
| Cache provider | JCache (JSR-107) API — implementation provided at runtime |
| Cache region | `vets` |
| Statistics | Enabled — accessible via JMX |
| Activation | `@Cacheable("vets")` on `VetRepository.findAll()` |
| Eviction | None configured; cache lives for the duration of the JVM process |
| Verified by | `PetclinicIntegrationTests.testFindAll()` — calls `findAll()` twice; second call is served from cache |

### Cache Flow

```
First call to VetRepository.findAll()
  → Cache MISS → DB query executed → result stored in "vets" cache
  
Subsequent calls to VetRepository.findAll()
  → Cache HIT → result returned from cache (no DB query)
```

---

## 14. Validation

### Owner Validation (Jakarta Bean Validation)

Handled by `@Valid` in controller + annotations on `Owner` / `Person`:

| Field | Annotation | Rule |
|---|---|---|
| `firstName` | `@NotEmpty` | Must not be blank |
| `lastName` | `@NotEmpty` | Must not be blank |
| `address` | `@NotEmpty` | Must not be blank |
| `city` | `@NotEmpty` | Must not be blank |
| `telephone` | `@NotEmpty`, `@Digits(fraction=0, integer=10)` | Must be numeric, max 10 digits |

### Pet Validation (Custom `PetValidator`)

| Field | Rule |
|---|---|
| `name` | Must not be blank (`StringUtils.hasLength`) |
| `type` | Required if the pet is new |
| `birthDate` | Must not be null |
| `name` (duplicate) | Checked in `PetController` against owner's existing pets |

### Visit Validation (Jakarta Bean Validation)

| Field | Annotation | Rule |
|---|---|---|
| `description` | `@NotEmpty` | Must not be blank |

### Validation Flow

```
HTTP Request → Controller @Valid parameter
    → BindingResult populated with errors
    → If errors: re-render form template (data preserved)
    → If no errors: save and redirect
```

Form errors are displayed inline using the `fragments/inputField.html` and `fragments/selectField.html` template fragments which iterate `BindingResult` errors.

---

## 15. Internationalization

Message bundles are located in `src/main/resources/messages/`:

| File | Locale |
|---|---|
| `messages.properties` | Default (fallback) |
| `messages_en.properties` | English |
| `messages_de.properties` | German |

Registered via:
```properties
spring.messages.basename=messages/messages
```

Templates reference messages using Thymeleaf's `#{key}` syntax. The active locale is determined by Spring MVC's `LocaleResolver` (defaults to `Accept-Language` HTTP header).

---

## 16. Testing Strategy

### Test Categories

| Type | Annotation | Scope | Files |
|---|---|---|---|
| Integration | `@SpringBootTest` | Full Spring context + H2 | `PetclinicIntegrationTests.kt` |
| Controller (MVC) | `@WebMvcTest` | Slice: web layer only, mocked repos | `*ControllerTest.kt` |
| Repository (JPA) | `@DataJpaTest` | Slice: JPA layer only, H2 | `*RepositoryTest.kt` |
| Unit | None (plain JUnit 5) | No Spring context | `VetTest.kt`, `PetTypeFormatterTest.kt`, `ValidatorTests.kt` |

### Test Coverage

| Module | Controller Test | Repository Test | Unit Test |
|---|---|---|---|
| Owner | `OwnerControllerTest` | `OwnerRepositoryTest` | — |
| Pet | `PetControllerTest` | `PetRepositoryTest` | `PetTypeFormatterTest` |
| Visit | `VisitControllerTest` | `VisitRepositoryTest` | — |
| Vet | `VetControllerTest` | `VetRepositoryTest` | `VetTest` |
| System | `CrashControllerTest` | — | — |
| Cross-module | — | — | `PetclinicIntegrationTests` |

### Integration Test

```kotlin
@ExtendWith(SpringExtension::class)
@SpringBootTest
class PetclinicIntegrationTests(@param:Autowired private val vets: VetRepository) {
    @Test
    fun testFindAll() {
        vets.findAll()           // cache MISS — DB hit
        vets.findAll()           // cache HIT — verifies caching works
    }
}
```

### Controller Test Pattern (`@WebMvcTest`)

```kotlin
@ExtendWith(SpringExtension::class)
@WebMvcTest(OwnerController::class)
class OwnerControllerTest {
    @Autowired lateinit var mockMvc: MockMvc
    @MockitoBean private lateinit var owners: OwnerRepository
    @MockitoBean private lateinit var visits: VisitRepository

    @Test
    fun testInitCreationForm() {
        mockMvc.perform(get("/owners/new"))
            .andExpect(status().isOk)
            .andExpect(model().attributeExists("owner"))
            .andExpect(view().name("owners/createOrUpdateOwnerForm"))
    }
}
```

`@WebMvcTest` loads only the web layer (controllers, formatters, validators). Repositories are replaced with Mockito mocks via `@MockitoBean`.

### VetController Test — Multi-format Assertions

```kotlin
@Test
fun testShowResourcesVetList() {
    mockMvc.perform(get("/vets.json").accept(APPLICATION_JSON))
        .andExpect(status().isOk)
        .andExpect(content().contentType(APPLICATION_JSON))
        .andExpect(jsonPath("$.vetList[0].id").value(1))
}

@Test
fun testShowVetListXml() {
    mockMvc.perform(get("/vets.xml").accept(APPLICATION_XML))
        .andExpect(status().isOk)
        .andExpect(content().contentType(APPLICATION_XML_VALUE))
        .andExpect(content().node(hasXPath("/vets/vetList[id=1]/id")))
}
```

### Running Tests

```bash
# All tests
./gradlew test

# Specific test class
./gradlew test --tests "*.OwnerControllerTest"

# With test report
./gradlew test jacocoTestReport
```

Test reports are generated in `build/reports/tests/test/index.html`.

---

## 17. Deployment

### 17.1 Running Locally

```bash
# Default profile (H2 in-memory)
./gradlew bootRun

# MySQL profile
./gradlew bootRun --args='--spring.profiles.active=mysql'

# Packaged JAR
./gradlew build
java -jar build/libs/spring-petclinic-kotlin-4.0.2.jar
```

Application starts at: `http://localhost:8080`  
H2 Console (dev only): `http://localhost:8080/h2-console`

### 17.2 Dockerfile

```dockerfile
FROM gradle:4.7.0-jdk8-alpine AS build
COPY --chown=gradle:gradle . /home/gradle/src
WORKDIR /home/gradle/src
RUN gradle build

FROM openjdk:8-jre-slim
EXPOSE 8080
COPY --from=build /home/gradle/src/build/libs/spring-petclinic-kotlin.jar /app/
RUN bash -c 'touch /app/spring-petclinic-kotlin.jar'
ENTRYPOINT ["java",
  "-XX:+UnlockExperimentalVMOptions",
  "-XX:+UseCGroupMemoryLimitForHeap",
  "-Djava.security.egd=file:/dev/./urandom",
  "-jar", "/app/spring-petclinic-kotlin.jar"]
```

Multi-stage build: Gradle build stage → slim JRE runtime stage.

```bash
# Build image
docker build -t spring-petclinic-kotlin .

# Run image
docker run -p 8080:8080 spring-petclinic-kotlin
```

### 17.3 Docker Compose (MySQL)

```yaml
mysql:
  image: mysql:5.7
  ports:
    - "3306:3306"
  environment:
    - MYSQL_ROOT_PASSWORD=
    - MYSQL_ALLOW_EMPTY_PASSWORD=true
    - MYSQL_USER=petclinic
    - MYSQL_PASSWORD=petclinic
    - MYSQL_DATABASE=petclinic
  volumes:
    - "./conf.d:/etc/mysql/conf.d:ro"
```

```bash
# Start MySQL container
docker-compose up -d

# Run application against it
./gradlew bootRun --args='--spring.profiles.active=mysql'
```

### 17.4 Google Jib (Containerization without Docker Daemon)

```bash
# Build and load into local Docker daemon
./gradlew jibDockerBuild

# Build and push to Docker Hub
./gradlew jib \
  -Djib.to.auth.username=<username> \
  -Djib.to.auth.password=<password>
```

Produces image: `springcommunity/spring-petclinic-kotlin:4.0.2` and `springcommunity/spring-petclinic-kotlin:latest`

### 17.5 Spring Boot Actuator Endpoints

All actuator endpoints are exposed via:
```properties
management.endpoints.web.exposure.include=*
```

| Endpoint | URL | Description |
|---|---|---|
| Health | `/actuator/health` | Application health status |
| Info | `/actuator/info` | Application info |
| Metrics | `/actuator/metrics` | Micrometer metrics |
| Caches | `/actuator/caches` | Cache statistics |
| Beans | `/actuator/beans` | Spring bean definitions |
| Env | `/actuator/env` | Environment properties |
| Mappings | `/actuator/mappings` | All request mappings |

---

## 18. API Reference

### HTML Endpoints (returns Thymeleaf views)

| Method | URL | Controller | View |
|---|---|---|---|
| GET | `/` | `WelcomeController` | `welcome` |
| GET | `/owners/find` | `OwnerController` | `owners/findOwners` |
| GET | `/owners` | `OwnerController` | `owners/ownersList` or redirect |
| GET | `/owners/new` | `OwnerController` | `owners/createOrUpdateOwnerForm` |
| POST | `/owners/new` | `OwnerController` | redirect or form |
| GET | `/owners/{ownerId}` | `OwnerController` | `owners/ownerDetails` |
| GET | `/owners/{ownerId}/edit` | `OwnerController` | `owners/createOrUpdateOwnerForm` |
| POST | `/owners/{ownerId}/edit` | `OwnerController` | redirect or form |
| GET | `/owners/{ownerId}/pets/new` | `PetController` | `pets/createOrUpdatePetForm` |
| POST | `/owners/{ownerId}/pets/new` | `PetController` | redirect or form |
| GET | `/owners/{ownerId}/pets/{petId}/edit` | `PetController` | `pets/createOrUpdatePetForm` |
| POST | `/owners/{ownerId}/pets/{petId}/edit` | `PetController` | redirect or form |
| GET | `/owners/*/pets/{petId}/visits/new` | `VisitController` | `pets/createOrUpdateVisitForm` |
| POST | `/owners/{ownerId}/pets/{petId}/visits/new` | `VisitController` | redirect or form |
| GET | `/vets.html` | `VetController` | `vets/vetList` |
| GET | `/oups` | `CrashController` | `error/5xx` |

### Data Endpoints

| Method | URL | Produces | Response |
|---|---|---|---|
| GET | `/vets.json` | `application/json` | `Vets` JSON object with `vetList` array |
| GET | `/vets.xml` | `application/xml` | `<vets>` XML with `<vetList>` elements |

#### Example JSON Response — `GET /vets.json`

```json
{
  "vetList": [
    {
      "id": 1,
      "firstName": "James",
      "lastName": "Carter",
      "specialties": []
    },
    {
      "id": 2,
      "firstName": "Helen",
      "lastName": "Leary",
      "specialties": [
        { "id": 1, "name": "radiology" }
      ]
    }
  ]
}
```

---

## 19. Key Design Decisions

| Decision | Rationale |
|---|---|
| `proxyBeanMethods = false` on `@SpringBootApplication` and `@Configuration` | Avoids CGLIB subclass generation; improves startup time and reduces memory. Safe when no `@Bean` methods call each other directly. |
| Constructor injection everywhere | Kotlin's primary constructor syntax makes this natural. Avoids `lateinit var` with `@Autowired`; enables `val` dependencies. |
| `-Xjsr305=strict` compiler flag | Bridges the gap between Kotlin null-safety and Jakarta/Spring annotation-based nullability, catching potential NPEs at compile time. |
| `spring.data.jpa.repositories.bootstrap-mode=deferred` | Allows JPA repositories to bootstrap in a parallel thread, reducing application startup time. |
| `spring.jpa.open-in-view=false` | Prevents the anti-pattern of keeping the JPA `EntityManager` open for the duration of a web request. Encourages loading all required data in the controller. |
| `Visit.petId` as `Int?` instead of `@ManyToOne` | Keeps `Visit` decoupled from `Pet`. Visits are queried directly by `pet_id` in `VisitRepository`. |
| `Pet.visits` as `@Transient` | Visits are loaded separately from `VisitRepository` in `OwnerController.showOwner()`. Avoids making `Pet` the aggregate root for visit loading and prevents over-fetching. |
| `Repository<T, ID>` as base interface | The most minimal Spring Data interface. Only explicitly declared methods are available — `delete`, `deleteAll`, `findAll` etc. are not automatically exposed. |
| `@Cacheable` directly on repository method | Simple and effective for a read-mostly, stable dataset like the vet list. |
| Custom `PetValidator` instead of Bean Validation annotations | Conditional validation logic (e.g., type only required for new pets) is easier to express programmatically. |
| H2 `VARCHAR_IGNORECASE` for `owners.last_name` | Enables case-insensitive LIKE searches without altering the JPQL query. SQL server-specific feature for H2 only. |

---

## 20. Dependency Reference

### Runtime Dependencies

| Artifact | Purpose |
|---|---|
| `spring-boot-starter-actuator` | Health, metrics, and ops endpoints |
| `spring-boot-starter-cache` | `@EnableCaching` and Spring cache abstraction |
| `spring-boot-starter-data-jpa` | Spring Data JPA + Hibernate ORM |
| `spring-boot-starter-validation` | Jakarta Bean Validation (Hibernate Validator) |
| `spring-boot-starter-webmvc` | Spring MVC (DispatcherServlet, etc.) |
| `spring-boot-starter-thymeleaf` | Thymeleaf template engine |
| `jackson-module-kotlin` | Kotlin-aware Jackson JSON serialization |
| `jaxb-runtime` | JAXB XML marshalling for `Vets.kt` |
| `javax.cache:cache-api` | JCache (JSR-107) API |
| `kotlin-reflect` | Required by Spring for Kotlin class introspection |
| `bootstrap` (webjar) | Bootstrap CSS framework (5.3.8) |
| `font-awesome` (webjar) | Icon font (4.7.0) |
| `webjars-locator-lite` | Version-agnostic WebJar URL resolution |
| `h2` | In-memory H2 database (runtime) |
| `mysql-connector-j` | MySQL JDBC driver (runtime) |

### Development Dependencies

| Artifact | Purpose |
|---|---|
| `spring-boot-devtools` | Live reload and fast restart during development |

### Test Dependencies

| Artifact | Purpose |
|---|---|
| `spring-boot-starter-test` | JUnit 5, Mockito, AssertJ, `@SpringBootTest`, `@WebMvcTest`, `@DataJpaTest` |
| `spring-boot-starter-webflux` | `WebTestClient` for reactive testing (included for full test support) |
| `junit-jupiter-api` | JUnit 5 API |
| `junit-jupiter-engine` | JUnit 5 runtime engine |

---

*End of Technical Workbook — Spring Petclinic Kotlin v4.0.2*
