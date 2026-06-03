# Case Study 001: Reviewing an AI + FinTech MVP from a QA and UX Perspective

## Overview

This case study documents an anonymized review of an early-stage AI + FinTech MVP.

The product included several core modules:

* Signature-based digital identity login
* Personal finance dashboard
* Financial data visualization
* AI assistant entry point
* Score / eligibility-related product flow
* Admin and platform-related backend structure

The goal of this review was not to perform a full security audit or code review, but to evaluate the product from a **Senior QA / Product Quality** perspective:

* Is the core user journey understandable?
* Does the product build trust during sensitive flows?
* Are product claims reflected clearly in the UI?
* Are key financial outputs explainable?
* What should be prioritized before broader user testing or release?

---

## Review Context

The MVP was provided as a private GitHub repository during an interview process.

Before running the project, I treated it as an untrusted third-party codebase and followed a controlled review process.

The review included:

1. Repository structure review
2. Dependency and install-script inspection
3. Isolated cloud-based runtime setup
4. Backend and frontend startup verification
5. Core login flow testing
6. UI/UX review of the dashboard and key product flows
7. Prioritized feedback preparation

---

## Environment Setup Approach

Instead of running the repository directly on my local machine, I deployed it in a temporary cloud VM environment.

This helped reduce local risk while still allowing hands-on product exploration.

Before installing dependencies, I checked for common high-risk patterns:

```bash
grep -R "postinstall\|preinstall\|prepare" .
grep -R "child_process\|exec(\|spawn(\|fork(" .
grep -R "curl\|wget\|bash -c\|sh -c" .
```

The purpose was to identify:

* Install-time script execution
* Hidden system command execution
* Remote script downloads
* Unexpected shell execution
* Suspicious automation behavior

After the initial inspection, I installed dependencies using:

```bash
npm install --ignore-scripts
```

This avoided triggering lifecycle scripts automatically during installation.

---

## Product Flow Reviewed

The main flow reviewed was:

```text
Landing / Login Page
↓
Digital identity connection
↓
Signature confirmation
↓
Dashboard
↓
Financial overview
↓
AI assistant and score-related entry points
```

This flow was important because it represents the user’s first interaction with the product and directly affects trust, activation, and product credibility.

---

## Key Finding 1: Supported Login Networks / Providers Were Not Clear in the UI

The README described support for multiple identity providers / networks.

However, in the actual onboarding UI, the user only sees a generic:

```text
Connect Wallet
```

button.

From a first-time user perspective, it is not immediately clear:

* Which providers are supported
* Which network or identity method should be used
* Whether multiple options are available
* What happens after connection
* Whether the flow is intended for technical users only

### Why This Matters

For products involving identity, finance, or account-level access, ambiguity during login can reduce user trust.

If the product supports multiple connection methods, the UI should make that visible during onboarding.

### Suggested Improvement

Add a clearer provider selection or supported-provider hint, for example:

```text
Supported login methods:
- Ethereum-compatible identity providers
- Solana-compatible identity providers
- Browser-based identity extensions
```

Or provide a connection modal such as:

```text
Choose how to connect:
[Provider A]
[Provider B]
[Provider C]
```

This would reduce uncertainty and make the first-time login flow more transparent.

---

## Key Finding 2: Signature Request Lacked Human-Readable Context

During login, the browser identity extension displayed a signature request containing a nonce-like random string.

From a technical perspective, this is likely part of a challenge-response login mechanism.

However, from a user perspective, signing a random string can feel unclear or risky.

### Why This Matters

For trust-sensitive products, users need to understand what they are signing and why.

A signature request should clearly explain:

* The purpose of the signature
* That it is used only for login
* That it does not trigger a transaction
* That no fee is involved
* That no asset or account permission is being granted

### Suggested Improvement

Use a human-readable login challenge message, for example:

```text
Welcome to [Product Name]

Please sign this message to verify ownership of your account.

This signature is used only for login.
No transaction will be sent.
No fee will be charged.

Nonce: xxxxxx
Timestamp: yyyy-mm-dd hh:mm:ss
```

This improves both user trust and perceived product maturity.

---

## Key Finding 3: Score / Eligibility Flow Needed More Explainability

The dashboard displayed a score or eligibility-related entry point prominently after login.

However, the UI did not clearly explain:

* How the score is calculated
* Which data points influence the result
* Whether the score is based on real or sample data
* What users can do to improve it
* When the score was last updated

### Why This Matters

In financial products, users usually do not only care about the result.

They also care about:

```text
Why did I get this result?
Can I trust it?
What can I do next?
```

Without explanation, scoring or eligibility outputs may feel like a black box.

### Suggested Improvement

Add an explanation layer near the score / eligibility entry point:

```text
Your score is based on:
- Account activity
- Spending behavior
- Budget consistency
- Recent transaction history
- Available balance trend
```

Also consider adding:

* Last updated timestamp
* Confidence level
* Improvement suggestions
* Data source explanation
* “Why am I seeing this?” link

This would improve transparency and user trust.

---

## Additional QA Observations

Beyond the three primary UX findings, I would also recommend testing the following areas before broader release:

### Onboarding

* First-time login
* Reconnection flow
* Rejected signature
* Unsupported provider
* Expired session
* Logout and re-login

### Dashboard

* Empty state
* Demo data state
* Real data state
* Loading state
* Error state
* Mobile responsiveness

### Financial Data Display

* Currency formatting
* Time range clarity
* Category breakdown
* Budget status
* Chart legends and tooltips
* Data freshness indicators

### AI Assistant

* What happens if no AI API key is configured?
* How does the assistant explain its reasoning?
* Does it reference user data correctly?
* Are uncertain answers handled safely?
* Are financial suggestions clearly framed as guidance rather than guarantees?

---

## Review Boundary

This review was intentionally limited to high-value product quality observations.

It did not include:

* Full penetration testing
* Full backend security audit
* Full API contract test suite
* Complete automation framework design
* Full product roadmap recommendation

For an interview or early-stage evaluation context, I believe it is important to provide meaningful feedback while keeping the scope clear.

---

## What This Case Demonstrates

This case demonstrates my approach to early-stage product quality review:

1. Treat third-party repositories carefully before execution
2. Use an isolated environment for hands-on testing
3. Validate whether README claims match actual UI behavior
4. Review trust-sensitive user flows from a non-technical user perspective
5. Focus on explainability, onboarding clarity, and release readiness
6. Provide prioritized feedback without over-scoping the review

---

## Key Takeaway

For AI + FinTech products, quality is not only about whether the system works.

It is also about whether users can understand, trust, and safely act on what the system presents.

A technically functional MVP can still fail user trust if:

* Login methods are unclear
* Signature requests are not understandable
* Scores or financial insights lack explanation
* Error and empty states are not handled
* Product promises are not reflected clearly in the UI

This is why product quality review should cover not only functionality, but also trust, explainability, and release readiness.
