# Test Cases — Restful-Booker REST API
**Document ID:** TC-RB-001  
**Version:** 1.0  
**Linked Plan:** TP-RB-001

---

## Part A — Core Flow (Happy Path)

| TC_ID | Endpoint | Scenario | Method | Execution Type | Expected Status | Assertions |
|-------|----------|----------|--------|----------------|-----------------|------------|
| TC-01 | `/auth` | Generate auth token with valid credentials | POST | Manual | 200 | • Body contains `token` key <br>• `token` value is non-empty string <br>• Save token to env variable `authToken` |
| TC-02 | `/booking` | Create a new booking with all valid fields | POST | Manual | 200 | • Body contains `bookingid` (integer) <br>• `booking.firstname` matches request <br>• `booking.totalprice` matches request <br>• Save `bookingid` to env variable `newBookingId` |
| TC-03 | `/booking` | Get list of all bookings | GET | Manual | 200 | • Response is a JSON array <br>• Array contains at least 1 object <br>• Each object has `bookingid` key |
| TC-04 | `/booking/{{newBookingId}}` | Get specific booking by ID just created | GET | Manual | 200 | • `firstname` matches TC-02 request value <br>• `lastname` matches TC-02 request value <br>• `checkin` and `checkout` match TC-02 values |
| TC-05 | `/booking/{{newBookingId}}` | Full update (PUT) of existing booking | PUT | Manual | 200 | • All fields in response match the new PUT payload <br>• `totalprice` reflects updated value |
| TC-06 | `/booking/{{newBookingId}}` | Partial update (PATCH) of `firstname` only | PATCH | Manual | 200 | • `firstname` in response matches patched value <br>• All other fields remain unchanged |
| TC-07 | `/booking/{{newBookingId}}` | Delete booking with valid token | DELETE | Manual | 201 | • Status code is `201` (known API deviation from REST standard `204`) <br>• Subsequent `GET /booking/{{newBookingId}}` returns `404` |
| TC-08 | `/booking/{{newBookingId}}` | Confirm booking no longer exists after delete | GET | Manual | 404 | • Status code is `404` <br>• Confirms full delete lifecycle |

---

## Part B — Data-Driven / Negative / Boundary Cases

| TC_ID | Endpoint | Scenario | Method | Execution Type | Expected Status | Assertions |
|-------|----------|----------|--------|----------------|-----------------|------------|
| TC-09 | `/auth` | Login with wrong password | POST | Data-Driven | 200 | • Body contains `"reason": "Bad credentials"` <br>• No `token` key present |
| TC-10 | `/auth` | Login with empty username | POST | Data-Driven | 200 | • Body contains `"reason": "Bad credentials"` |
| TC-11 | `/auth` | Login with missing `username` field entirely | POST | Data-Driven | 200 | • Body contains `"reason": "Bad credentials"` |
| TC-12 | `/booking` | Create booking with missing `firstname` | POST | Data-Driven | 500 | • Status code is `500` <br>• Document actual response body |
| TC-13 | `/booking` | Create booking with missing `bookingdates` object | POST | Data-Driven | 500 | • Status code is `500` |
| TC-14 | `/booking` | Create booking with `totalprice` as a string (`"abc"`) | POST | Data-Driven | 200 | • Document whether API accepts or rejects; log as defect if accepted without error |
| TC-15 | `/booking` | Create booking with `checkin` date after `checkout` date | POST | Data-Driven | 200 or 400 | • If `200`: document as validation defect (no server-side date logic) <br>• If `400`: assert error message |
| TC-16 | `/booking` | Create booking with invalid date format (`31-05-2026`) | POST | Data-Driven | 400 or 500 | • Status code indicates failure <br>• Document actual behavior vs expected |
| TC-17 | `/booking` | Create booking with `totalprice: 0` (boundary – zero) | POST | Data-Driven | 200 | • `totalprice` value is `0` in response |
| TC-18 | `/booking` | Create booking with very large `totalprice` (999999999) | POST | Data-Driven | 200 | • Value stored and returned correctly |
| TC-19 | `/booking/{{newBookingId}}` | Update booking without auth token (no Cookie header) | PUT | Manual | 403 | • Status code is `403` |
| TC-20 | `/booking/{{newBookingId}}` | Delete booking with invalid/expired token | DELETE | Manual | 403 | • Status code is `403` |
| TC-21 | `/booking/99999999` | Get a booking with non-existent ID | GET | Manual | 404 | • Status code is `404` |
| TC-22 | `/booking/abc` | Get a booking with non-numeric ID | GET | Manual | 404 | • Status code is `404` or `400`; document actual |
| TC-23 | `/booking` | Create booking with empty string `firstname: ""` | POST | Data-Driven | 200 or 400 | • Document whether empty strings are accepted; log defect if no validation |
| TC-24 | `/booking/{{newBookingId}}` | PATCH without `Content-Type: application/json` header | PATCH | Manual | 400 | • Status code is `400` or `500`; document actual |