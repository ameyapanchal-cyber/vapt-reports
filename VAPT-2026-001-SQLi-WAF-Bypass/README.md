# VAPT-2026-001 — SQL Injection with WAF Bypass

| Field | Value |
|-------|-------|
| **Severity** | 🔴 CRITICAL |
| **CVSS v3.1** | 9.1 — AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:N |
| **CWE** | CWE-89: SQL Injection |
| **OWASP** | A03:2021 – Injection |
| **Target** | PortSwigger Web Security Academy (Authorized Lab) |
| **Testing Date** | 2026-08-01 |

## Summary

A SQL injection vulnerability in the XML-based stock-check endpoint allowed complete disclosure of all user credentials, including the administrator account. A WAF was deployed to block SQL keyword patterns; it was bypassed by encoding the payload as XML numeric character references (`&#xHH;`), exploiting a parser differential between the WAF (raw bytes) and the XML parser (decodes entities first).

**Impact:** Full credential dump → administrator account takeover. WAF provides no protection.

**Fix:** Parameterized queries (prepared statements). The WAF bypass is a secondary issue; the root cause is string concatenation of untrusted input into SQL.

## Report

📄 [VAPT_Report_VAPT-2026-001_SQLi_WAF_Bypass.docx](VAPT_Report_VAPT-2026-001_SQLi_WAF_Bypass.docx)

## Key Techniques

- SQL injection via XML body (`Content-Type: application/xml`)
- WAF bypass via XML numeric character entity encoding (`&#xHH;`)
- Parser differential exploitation (WAF vs XML parser)
- UNION-based credential extraction with `||` concatenation

## Screenshots

*Add lab solved banner and Burp Repeater screenshots to the `screenshots/` folder.*
