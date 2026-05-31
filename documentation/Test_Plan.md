# Test Plan — Restful-Booker REST API
**Document ID:** TP-RB-001  
**Version:** 1.0  
**Date:** 2026-05-31  
**Author:** Ahmed Abdelmawgod  
**Standard:** Based on IEEE 829

---

## 1. Introduction & Objectives

### 1.1 Introduction
This test plan covers manual API testing of the **Restful-Booker** public demo API
(https://restful-booker.herokuapp.com). The API simulates a hotel booking system and
exposes endpoints for authentication and full CRUD management of bookings.

### 1.2 Objectives
- Validate all API endpoints return correct HTTP status codes per specification.
- Verify request/response schemas match documented contracts.
- Confirm data integrity: values sent in requests are correctly persisted and
  returned in subsequent reads.
- Identify boundary conditions and negative-path failures.
- Establish a reusable Postman collection for regression use.

---

## 2. Scope

### 2.1 In-Scope
| Area | Endpoints Covered |
|------|-------------------|
| Authentication | `POST /auth` – Create token |
| Booking – Create | `POST /booking` |
| Booking – Read | `GET /booking`, `GET /booking/:id` |
| Booking – Full Update | `PUT /booking/:id` |
| Booking – Partial Update | `PATCH /booking/:id` |
| Booking – Delete | `DELETE /booking/:id` |

### 2.2 Out-of-Scope
- Load / performance testing.
- Security penetration testing (SQL injection, auth bypass at scale).
- UI testing (no frontend is under test).
- SLA / uptime monitoring.

---

## 3. Test Approach

| Type | Method |
|------|--------|
| Functional – Happy Path | Manual execution via Postman |
| Functional – Negative Path | Data-Driven via Postman variables |
| Contract / Schema | JSON schema assertions in Postman Tests tab |
| Data-Driven Boundary | Parametrized Postman requests |

---

## 4. Entry & Exit Criteria

### Entry Criteria
- [ ] Postman installed and environment configured.
- [ ] API reachable (health check: `GET /booking` returns 200).
- [ ] Auth token successfully generated.

### Exit Criteria
- [ ] All P1 (Critical) and P2 (High) test cases executed.
- [ ] No open Critical defects blocking core CRUD flow.
- [ ] Test results documented in Test_Cases.md.

---

## 5. Known Behaviors & Risk Register

| Risk ID | Description | Likelihood | Impact | Mitigation |
|---------|-------------|------------|--------|------------|
| R-01 | **Heroku cold start** – First request after inactivity may time out (30 s) | High | Medium | Send a warm-up `GET /booking` before test suite; retry once on timeout. |
| R-02 | **Non-standard DELETE response** – API returns `201 Created` on successful delete instead of the standard `204 No Content` | High | Low | Assert `201` (not `204`) in delete test cases; document as known deviation. |
| R-03 | **Token expiration** – Auth token expires after ~10 min | Medium | High | Re-authenticate at the start of each test session; store token in env variable. |
| R-04 | **Shared public data** – Other users can modify or delete bookings created during testing | High | Medium | Capture `bookingid` from `POST /booking` and reference it immediately; do not rely on pre-existing IDs. |
| R-05 | **No input validation enforcement** – API accepts some malformed payloads without error | Medium | Medium | Document actual vs. expected behavior; raise as defects if business rules require validation. |
| R-06 | **Date field format sensitivity** – `checkin`/`checkout` must be `YYYY-MM-DD`; other formats may return 500 | Medium | Medium | Include boundary cases for date format in DDT rows. |
| R-07 | **Missing `Content-Type` header** – Omitting `Content-Type: application/json` causes 400 or 500 errors | Medium | Medium | Always include required headers; test omission as a negative case. |

---

## 6. Test Environment

| Item | Value |
|------|-------|
| Base URL | `https://restful-booker.herokuapp.com` |
| Protocol | HTTPS |
| Tool | Postman v10+ |
| Auth Mechanism | Cookie-based token (`Cookie: token=<value>`) |
| Default credentials | `admin` / `password123` |

---

## 7. Deliverables

| Deliverable | Location |
|-------------|----------|
| This Test Plan | `documentation/Test_Plan.md` |
| Test Cases | `documentation/Test_Cases.md` |
| Postman Collection | `postman_artifacts/RestfulBooker-Manual.postman_collection.json` |
| Postman Environment | `postman_artifacts/Restful_Booker_DEV.postman_environment.json` |

