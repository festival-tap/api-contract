# festival-tap/api-contract

Design-first **OpenAPI 3.0** REST specification and generated data-transfer models for the **[festival-tap](https://github.com)** platform.

This repository defines the **shared wire protocol** between the web client (`ui`) and HTTP ingress adapters (`ingress`). Neither side hand-writes request/response payloads—all models are generated directly from the OpenAPI specification and published as immutable versioned artifacts.

---

## Table of Contents

<details open>
<summary><strong>Contents</strong></summary>

* [Contract Principles](#contract-principles)
* [Repository Structure](#repository-structure)
* [API Overview](#api-overview)
* [Downstream Consumers](#downstream-consumers)
* [Local Build & Validation](#local-build--validation)
* [Packaging & Release Governance](#packaging--release-governance)
</details>

---

## Contract Principles

* **Design-First Schema Ownership:** Every REST endpoint, HTTP status code, header (`X-Festival-Pass`), and JSON payload is authored in OpenAPI 3.0 before backend or frontend implementation begins.
* **Zero Hand-Written DTOs:** Java 24 records/DTOs and TypeScript interfaces are generated automatically from `src/main/resources/openapi.yaml` during build, eliminating serialization drift.
* **Complementary to Consumer-Driven Contracts (`Pact`):** `api-contract` defines the **structural schema** (what is valid), while Pact tests in `ui` and `ingress` verify the **behavioral interaction** (what endpoints and fields are actually used).

---

## Repository Structure

```text
api-contract/
├── .github/
│   └── workflows/
│       ├── ci.yml                     # OpenAPI linting, breaking-change & DTO build checks
│       └── publish.yml                # Publishes versioned DTO JAR to GitHub Packages on tag
├── src/
│   └── main/
│       └── resources/
│           └── openapi.yaml           # Canonical OpenAPI 3.0 specification
├── .gitignore
├── LICENSE
├── pom.xml                            # Maven OpenAPI Generator build (Java 24 DTOs)
└── README.md
```

---

## API Overview

| Method | Path | Auth / Header | Behavioral Spec (`user-stories`) | Summary |
| :--- | :--- | :--- | :--- | :--- |
| **`POST`** | `/api/v1/dispensers` | Admin Auth | `us-01`, `us-06` | Register a new beer dispenser (`flow_volume`, pricing). |
| **`GET`** | `/api/v1/dispensers` | `X-Festival-Pass` | `us-02`, `us-07` | List all active dispensers and live pricing. |
| **`PUT`** | `/api/v1/dispensers/{id}/status` | `X-Festival-Pass` | `us-03`, `us-04`, `us-07` | Open or close a tap (`0.15 L` automatic safety shut-off). |
| **`GET`** | `/api/v1/dispensers/{id}/spending` | Admin Auth | `us-05`, `us-06` | Retrieve dispensed volume and revenue totals from event replay. |

*(Exact paths and schemas are governed by `src/main/resources/openapi.yaml`.)*

---

## Downstream Consumers

* **`festival-tap/ingress`:** Imports the published Java DTO JAR from **GitHub Packages** to deserialize incoming HTTP/API Gateway requests and serialize responses cleanly without coupling to `core` domain types.
* **`festival-tap/ui`:** Generates TypeScript API models and HTTP client services directly from `openapi.yaml` for the Angular 21 self-service bar and admin dashboard.
* **`festival-tap/user-stories`:** Validates all endpoints black-box over HTTPS against the documented request/response payloads.

---

## Local Build & Validation

Validate the OpenAPI specification and compile the generated Java 24 DTO classes locally:

```bash
mvn clean verify
```

Generated Java sources are output to `target/generated-sources/openapi`.

---

## Packaging & Release Governance

* **Semantic Versioning (`SemVer`):** Breaking schema or endpoint changes increment `MAJOR`; backward-compatible field or endpoint additions increment `MINOR`; documentation or validation refinements increment `PATCH`.
* **Immutable Package Publishing:** Pushing a version tag (`v1.0.0`) triggers GitHub Actions to publish an immutable JAR (`com.festivaltap:api-contract`) to **GitHub Packages** and attach the standalone `openapi.yaml` artifact to the GitHub Release.
