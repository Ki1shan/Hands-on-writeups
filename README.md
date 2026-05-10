# 🚨 Zorvyn Internship Scam — Technical Investigation Report

![Type](https://img.shields.io/badge/type-investigation-red)
![Status](https://img.shields.io/badge/status-exposed-critical)
![Method](https://img.shields.io/badge/method-web%20analysis-orange)
![Verdict](https://img.shields.io/badge/verdict-SCAM%20CONFIRMED-red)

> A step-by-step technical investigation exposing a fake cybersecurity internship scam — using web analysis, parameter tampering, and backend verification to confirm fraud.

---

## ⚠️ Disclaimer

This report is published for **educational awareness and public warning** only. No harmful or unauthorized actions were performed during this investigation. All findings were obtained through passive observation and legitimate parameter testing on systems presented to me as an internee.

---

## Overview

What initially appeared to be a legitimate Cybersecurity Analyst internship at **"Zorvyn FinTech Pvt. Ltd."** was identified through systematic technical analysis as a well-structured internship scam.

The scam combines:
- Social engineering (fake offer letters, manager communication, dashboards)
- Client-side web manipulation (no real backend)
- A reimbursement-based payment trap targeting students

This repository contains the full investigation — email trails, screenshots, and technical analysis — to help others recognize and avoid similar scams.

---

## Evidence Files

| File | Description |
|------|-------------|
| `Your Offer Letter...pdf` | Fake offer letter (₹45,000/month salary promised) |
| `Welcome to the Team...pdf` | Onboarding email establishing false legitimacy |
| `Please Complete Your Welcome Kit Form...pdf` | Data collection under fake onboarding |
| `Your Project Overview and Training Task...pdf` | Task assignment leading to software purchase |
| `Follow Up on Pending Task and Software Reimbursement.pdf` | Reimbursement promise to justify payment |
| `Your Welcome Kit Has Been Ordered...pdf` | Fake logistics to build trust |
| `Your Welcome Kit is Packed and Ready to Ship...pdf` | Continued fake delivery simulation |
| `Your Signed Offer Letter Submission is Pending.pdf` | Pressure tactic for personal data |
| Screenshots | Platform analysis — URL tampering, localhost exposure, fake modules |

---

## Investigation Methodology

Rather than blindly trusting the platform, I approached it as a penetration tester:

- Observed system behavior and UI patterns
- Analyzed URL structures and parameter handling
- Tested parameter manipulation and access control
- Verified backend presence through behavior analysis
- Cross-checked email workflow and domain history

---

## Technical Findings

### Finding 1 — Reimbursement-Based Payment Trap
The company required interns to purchase software upfront and promised reimbursement through their portal.

**Red flag:** Legitimate companies never require interns to make upfront purchases. Reimbursement promises with no paper trail or official policy documentation are a classic social engineering hook.

---

### Finding 2 — URL-Based Identity Injection (Broken Access Control)

The reimbursement portal passed full user identity through URL parameters:

```
/reimbursements/approved?full_name=KISHAN&email=kishan@gmail.com&employee_id=ZFT-1234&software=...
```

**Test performed:** Modified `full_name`, `email`, and `employee_id` values directly in the URL.

**Result:** Changes reflected instantly on the page — name, email, and amount all updated.

**Impact:**
- Zero backend validation
- Fully client-side controlled system
- Any user can impersonate any identity
- Broken Access Control (OWASP A01)

---

### Finding 3 — Localhost URL Exposure

The platform leaked internal development URLs:

```
http://localhost:3003/...
```

**Impact:**
- Development environment not separated from what users see
- No proper production deployment exists
- Strong indicator of a fake or unfinished backend

---

### Finding 4 — No Real Backend (SQL Injection Null Result)

Standard SQL injection probes returned no errors and no unexpected behavior — not because of security, but because:

> **There is no database to inject into.**

The system is entirely client-side simulated. This confirms the absence of any real backend infrastructure.

---

### Finding 5 — Fake Dashboard Modules

| Module | Claimed Function | Reality |
|--------|-----------------|---------|
| Logistics | "Welcome kit shipped" | No tracking ID, no courier integration |
| Webmail | Company inbox | "Activation in 7–8 days" — never activates |
| Referrals | Referral system | Locked, depends on fake webmail |
| Reimbursement | Payment portal | Client-side only, no real payment |

Each module exists purely to build visual legitimacy — none have functional backends.

---

### Finding 6 — Domain Rotation

The platform shifted domains mid-engagement:

```
Original: zorvyn.org / zorvyn.co
Shifted:  zorvyn.live
```

**Significance:** Domain switching mid-operation is a common technique used by scam operators to evade detection, reset reputation, and avoid being flagged by search engines or fraud databases.

---

### Finding 7 — Psychological Manipulation Sequence

The scam follows a deliberate trust-building sequence before triggering the payment:

```
Step 1: Offer letter sent (₹45,000 salary)
Step 2: Welcome email + manager introduction
Step 3: Dashboard access granted
Step 4: Welcome kit "ordered" and "shipped"
Step 5: Task assigned (software required)
Step 6: Software purchase requested
Step 7: Reimbursement promised via fake portal
Step 8: No payment made
```

Each step increases psychological investment before the financial trap is triggered.

---

## Technical Summary

| Component | Claimed | Reality |
|-----------|---------|---------|
| Authentication | Secure login | Weak / bypassable |
| Backend | Full web platform | Not present — client-side simulation |
| Data Handling | Secure storage | URL parameters — no server validation |
| Database | User records | Does not exist |
| Logistics | Real courier | Fake tracking, no integration |
| Webmail | Company email | Never activates |
| Infrastructure | Production platform | Localhost leaked, unfinished |
| Security | Enterprise-grade | Non-existent |

---

## Impact Assessment

**Severity: CRITICAL**

- **Financial** — Students pay for software with no reimbursement
- **Data** — Personal data (name, email, address, bank details) collected via fake forms
- **Psychological** — Prolonged manipulation creates emotional investment before the trap triggers
- **Reputational** — Fake employment records created in students' names

---

## Verdict

```
⚠️  CONFIRMED INTERNSHIP SCAM

This is not a legitimate company or internship program.

Zorvyn FinTech Pvt. Ltd. operates a fully structured
social engineering + payment exploitation scheme
targeting cybersecurity students and job seekers.
```

---

## Red Flags to Watch For

If you encounter a similar internship offer, watch for:

- Reimbursement required for any software or equipment
- Offer letter with high salary but vague job scope
- Portal where URL parameters control your displayed data
- Webmail "activating in 7-8 days" that never activates
- Welcome kit with no real tracking information
- Domain that changes during your engagement
- Manager communicates only via email, never video call

---

## Key Learnings

- Always verify company registration and physical address independently
- Test reimbursement portals — legitimate ones don't put your data in URLs
- Localhost URLs leaking into production is an immediate red flag
- A polished UI means nothing without a real backend
- Social engineering attacks work by building trust over time before striking

---

## Author

**Kishan N**
Offensive Security Engineer | Cybersecurity Researcher

Investigated this scam using penetration testing methodology after recognizing suspicious patterns in the platform's behavior. Published publicly to protect other students from the same trap.

---

*Sometimes the most valuable security lessons don't come from a lab — they come from someone trying to scam you.*
