# VAPT-2026-003 (DRAFT — user submits on Intigriti) — Static client credentials in public bdese bundle

- **Asset:** a Tier-4 program asset + its backing API host (exact hostnames withheld — closed report, details available to the program)
- **Proposed severity:** Informational (low at most — triage decides; see Honest impact note)
- **Class:** Sensitive data exposure in client bundle / hardening

## Summary
The public JavaScript bundle of the BDES back-office SPA ships two static
client-side secrets: an API header key (`x-bdes-api-key`, sent on every API
call) and the AES-256-CBC encrypt/decrypt configuration (`secretKey` /
`secretIv`, REACT_APP_* build values) used to protect user PII fields
(email, password, phone, role) client-side. Full original TypeScript sources
(48 files) are also served via `.map` files.

## Repro (passive, no auth needed)
1. `GET <program-asset>/static/js/main.<hash>.chunk.js`
   → `axios.create({baseURL:"<program-api-host>", ...,
   headers:{"x-bdes-api-key":"<STATIC_VALUE_REDACTED>"}})`
2. Same bundle → `secretKey:"<REDACTED>", secretIv:"<REDACTED>"`,
   `encryptionMethod:"aes-256-cbc"`, `decryptUser()` over
   email/password/phone/role/oldPasswords.
3. `GET <program-asset>/static/js/main.<hash>.chunk.js.map`
   → `sourcesContent` with 48 original `.ts` sources.

## What was tested (negative results included)
- `/` and `/me` with correct key / no key / wrong key → identical responses
  (key gates nothing on reachable unauthenticated routes).
- 6 real backend routes (`/users`, `/user/1`, `/documents`, `/document/1`,
  `/structure`, `/sections`) × key/no-key (12 singles, 1s spacing) →
  ALL identical 401 `No refresh token found` (1122/1261B). Key changes
  nothing without a session on any mapped route.
- TruffleHog filesystem scan of saved bundles: 0 verified / 0 unverified —
  bespoke app key, not a cloud/infra credential.
- 20 conventional singles → uniform Express 404-echo except `/me` (401,
  session-gated). Authenticated-route impact of the key is UNTESTED (no
  back-office account available).
- Full 2.6MB corpus scanned: no AWS keys, tokens, private keys, or
  credentials-in-URL beyond the items above.

## Honest impact note (for triage)
Standalone impact is limited: the key alone unlocks nothing reachable without
a session. Risk is conditional — IF any authenticated route trusts the static
key for authorization, it becomes an auth-bypass enabler; and the embedded
AES config weakens the PII-encryption layer to anyone reading the bundle.
Recommend: server-side session enforcement everywhere, remove static key,
rotate it, stop shipping `.map` sourcesContent to production.

## Remediation
Move all authorization server-side (session-bound), delete the static header
key (rotate immediately), keep encryption keys off the client, disable
production source maps or strip `sourcesContent`.
