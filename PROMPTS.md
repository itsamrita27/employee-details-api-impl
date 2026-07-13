# Employee Details API — Prompts Used

This file documents all the AI-assisted prompts used to design, build, and evolve the
`employee-details-api` (RAML spec) and `employee-details-api-impl` (MuleSoft implementation)
projects using Anypoint Code Builder and the MuleSoft Developer Agent.

---

## Project Overview

| Item | Value |
|---|---|
| API Spec Project | `c:\mulesoftAI-WS\employee-details-api` |
| Impl Project | `c:\mulesoftAI-WS\employee-details-api-impl` |
| Data Store | FTP (`employees.json`) |
| Runtime | Mule 4.11.2 |
| RAML Version | 1.0 |
| Exchange Asset | `employee-details-api` (org: `ff280dfa-308f-41a2-a85c-60ed5498fe4e`) |

---

## Phase 1 — API Specification Design

### Prompt 1 — Create RAML API Spec Project

```
Create a RAML 1.0 API specification for an Employee Details API.
The API should manage employee records with the following fields:
  - id (number)
  - name (string)
  - salary (number)
  - location (string)

Include:
- GET /employees — retrieve all employees, with an optional id query parameter to filter by a single employee
- POST /employees — create a new employee (request body: id, name, salary, location), response 201
- PUT /employees — update an existing employee (request body: id + fields to update), response 204

Use RAML data types, examples, and a reusable error-responses trait (400, 404, 409, 500).
Organize into datatype/, example/, and trait/ folders.
```

**Files produced:**
- `employee-details-api.raml`
- `datatype/employee.raml`
- `datatype/employee-list-response.raml`
- `datatype/employee-create-request.raml`
- `datatype/employee-create-response.raml`
- `datatype/employee-update-request.raml`
- `datatype/error-response.raml`
- `trait/error-trait.raml`
- `example/employee-list-response-example.json`
- `example/employee-get-by-id-response-example.json`
- `example/employee-create-request-example.json`
- `example/employee-create-response-example.json`
- `example/employee-update-request-example.json`
- `example/error-400-example.json`
- `example/error-404-example.json`
- `example/error-409-example.json`
- `example/error-500-example.json`

---

### Prompt 2 — Publish RAML Spec to Anypoint Exchange

```
Publish the Employee Details API RAML spec to Anypoint Exchange as version 1.0.0.
```

**Result:** Asset `employee-details-api` published at version `1.0.0` under org `ff280dfa-308f-41a2-a85c-60ed5498fe4e`.

---

## Phase 2 — Implementation Project Setup

### Prompt 3 — Scaffold Mule Implementation from Exchange API Spec

```
Implement the Employee Details API spec from Anypoint Exchange.
Create a new Mule 4 implementation project called employee-details-api-impl
that scaffolds APIKit flows from the published RAML spec.
```

**Files produced:**
- `pom.xml` (with `employee-details-api:1.0.0:raml` dependency)
- `src/main/mule/main.xml` (APIKit main flow + console flow)
- `src/main/mule/globalConfig.xml` (HTTP, FTP, APIKit configs)
- `src/main/resources/config.yaml`
- `mule-artifact.json`

---

### Prompt 4 — Configure Global Connections and Properties

```
Configure the following global elements in the Mule implementation project:
- HTTP Listener on host 0.0.0.0, port 8081 (from config.yaml property placeholders)
- FTP Connector with host, port, username, password, workingDir from config.yaml
- APIKit Router config pointing to the published RAML spec in Exchange
- A default global error handler reference

Use property placeholders for all environment-specific values in config.yaml.
Never hardcode credentials.
```

**Files modified:**
- `src/main/mule/globalConfig.xml`
- `src/main/resources/config.yaml`

---

## Phase 3 — Implementation Flows

### Prompt 5 — Implement GET /employees Flow

```
Implement the GET /employees flow in the Mule implementation project.
- Read employee data from an FTP file (employees.json)
- Support an optional id query parameter to filter by a single employee
- If id is provided and no match is found, return 404 (APP:NOT_FOUND)
- Return the full employee array when no id filter is given
- Use DataWeave 2.0 for the transformation
- Add INFO-level logging for request and response
- Reference the global error handler
Follow MuleSoft best practices.
```

**Flow added:** `get:\employees:employee-details-api-config`

---

### Prompt 6 — Implement POST /employees Flow

```
Implement the POST /employees flow in the Mule implementation project.
- Accept a JSON request body with id, name, salary, location
- Read the existing employees from the FTP data file
- Check for duplicate id — if exists, raise APP:CONFLICT (409)
- Append the new employee to the array and write back to FTP (OVERWRITE mode)
- Return 201 Created with id, name, status='Created' in the response body
- Add INFO-level logging for request and success
- Reference the global error handler
Follow MuleSoft best practices.
```

**Flow added:** `post:\employees:application\json:employee-details-api-config`

---

### Prompt 7 — Implement PUT /employees Flow

