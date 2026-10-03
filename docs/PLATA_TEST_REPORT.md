#  Plata Android App — Exploratory QA Testing Report

> **Author:** Nikita Grachev  
> **Role:** QA Engineer (Mobile / Fullstack)  
> **Target App:** Plata (Banco Plata Mexico) — Android Application  
> **Date:** October 2026  
> **Environment:** Physical Android Device (Cross-border / Roaming Environment)  

---

##  Executive Summary

This report covers a comprehensive **Exploratory Testing** session for the **Plata Android Mobile Application** (Registration, Onboarding, and Authentication flows). Special focus was directed toward edge cases, input sanitization, security mechanisms (OTP Rate Limiting, Brute-Force defense), and legal WebViews.

### Testing Overview Matrix

| Total Scenarios Executed | Passed | UX / UI Observations | Critical / Blocker Bugs |
| :---: | :---: | :---: | :---: |
| **15** | **15** | **2** | **0** |

---

## 🛠 Test Methodology & Scope

- **Testing Type:** Black-Box Exploratory Testing, Boundary Value Analysis (BVA), Negative Validation, Defensive Design & Security Audit.
- **Coverage Areas:**
  1. System Permissions & First-Launch Graceful Degradation
  2. Input Field Boundary & Sanitization (Phone Normalization, Clipboard Paste)
  3. Country Code Switcher & Dynamic Field State
  4. Legal Integration (Privacy Notice WebView)
  5. Account Recovery Flow (`Lost access to number`)
  6. OTP Verification, Resend Channels, & Rate Limiting

---

##  Test Execution Details

### 1. Permissions & Onboarding

| ID | Test Scenario | Steps / Input | Expected Result | Actual Result | Status |
| :--- | :--- | :--- | :--- | :--- | :---: |
| **TC-01** | First Launch Permission Handling | Launch app, reject all system permissions (Location, Contacts, Push, Phone State). | App gracefully degrades and proceeds without blocking or crashing. | Onboarding continues smoothly; user is navigated to phone entry screen. | **PASSED** |

---

### 2. Registration & Phone Number Validation

| ID | Test Scenario | Steps / Input | Expected Result | Actual Result | Status |
| :--- | :--- | :--- | :--- | :--- | :---: |
| **TC-02** | Empty Field Submission | Tap `Continue` with empty phone input. | Block form submission, trigger validation feedback. | Toast notification `"Enter your phone number"` appears; navigation blocked. | **PASSED** |
| **TC-03** | Boundary Length Analysis | Enter phone numbers < 10 digits and > 10 digits. | Strict 10-digit limit enforcement. | Accepts 10 digits; input beyond 10 digits is masked/blocked. | **PASSED** |
| **TC-04** | Clipboard Paste Normalization | Paste international format `+521234567890`. | Strip country code and adapt to 10 local digits. | Prefix `+52` is stripped automatically, leaving 10 valid digits. | **PASSED** |
| **TC-05** | Non-Numeric Input Filtering | Manual letters or paste `abc1234567` / `12 34 56 78 90`. | Sanitize non-numeric and whitespace characters. | Dial pad enforced; whitespaces and alphabetic characters stripped on paste. | **PASSED** |

---

### 3. Country Switcher & State Management

| ID | Test Scenario | Steps / Input | Expected Result | Actual Result | Status |
| :--- | :--- | :--- | :--- | :--- | :---: |
| **TC-06** | Country Selection Options | Tap country selector badge (`🇲🇽 Mexico`). | Open bottom sheet modal with valid country options. | `Choose country` modal displays Mexico (+52) and Colombia (+57). | **PASSED** |
| **TC-07** | Region Switch State Logic | Switch country between Mexico (+52) and Colombia (+57). | Dynamic prefix update and clean state handling. | Dial code updates (`+52` $\rightarrow$ `+57`); phone field is cleared cleanly to prevent syntax errors. | **PASSED** |

---

### 4. Legal Sub-flows & Recovery System

| ID | Test Scenario | Steps / Input | Expected Result | Actual Result | Status |
| :--- | :--- | :--- | :--- | :--- | :---: |
| **TC-08** | Privacy Notice WebView | Click `"Privacy notice"` link on start screen. | Render legal agreement in WebView with working stack navigation. | Opens `Aviso de privacidad` in-app; native back button (`←`) returns safely. | **PASSED** |
| **TC-09** | Recovery Flow Initialization | Tap `"Lost access to number"`. | Open 3-step recovery guide screen. | Informational recovery guide rendered with `Continue` CTA. | **PASSED** |
| **TC-10** | Recovery Form Server Validation | Submit non-existent pair (Phone + CURP `ABCD123456EFGHIJ00`). | Server-side validation error handling. | Backend response: `"A customer with this phone number and CURP wasn’t found"`. | **PASSED** |
| **TC-11** | Flow Interruption & Safety Modal | Exit recovery form with pre-filled user data. | Confirmation prompt to prevent accidental data loss. | Pop-up modal `"If you close this flow, the data... will be lost"` is displayed. | **PASSED** |

---

### 5. OTP Security & Rate Limiting

| ID | Test Scenario | Steps / Input | Expected Result | Actual Result | Status |
| :--- | :--- | :--- | :--- | :--- | :---: |
| **TC-12** | OTP Transition & Countdown | Submit valid 10-digit number. | Render PIN input slots and active resend timer. | `Confirmation` screen opens with masked slots and running countdown. | **PASSED** |
| **TC-13** | Alternative Resend Channels | Tap `"Resend code"` after countdown expiry. | Provide fallback OTP channels. | Bottom sheet opens offering SMS resend or `Phone Call` delivery. | **PASSED** |
| **TC-14** | Invalid OTP Client Handling | Input incorrect 4-digit code (`0000`). | Error toast & automatic slot clearance for retry. | Toast `"Wrong code"` appears; input fields automatically reset. | **PASSED** |
| **TC-15** | Rate Limiting & Brute-Force Defense | Input incorrect code (`0000`) 5 consecutive times. | Terminate session to block brute-force attacks. | Session terminated on 5th failure; toast `"Too many attempts..."`; redirect to start. | **PASSED** |

---

##  Defensive UX Observations

1. **Implicit Field Dependency (Lost Access Flow):**
   - **Finding:** The `CURP` field remains disabled until the `Old Phone Number` field is populated.
   - **Recommendation:** Add explicit `Disabled` visual styling or a helpful hint (e.g., *"Enter old phone number first"*).
2. **Silent Input Reset on Region Change:**
   - **Finding:** Changing country automatically clears pre-entered numbers without notification.
   - **Recommendation:** Show a toast/micro-dialog confirming that the input was reset due to format changes.

---

##  Conclusion

The **Plata Android App** demonstrates robust production readiness:
- Solid permission-handling architecture.
- Clean client-side input sanitization.
- Strong security enforcement (Rate limiting triggering precisely on the 5th attempt).

---

### 🔗 Contact Information
- **Candidate:** Nikita Grachev
- **Email:** ngrachev40@gmail.com
