# HackerOne submission — copy/paste (you click Send)

**Title:** User enumeration via distinct authenticateV3 error messages (+ password-reset oracle)

**Severity:** Low

**Description:**

The login endpoint discloses whether an email is registered:

1. `POST <program-login-endpoint>`
   Body: `{"email":"<nonexistent>@gmail.com","password":"<anything>", ... (standard login fields)}`
   → `code 110`, "No account found with this email. Please register to continue."

2. Same request with my own registered test account (booking-free, created for this test):
   → `code 116`, "Your password has expired. We've sent you an email with instructions to reset it." — and a genuine reset email arrived at that inbox.

Only these two single requests were sent. No brute force, no чуж accounts, no PII pulled.

**Impact:** An attacker can harvest registered emails for phishing / credential-stuffing lists, and trigger unsolicited reset emails to victims.

**Fix suggestion:** Return an identical generic message and code whether or not the account exists (and equalize timing); rate-limit the endpoint.

**Attachments:** Repeater screenshots (redacted) + inbox screenshot — attached.
