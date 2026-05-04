# API Security Risk Analysis Report

**Date:** 2026-05-04


---

## Overview

This report presents the results of an API Security Risk Analysis conducted using Postman. The analysis evaluates two public APIs - ReqRes and JSONPlaceholder - for common security vulnerabilities including authentication weaknesses, open access risks, data exposure, and input validation issues. The goal is to identify potential security risks and provide actionable recommendations.

- Total Tests Run: *30*
- Tests Passed: **9**
- Tests Failed: **1** (List Posts - X-Powered-By header exposure)


---

## ReqRes API Tests

### 1. List Users (Auth Check)
- **Method:** GET
- **URL:** `https://reqres.in/api/users`
- **Security Risk Tested:** Accessibility without authentication
- **Finding:** Endpoint requires `x-api-key` header; returns 401 without it. Authentication is properly enforced.

### 2. Get Single User
- **Method:** GET
- **URL:** `https://reqres.in/api/users/1`
- **Security Risk Tested:** Data exposure for individual user records
- **Finding:** Returns full user PII (name, email, avatar) with valid API key. PII exposure should be minimized.

### 3. Access Non-Existent User (IDOR Check)
- **Method:** GET
- **URL:** `https://reqres.in/api/users/9999`
- **Security Risk Tested:** Insecure Direct Object Reference (IDOR) vulnerabilities
- **Finding:** Returns 404 properly; no data leakage. IDOR protection is adequate.

### 4. Login - Weak Auth Check
- **Method:** POST
- **URL:** `https://reqres.in/api/login`
- **Security Risk Tested:** Weak authentication mechanisms
- **Finding:** Accepts simple email/password with no rate limiting observed. Brute-force attacks are possible.

### 5. Login with Missing Password (Input Validation)
- **Method:** POST
- **URL:** `https://reqres.in/api/login`
- **Security Risk Tested:** Input validation on auth endpoints
- **Finding:** Returns 400 with descriptive error "Missing password" - information disclosure risk.


---

## JSONPlaceholder API Tests

### 1. List Posts (Open Endpoint Check)
- **Method:** GET
- **URL:** `https://jsonplaceholder.typicode.com/posts`
- **Security Risk Tested:** Open endpoint without authentication
- **Finding:** FAILED — `X-Powered-By` header exposes server technology (Express). Technology fingerprinting risk.

### 2. Get Single Post
- **Method:** GET
- **URL:** `https://jsonplaceholder.typicode.com/posts/1`
- **Security Risk Tested:** Data retrieval security
- **Finding:** Passes; data returned without sensitive info.

### 3. Create Post (No Auth Required - Risk)
- **Method:** POST
- **URL:** `https://jsonplaceholder.typicode.com/posts`
- **Security Risk Tested:** Unauthenticated write access
- **Finding:** HIGH RISK — POST accepted without any authentication. Anyone can create resources.

### 4. Delete Resource Without Auth
- **Method:** DELETE
- **URL:** `https://jsonplaceholder.typicode.com/posts/1`
- **Security Risk Tested:** Unauthenticated delete access
- **Finding:** HIGH RISK — DELETE accepted without any authentication. Anyone can delete resources.

### 5. Access Users Data (Data Exposure Check)
- **Method:** GET
- **URL:** `https://jsonplaceholder.typicode.com/users`
- **Security Risk Tested:** PII exposure without authentication
- **Finding:** MEDIUM RISK — Returns names, emails, addresses, phone numbers without auth.


---

## Security Findings Summary

| API | Endpoint | Risk | Severity |
|-----|----------|------|----------|
| JSONPlaceholder | POST /posts | Unauthenticated Write Access | High |
| JSONPlaceholder | DELETE/posts/1 | Unauthenticated Delete Access | High |
| ReqRes | POST /login | No Rate Limiting on Auth | Medium |
| JSONPlaceholder | GET /users | PII Exposure Without Auth | Medium |
| ReqRes | GET /users/1 | PII Returned With API Key | Medium |
| ReqRes | POST /login (invalid) | Descriptive Error Message | Low |
| JSONPlaceholder | GET /posts | X-Powered-By Header Exposure | Low |



---

## Recommendations

1. **Implement Authentication on Write Operations (High Priority):** All POST, PUT, PATCH, and DELETE endpoints should require valid authentication. Unauthenticated write access is a critical security risk.

2. **Enforce Rate Limiting on Auth Endpoints:** Add rate limiting to login and authentication endpoints to prevent brute-force attacks. Consider implementing account lockout policies.

3. **Minimize PII Exposure:** Review all API responses to ensure only necessary data is returned. PII such as emails, addresses, and phone numbers should not be exposed without proper authorization.

4. **Remove Server Technology Headers:** Disable or remove `X-Powered-By` and similar headers that expose server technology stack information, reducing the attack surface.

5. **Generic Error Messages:** Replace descriptive error messages (e.g., "Missing password") with generic ones to prevent information disclosure that could aid attackers.

6. **Implement Authorization Checks:** Ensure that authenticated users can only access and modify their own resources. Implement proper authorization checks to prevent IDOR vulnerabilities.

7. **Use HTTPS Everywhere:** Ensure all API endpoints are accessible only over HTTPS to prevent data interception.

8. **Regular Security Audits:** Conduct regular security audits and penetration testing to identify and address new vulnerabilities as the API evolves.

---

*Report generated using Postman API Security Testing Suite.*
