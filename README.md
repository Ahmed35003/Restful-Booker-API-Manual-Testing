# Restful-Booker API Manual Testing

A structured, Postman-based manual API testing suite for the **[Restful-Booker](https://restful-booker.herokuapp.com)** public demo API — a simulated hotel booking system exposing authentication and full CRUD endpoints.

> **Implementation Status:** Core flow (happy path) test cases completed — 8 test cases covering the full booking lifecycle from authentication through deletion.

---

## Table of Contents

- [Overview](#overview)
- [Project Structure](#project-structure)
- [Prerequisites](#prerequisites)
- [Getting Started](#getting-started)
- [Implemented Test Cases](#implemented-test-cases)
- [Running the Tests](#running-the-tests)
- [Known API Behaviors & Quirks](#known-api-behaviors--quirks)
- [Test Environment Details](#test-environment-details)
- [Deliverables](#deliverables)


---

## Overview

This project provides a complete API testing solution for the Restful-Booker demo API. It includes a comprehensive **Test Plan** (IEEE 829-based), a detailed **Test Cases** document covering both happy-path and negative-path scenarios, and a ready-to-use **Postman Collection** with a pre-configured **development environment**.

The currently implemented scope covers the **core CRUD lifecycle** — authenticating against the API, creating a booking, reading it back, performing a full and partial update, deleting it, and confirming deletion. This validates the fundamental contract of every major endpoint in a sequential, dependency-aware flow.

## Project Structure

```
restful-booker-api-testing/
├── .gitignore
├── README.md                              # This file
├── documentation/
│   ├── Test_Plan.md                       # Full test plan (IEEE 829 based)
│   └── Test_Cases.md                      # All test cases (Part A + Part B)
└── postman_artifacts/
    ├── RestfulBooker-Manual.postman_collection.json           # Postman request collection
    └── Restful_Booker_DEV.postman_environment.json      # Dev environment variables
```

| Directory / File | Purpose |
|---|---|
| `documentation/` | Test strategy documents — the test plan defines scope and approach, while the test cases document provides the full spec for every scenario (implemented and planned). |
| `postman_artifacts/` | Import-ready Postman files. The collection contains all requests with pre-written test scripts; the environment file holds base URL, credentials, and dynamic variables (`authToken`, `newBookingId`). |

---

## Prerequisites

| Requirement | Details |
|---|---|
| **Postman** | v10 or later — [Download here](https://www.postman.com/downloads/) |
| **Internet access** | The target API is hosted on Heroku (public, no auth required for read endpoints) |
| **Basic API knowledge** | Familiarity with HTTP methods, status codes, and JSON payloads is assumed |

No coding or command-line skills are required — the entire suite runs inside Postman.

---

## Getting Started

### 1. Clone or download this repository

```bash
git clone <repository-url>
cd restful-booker-api-testing
```

### 2. Import the Postman environment

1. Open Postman.
2. Navigate to **Environments** (gear icon on the left sidebar or the "Environments" tab).
3. Click **Import**, select `postman_artifacts/Restful_Booker_DEV.postman_environment.json`.
4. Set the imported environment as **active** by selecting it from the environment dropdown (top-right).

The environment provides these key variables:

| Variable | Default Value | Description |
|---|---|---|
| `baseUrl` | `https://restful-booker.herokuapp.com` | API base URL |
| `username` | `admin` | Auth username |
| `password` | `password123` | Auth password |
| `authToken` | *(empty — populated at runtime)* | Set automatically by TC-01 after successful authentication |
| `newBookingId` | *(empty — populated at runtime)* | Set automatically by TC-02 after successful booking creation |

### 3. Import the Postman collection

1. In Postman, navigate to **Collections**.
2. Click **Import**, select `postman_artifacts/RestfulBooker-Manual.postman_collection.json`.
3. The collection will appear under the **Collections** panel with all pre-configured requests.

### 4. Verify connectivity

Before running tests, confirm the API is reachable:

1. Open the `TC-03 — Get All Bookings` request in the collection.
2. Click **Send**.
3. Verify you receive a `200 OK` response with a JSON array.

> **Note on Heroku cold starts:** If the first request takes more than 30 seconds or times out, the Heroku dyno may have gone to sleep. Simply retry the request — the second attempt should succeed immediately.

---

## Implemented Test Cases

The following 8 test cases implement the **complete core CRUD lifecycle**. They must be executed **in order** due to runtime dependencies (auth token and booking ID are chained across requests).

### Part A — Core Flow (Happy Path)

| TC ID | Endpoint | Scenario | Method | Key Assertions |
|---|---|---|---|---|
| **TC-01** | `/auth` | Generate auth token with valid credentials | `POST` | Response contains a non-empty `token` field; token is saved to `authToken` environment variable |
| **TC-02** | `/booking` | Create a new booking with all valid fields | `POST` | Response returns a numeric `bookingid`; all submitted fields (`firstname`, `lastname`, `totalprice`, `depositpaid`, `bookingdates`) are echoed correctly; `bookingid` saved to `newBookingId` |
| **TC-03** | `/booking` | Retrieve a list of all bookings | `GET` | Response is a JSON array containing at least one object with a `bookingid` key |
| **TC-04** | `/booking/{{newBookingId}}` | Retrieve the specific booking created in TC-02 | `GET` | All fields match the values submitted during creation (`firstname`, `lastname`, `checkin`, `checkout`, `totalprice`) |
| **TC-05** | `/booking/{{newBookingId}}` | Full update (replace) of the booking | `PUT` | Every field in the response matches the new PUT payload; `totalprice` reflects the updated value |
| **TC-06** | `/booking/{{newBookingId}}` | Partial update — change `firstname` only | `PATCH` | `firstname` is updated to the patched value; all other fields remain unchanged from TC-05 |
| **TC-07** | `/booking/{{newBookingId}}` | Delete the booking with a valid token | `DELETE` | Status code is `201` (API-specific — see [known behaviors](#known-api-behaviors--quirks)); response confirms deletion |
| **TC-08** | `/booking/{{newBookingId}}` | Confirm booking no longer exists after deletion | `GET` | Status code is `404`, confirming the booking has been fully removed |

### Execution Flow Diagram

```
TC-01 (Auth) ──► TC-02 (Create) ──► TC-03 (List All)
                                        │
                                        ▼
                                  TC-04 (Read by ID)
                                        │
                                        ▼
                                  TC-05 (Full Update)
                                        │
                                        ▼
                                  TC-06 (Partial Update)
                                        │
                                        ▼
                                  TC-07 (Delete)
                                        │
                                        ▼
                                  TC-08 (Confirm 404)
```

Each test includes **Postman test scripts** in the `Tests` tab that automate assertions and variable chaining, so running the collection sequentially via the Postman Runner will execute the entire flow without manual intervention.

---

## Running the Tests

### Option A: Run via Postman Collection Runner (Recommended)

1. Ensure the **DEV environment** is active (dropdown in top-right).
2. Click the **"..."** menu next to the **Restful-Booker** collection.
3. Select **Run collection** (or click the **Runner** button).
4. Verify the execution order is **TC-01 through TC-08** (sequential).
5. Click **Run Restful-Booker**.
6. Review the results panel — all 8 tests should show a green pass.

### Option B: Run manually one by one

Execute each request in order (TC-01 → TC-08). After each request, check the **Test Results** section at the bottom of the response pane for pass/fail status. Manual execution is useful for debugging or exploring individual responses.

### Interpreting Results

| Indicator | Meaning |
|---|---|
| ✅ Green checkmark | All assertions passed — endpoint behaves as expected |
| ⚠️ Yellow warning | Test script partially passed; review the response body for details |
| ❌ Red cross | One or more assertions failed — check the response and compare against the expected behavior in `documentation/Test_Cases.md` |

---

## Known API Behaviors & Quirks

The Restful-Booker API has several non-standard behaviors that are important to understand before interpreting test results:

| Behavior | Impact on Testing | Handling |
|---|---|---|
| **Heroku cold starts** | First request after inactivity may time out (up to 30 s) | Send a warm-up `GET /booking` before the test suite; retry once on timeout |
| **DELETE returns `201 Created`** | Contradicts REST convention of `204 No Content` | TC-07 asserts `201` instead of `204` — documented as a known deviation |
| **Token expiration (~10 min)** | Auth token becomes invalid after roughly 10 minutes | Re-run TC-01 at the start of each session; do not reuse stale tokens |
| **Shared public data** | Other users on the internet can modify or delete your bookings | Always capture and use `newBookingId` from the same session; never rely on pre-existing IDs |
| **No strict input validation** | API may accept malformed payloads without error (e.g., string for numeric fields) | These scenarios are documented as potential defects in Part B test cases (not yet implemented) |
| **Date format sensitivity** | `checkin`/`checkout` must be `YYYY-MM-DD`; other formats may cause `500` errors | Always use ISO 8601 date format in request payloads |

For a complete risk register with likelihood and impact assessments, refer to Section 5 of `documentation/Test_Plan.md`.

---

## Test Environment Details

| Item | Value |
|---|---|
| **Base URL** | `https://restful-booker.herokuapp.com` |
| **Protocol** | HTTPS |
| **Test Tool** | Postman v10+ |
| **Authentication** | Cookie-based token (`Cookie: token=<value>`) |
| **Default Credentials** | `admin` / `password123` |
| **Test Standard** | IEEE 829 |

---

## Deliverables

| Document | Location | Status |
|---|---|---|
| Test Plan | `documentation/Test_Plan.md` | Complete — scope, approach, risks, entry/exit criteria |
| Test Cases | `documentation/Test_Cases.md` | Complete — Part A (8 cases, implemented) + Part B (16 cases, planned) |
| Postman Collection | `postman_artifacts/RestfulBooker-Manual.postman_collection.json` | Part A implemented with automated test scripts |
| Postman Environment | `postman_artifacts/Restful_Booker_DEV.postman_environment.json` | Configured with base URL, credentials, and runtime variables |

---