```
Implement the PUT /employees flow in the Mule implementation project.
- Accept a JSON body with id and optional fields (name, salary, location) to update
- Read the existing employees from the FTP data file
- Find the employee by id and merge only the provided fields (no duplicate keys)
- Write the updated list back to FTP (OVERWRITE mode)
- Return 204 No Content with an empty body
- Add INFO-level logging for request and success
- Reference the global error handler
Follow MuleSoft best practices.
```

**Flow added:** `put:\employees:application\json:employee-details-api-config`

---

### Prompt 8 — Add Global Error Handler

```
Create a comprehensive global error handler for the Employee Details API implementation.
Handle the following error types with appropriate HTTP status codes and JSON error bodies
(fields: code, message, description, timestamp):
- APIKIT:BAD_REQUEST → 400
- APIKIT:NOT_FOUND → 404
- APIKIT:METHOD_NOT_ALLOWED → 405
- APIKIT:NOT_ACCEPTABLE → 406
- APIKIT:UNSUPPORTED_MEDIA_TYPE → 415
- HTTP:BAD_REQUEST → 400
- HTTP:UNAUTHORIZED → 401
- HTTP:FORBIDDEN → 403
- HTTP:NOT_FOUND → 404
- HTTP:TIMEOUT → 408
- HTTP:CONNECTIVITY → 503
- FTP:FILE_DOESNT_EXIST → 404
- FTP:ILLEGAL_PATH → 400
- FTP:CONNECTIVITY → 503
- FTP:RETRY_EXHAUSTED → 503
- APP:NOT_FOUND → 404
- APP:CONFLICT → 409
- ANY → 500 (catch-all)
Set the httpStatus variable for each handler.
```

**File produced:** `src/main/mule/globalErrorHandler.xml`

---

## Phase 4 — DELETE Method Addition

### Prompt 9 — Add DELETE Method to API Spec and Implementation

```
Add Delete method in api spec and queryParam is id and response body should be empty
with 204 status code. also do implementation in impl api with mulesoft best practices
```

**Changes made:**

**API Spec (`employee-details-api.raml`):**
- Added `delete:` method to `/employees`
- Query parameter: `id` (number, required: true)
- Applied `ErrorLib.error-responses` trait
- Response: 204 No Content

**Exchange (`exchange.json`):**
- Version bumped: `1.0.0` → `1.0.1`

**Published to Exchange:**
- Asset `employee-details-api` v`1.0.1` published successfully

**Impl project (`pom.xml`):**
- RAML dependency updated: `1.0.0` → `1.0.1`

**Impl project (`globalConfig.xml`):**
- APIKit config `api` reference updated to `employee-details-api:1.0.1:raml`

**Implementation (`implementation.xml`):**
- Added flow `delete:\employees:employee-details-api-config`
- Logic: Log → Store `employeeId` variable → FTP Read → Validate exists (APP:NOT_FOUND if missing) → DataWeave filter out record → FTP Write (OVERWRITE) → Set httpStatus=204 → Set null payload → Log success

---

### Prompt 10 — Generate Prompts Documentation File

```
provide all the prompts used in this project in a file
```

**File produced:** `PROMPTS.md` (this file)

---

## API Endpoint Summary

| Method | Path | Query Param | Request Body | Response | Notes |
|---|---|---|---|---|---|
| `GET` | `/api/employees` | `id` (number, optional) | — | `200` Employee array or single Employee | 404 if id given but not found |
| `POST` | `/api/employees` | — | `EmployeeCreateRequest` | `201` Created confirmation | 409 if duplicate id |
| `PUT` | `/api/employees` | — | `EmployeeUpdateRequest` | `204` No Content | Field-level merge |
| `DELETE` | `/api/employees` | `id` (number, **required**) | — | `204` No Content | 404 if employee not found |

---

## DataWeave Transformations Used

| Flow | Purpose | Key DW Operation |
|---|---|---|
| GET | Filter by id or return all | `filter`, conditional `if/else`, array index `[0]` |
| POST | Append new record to array | `++` array concatenation |
| PUT | Field-level in-place merge | `map` with `default` operator |
| DELETE | Remove record from array | `filter` with negated condition |

---

## Error Codes Reference

| Code | Type | Trigger |
|---|---|---|
| 400 | `APIKIT:BAD_REQUEST` | Missing/invalid request fields |
| 404 | `APP:NOT_FOUND` | Employee id not found |
| 405 | `APIKIT:METHOD_NOT_ALLOWED` | Unsupported HTTP method |
| 409 | `APP:CONFLICT` | Duplicate employee id on POST |
| 503 | `FTP:CONNECTIVITY` | FTP server unreachable |
| 500 | `ANY` | Unhandled/unexpected errors |

---

## Technology Stack

| Component | Version |
|---|---|
| Mule Runtime | 4.11.2 |
| APIKit Module | 1.11.17 |
| HTTP Connector | 1.11.3 |
| FTP Connector | 1.8.8 |
| RAML | 1.0 |
| DataWeave | 2.0 |
| Java | 17 |

---

*Reference: [MuleSoft Documentation](https://docs.mulesoft.com/general/)*