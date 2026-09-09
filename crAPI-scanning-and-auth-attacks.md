# 01 — Scanning APIs for Vulnerabilities

## 1. Nikto

```bash
nikto -h http://127.0.0.1:8888
```

## 2. OWASP ZAP

**2.1 — Automated Scan**
Run ZAP's automated scanner against:
```
http://127.0.0.1:8888
```

**2.2 — Manual Explore**
Also walk through the app manually inside ZAP (same coverage as the automated crawl) to catch anything the automated spider misses.

---

# 02 — Classic Authentication Attacks

1. Stop **mitmweb**.
2. In **Postman**, go to proxy settings and set the port to `8080`.
3. Open **Burp Suite** → Proxy settings → set Burp's listener to the **same port** (`8080`) so Postman's traffic routes through Burp.
4. Using the Postman collection/documentation built earlier, send a request through Postman (start with the **login** request) and capture it in Burp.
5. Send the captured login request to **Repeater**.
6. Resend it a few times to check for rate limiting or watch for changes in the response code.
7. If nothing changes (no rate limiting) → move to **brute force**.
8. Send the request to Repeater, then brute-force it with **wfuzz**:

```bash
wfuzz -d '{"email":"test1@email.com","password":"FUZZ"}' \
  -H 'Content-Type: application/json' \
  -z file,rockyou.txt \
  -u http://127.0.0.1:8888/identity/api/auth/login
```

---

# 03 — BOLA (Broken Object Level Authorization)

**Concept:** User A can view information belonging to User B by manipulating an object reference. (This overlaps with **IDOR** — same root cause, viewed through the OWASP API Top 10 naming.)

**Test:** Grab a user ID from a community post's response, then send that ID in a different request to the **identity** API and see if it returns that user's private data.

---

# 04 — BFLA (Broken Function Level Authorization)

**Test 1:** In the collection's **order** section, change the order ID (e.g. from `10`) to a different value and see if you can act on an order that isn't yours.

**Test 2:** On an uploaded video's rename endpoint, change the method from `PUT` to `DELETE` and try to delete the video — while authenticated as a regular user, not admin — to see if the privilege check is missing at the function level.

---

# 05 — Advanced Vulnerability Exploitation

**Postman Runner (Collection Runner):**
Use the Collection Runner to check API behavior across versions — there's an option to run against a single request or the **entire collection** at once, which is a fast way to sweep every endpoint for a given check.

**Steps:**
1. After identifying `v3` endpoints via `mitmproxy2swagger`, go to the **forgot password** flow.
2. Target the **OTP check** endpoint and brute-force the OTP using wfuzz:

```bash
wfuzz -d '{"email":"test@test.com", "otp":"FUZZ", "password":"NewPassword@123"}' \
  -H 'Content-Type: application/json' \
  -z file,/usr/share/seclists/Fuzzing/4-digits-0000-9999.txt \
  -u http://127.0.0.1:8888/identity/api/auth/v2/check-otp \
  --hc 500
```

`--hc 500` hides responses with HTTP 500 status, so only valid/interesting responses (successful OTP guesses) remain visible in the output.
