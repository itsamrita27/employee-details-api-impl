# Employee Details API — Implementation

A **MuleSoft 4** implementation of the Employee Details API, built with API-led connectivity principles.  
Exposes a RESTful CRUD interface over `/api/employees` backed by an FTP-stored JSON data file and governed by a RAML 1.0 specification published to Anypoint Exchange.

---

## Table of Contents

- [Project Overview](#project-overview)
- [Technology Stack](#technology-stack)
- [Prerequisites](#prerequisites)
- [Project Structure](#project-structure)
- [API Endpoints](#api-endpoints)
- [Request & Response Examples](#request--response-examples)
- [Error Handling](#error-handling)
- [Running Locally](#running-locally)
- [Building the Project](#building-the-project)
- [Related Projects](#related-projects)

---

## Project Overview

| Property | Value |
|---|---|
| **Artifact ID** | `employee-details-api-impl` |
| **Group ID** | `com.mycompany` |
| **Version** | `1.0.0-SNAPSHOT` |
| **Mule Runtime** | `4.11.2` (minimum `4.11.0`) |
| **Java Version** | `17` |
| **API Spec** | `employee-details-api` v`1.0.1` (RAML 1.0) |
| **Data Store** | FTP — `employees.json` |
| **Base Path** | `/api/*` |
| **Console Path** | `/console/*` |

---

## Technology Stack

| Component | Version |
|---|---|
| Mule Runtime | 4.11.2 |
| APIKit Module | 1.11.17 |
| HTTP Connector | 1.11.3 |
| FTP Connector | 1.8.8 |
| DataWeave | 2.0 |
| RAML | 1.0 |
| Java | 17 |

---

## Prerequisites

- **Anypoint Code Builder** (VS Code extension) or **Anypoint Studio**
- **Java JDK 17**
- **Maven 3.8+**
- **FTP Server** running locally (or remotely) with a file `employees.json` in the configured working directory
- Access to **Anypoint Exchange** (to resolve the RAML dependency)

### FTP Setup

Create an `employees.json` file on your FTP server with the following initial content:

```json
[
  {
    "id": 1001,
    "name": "Alice Johnson",
    "salary": 75000,
    "location": "New York"
  }
]
```

---

## Project Structure

```
employee-details-api-impl/
├── src/
│   ├── main/
│   │   ├── mule/
│   │   │   ├── main.xml                # HTTP Listener + APIKit Router + Console flows
│   │   │   ├── implementation.xml      # GET, POST, PUT, DELETE flow implementations
│   │   │   ├── globalConfig.xml        # HTTP, FTP, APIKit global configurations
│   │   │   └── globalErrorHandler.xml  # Centralized error handling (400–503)
│   │   └── resources/
│   │       ├── config.yaml             # Environment-specific properties
│   │       └── log4j2.xml              # Logging configuration
│   └── test/
│       ├── munit/                      # MUnit test suites
│       └── resources/
│           └── log4j2-test.xml
├── pom.xml                             # Maven project descriptor
├── mule-artifact.json                  # Mule application metadata
├── PROMPTS.md                          # AI-assisted development prompt history
└── README.md                           # This file
```
---

## API Endpoints

Base URL: `http://localhost:8081/api`  
API Console: `http://localhost:8081/console`

### `GET /employees`

Retrieve all employees. Optionally filter by a single employee using the `id` query parameter.

| Parameter | Type | Required | Description |
|---|---|---|---|
| `id` | `number` | No | Filter to return the employee matching this ID |

**Responses:**
- `200 OK` — Returns employee array (or single employee object when `id` is provided)
- `404 Not Found` — When `id` is provided but no matching employee exists

---

### `POST /employees`

Create a new employee record.

**Request Body** (`application/json`):

| Field | Type | Required | Description |
|---|---|---|---|
| `id` | `number` | Yes | Unique employee identifier |
| `name` | `string` | Yes | Full name of the employee |
| `salary` | `number` | Yes | Employee salary |
| `location` | `string` | Yes | Employee work location |

**Responses:**
- `201 Created` — Employee created; returns `{ id, name, status: "Created" }`
- `409 Conflict` — Employee with the same `id` already exists

---

### `PUT /employees`

Update an existing employee's fields. Only the provided fields are updated.

**Request Body** (`application/json`):

| Field | Type | Required | Description |
|---|---|---|---|
| `id` | `number` | Yes | ID of the employee to update |
| `name` | `string` | No | New name |
| `salary` | `number` | No | New salary |
| `location` | `string` | No | New location |

**Responses:**
- `204 No Content` — Employee updated successfully; empty response body
- `400 Bad Request` — Invalid or missing required fields

---

### `DELETE /employees`

Delete an employee record by ID.

| Parameter | Type | Required | Description |
|---|---|---|---|
| `id` | `number` | **Yes** | ID of the employee to delete |

**Responses:**
- `204 No Content` — Employee deleted successfully; empty response body
- `404 Not Found` — No employee with the given `id` exists
- `400 Bad Request` — `id` query parameter is missing

---

## Request & Response Examples

### GET all employees
```http
GET http://localhost:8081/api/employees
```
```json
[
  { "id": 1001, "name": "Alice Johnson", "salary": 75000, "location": "New York" },
  { "id": 1002, "name": "Bob Smith",    "salary": 82000, "location": "Chicago"  }
]
```

### GET employee by ID
```http
GET http://localhost:8081/api/employees?id=1001
```
```json
{ "id": 1001, "name": "Alice Johnson", "salary": 75000, "location": "New York" }
```

### POST — Create employee
```http
POST http://localhost:8081/api/employees
Content-Type: application/json

{ "id": 1003, "name": "Carol White", "salary": 68000, "location": "Austin" }
```
```json
{ "id": 1003, "name": "Carol White", "status": "Created" }
```

### PUT — Update employee
```http
PUT http://localhost:8081/api/employees
Content-Type: application/json

{ "id": 1001, "salary": 80000, "location": "San Francisco" }
```
_(HTTP 204 — empty body)_

### DELETE — Delete employee
```http
DELETE http://localhost:8081/api/employees?id=1003
```
_(HTTP 204 — empty body)_

---

## Error Handling

All errors return a consistent JSON body:

```json
{
  "code": 404,
  "message": "Not Found",
  "description": "Employee not found",
  "timestamp": "2026-07-14T01:00:00Z"
}
```

The global error handler (`globalErrorHandler.xml`) covers:

| HTTP Status | Error Type(s) | Scenario |
|---|---|---|
| `400` | `APIKIT:BAD_REQUEST`, `HTTP:BAD_REQUEST`, `FTP:ILLEGAL_PATH` | Malformed request / invalid FTP path |
| `401` | `HTTP:UNAUTHORIZED` | Authentication required |
| `403` | `HTTP:FORBIDDEN` | Access denied |
| `404` | `APIKIT:NOT_FOUND`, `APP:NOT_FOUND`, `HTTP:NOT_FOUND`, `FTP:FILE_DOESNT_EXIST` | Resource not found |
| `405` | `APIKIT:METHOD_NOT_ALLOWED` | HTTP method not supported |
| `406` | `APIKIT:NOT_ACCEPTABLE` | Unsupported `Accept` media type |
| `408` | `HTTP:TIMEOUT` | Request timeout |
| `409` | `APP:CONFLICT` | Duplicate employee ID on POST |
| `415` | `APIKIT:UNSUPPORTED_MEDIA_TYPE` | Unsupported `Content-Type` |
| `500` | `ANY` | Catch-all for unexpected errors |
| `503` | `HTTP:CONNECTIVITY`, `FTP:CONNECTIVITY`, `FTP:RETRY_EXHAUSTED` | Service/FTP unavailable |


---

## Running Locally

### Using Anypoint Code Builder (VS Code)

1. Open the project in VS Code with the Anypoint Code Builder extension installed.
2. Verify your `config.yaml` values point to a running FTP server.
3. Use the **Run** button or the MuleSoft Developer Agent to start the app locally.
4. Access the API Console at: `http://localhost:8081/console`

### Using Maven

```bash
mvn clean package
```

This resolves the RAML dependency from Anypoint Exchange and packages the application.

---

## Building the Project

```bash
# Clean and package (produces deployable .jar in target/)
mvn clean package -DskipTests

# Run with tests
mvn clean verify
```

> **Note:** Ensure your Maven `settings.xml` includes credentials for `anypoint-exchange-v3`  
> (`https://maven.anypoint.mulesoft.com/api/v3/maven`) to resolve the RAML dependency.

---

*For MuleSoft documentation, visit [https://docs.mulesoft.com/general/](https://docs.mulesoft.com/general/)*