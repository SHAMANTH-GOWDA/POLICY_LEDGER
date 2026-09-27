# POLICY_LEDGER

# Policy Ledger — Spring Boot Life Insurance Application

A guided Spring Boot application designed to manage **life insurance policies, customers, riders, and claims**. The project follows a layered architecture so that the implementation can be understood progressively, starting with the domain and database layer and moving through repositories, business services, validation, REST APIs, and centralized exception handling.

> **Project status:** The guided implementation is complete, including the Maven configuration. The application can be compiled, packaged, and executed using the commands provided below.

## Project Layout

The project is organized into separate packages based on their responsibilities:

```text
.
├── README.md
├── pom.xml
├── .gitignore
└── src/
    └── main/
        ├── java/com/hdfclife/ledger/
        │   ├── LedgerApplication.java
        │   ├── config/
        │   │   ├── DataSeeder.java
        │   │   ├── OpenApiConfig.java
        │   │   └── RestExceptionHandler.java
        │   ├── domain/
        │   │   ├── Claim.java
        │   │   ├── Customer.java
        │   │   ├── Policy.java
        │   │   └── Rider.java
        │   ├── dto/
        │   │   ├── ClaimResponse.java
        │   │   ├── CreateClaimRequest.java
        │   │   ├── CreatePolicyRequest.java
        │   │   ├── ErrorResponse.java
        │   │   └── PolicyResponse.java
        │   ├── exception/
        │   │   ├── ClaimNotFoundException.java
        │   │   ├── DeskException.java
        │   │   ├── DuplicatePolicyException.java
        │   │   ├── InvalidRequestException.java
        │   │   └── PolicyNotFoundException.java
        │   ├── repo/
        │   │   ├── ClaimRepository.java
        │   │   ├── CustomerRepository.java
        │   │   ├── PolicyRepository.java
        │   │   └── RiderRepository.java
        │   ├── service/
        │   │   ├── ClaimService.java
        │   │   └── PolicyService.java
        │   ├── validation/
        │   │   ├── PolicyType.java
        │   │   └── PolicyTypeValidator.java
        │   └── web/
        │       ├── ClaimController.java
        │       └── PolicyController.java
        └── resources/
            ├── application.yml
            ├── application-dev.yml
            ├── application-prod.yml
            └── db/migration/
                ├── V1__hdfc_life_schema.sql
                ├── V2__seed_reference_riders.sql
                └── V3__seed_policy_ledger.sql
```

## Layer and Package Responsibilities

The application separates responsibilities across different layers:

* **`LedgerApplication.java`**
  Acts as the main Spring Boot entry point and starts the application.

* **`config/`**
  Contains application-level configuration classes, OpenAPI/Swagger configuration, data-seeding support, and the centralized REST exception handler. Reference and sample data are currently loaded through Flyway migrations.

* **`domain/`**
  Contains the JPA entity classes representing the application's core data model. These entities correspond to the Flyway-managed tables: `customers`, `policies`, `claims`, `riders`, and `policy_riders`.

* **`dto/`**
  Defines the objects exchanged between the API and its clients. This package contains request models, response models, and the common error response structure used by the application.

* **`exception/`**
  Contains application-specific exceptions for situations such as missing policies or claims, duplicate policies, and invalid requests. A common base exception is also provided.

* **`repo/`**
  Represents the persistence layer. The repositories provide database access for customers, policies, claims, and riders.

* **`service/`**
  Implements the application's business logic. The policy and claim services coordinate repository operations and apply the required business rules before returning results to the API layer.

* **`validation/`**
  Contains policy-product validation logic. Supported policy types include `TERM`, `ULIP`, and `ENDOWMENT`, with a custom `@ValidPolicyType` validator used to validate policy types.

* **`web/`**
  Provides the REST API controllers responsible for handling policy and claim requests.

* **`resources/application*.yml`**
  Stores the default, development, and production configuration for the application.

* **`resources/db/migration/`**
  Contains the Flyway migration scripts responsible for creating the database schema and loading rider reference data and sample policy records.

## Application Configuration

The application starts with the **`dev` profile by default** and runs on port **8080**.

For development, the application uses an in-memory **H2 database**:

```text
jdbc:h2:mem:ledgerdb
```

The development configuration also enables:

* SQL statement logging
* Flyway database migrations
* H2 database console

The H2 console can be accessed through:

```text
http://localhost:8080/h2-console
```

Because H2 is configured as an in-memory database, all database contents are recreated whenever the application is restarted.

### Production Database

The production profile is configured to use **PostgreSQL**:

```text
DB_URL=jdbc:postgresql://localhost:5432/ledgerdb
DB_USERNAME=postgres
DB_PASSWORD=postgres
```

These environment variables should be changed according to the PostgreSQL installation being used.

Hibernate is configured to **validate the existing database schema rather than create or modify it**. The database structure is therefore managed through Flyway migrations.

## Building and Starting the Application

The project requires a **JDK 17 or later** and a Maven version compatible with the Spring Boot version specified in `pom.xml`.

From the root directory of the project, the application can be started with:

```bash
# Launch using the default development configuration
mvn spring-boot:run

# Explicitly run with the development profile
mvn spring-boot:run -Dspring-boot.run.profiles=dev

# Build the project
mvn clean package

# Start the generated JAR
java -jar target/*.jar
```

### Running with PostgreSQL

To run the application using the production PostgreSQL configuration, provide the required database environment variables:

```bash
export DB_URL="jdbc:postgresql://localhost:5432/ledgerdb"
export DB_USERNAME="postgres"
export DB_PASSWORD="your-password"

mvn spring-boot:run -Dspring-boot.run.profiles=prod
```

Once the application has started, the main application URL is:

```text
http://localhost:8080
```

When running with the development profile, the H2 console is available at:

```text
http://localhost:8080/h2-console
```

The Swagger/OpenAPI interface can be accessed at:

```text
http://localhost:8080/swagger-ui.html
```

## REST API

The application exposes endpoints for creating and retrieving policies and claims.

| Method | Endpoint                   | Purpose                                                    |
| ------ | -------------------------- | ---------------------------------------------------------- |
| `POST` | `/api/policies`            | Creates a new policy and optionally associates rider codes |
| `GET`  | `/api/policies`            | Retrieves the complete list of policies                    |
| `GET`  | `/api/policies/{policyNo}` | Retrieves a specific policy using its policy number        |
| `POST` | `/api/claims`              | Creates a claim for a policy that is currently Active      |
| `GET`  | `/api/claims`              | Returns all claims                                         |
| `GET`  | `/api/claims/{claimNo}`    | Retrieves a specific claim using its claim number          |

## Flyway Database Migrations

Database initialization is handled by **Flyway**. The migration files are executed sequentially according to their version numbers.

### V1 — Database Schema

```text
V1__hdfc_life_schema.sql
```

Creates the relational tables required for:

* Customers
* Policies
* Claims
* Riders
* Policy-rider relationships

### V2 — Rider Reference Data

```text
V2__seed_reference_riders.sql
```

Adds the predefined rider/reference types used by the application.

### V3 — Sample Policy Data

```text
V3__seed_policy_ledger.sql
```

Adds sample customers, policies, and claims and establishes a rider association with a policy.

This approach allows the database structure and initial reference/sample data to be reproduced consistently whenever the migrations are executed.
