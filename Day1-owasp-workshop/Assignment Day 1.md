# Day 1 Security Testing Exercise – OWASP Juice Shop

## Objective
Reinforce everything covered in Session 1 by independently hunting for six vulnerability categories in OWASP Juice Shop.

By completing this assignment, you will be able to:

- Identify vulnerability types independently
- Document findings like a security tester
- Explain the root cause behind each issue
- Write QA test cases that would detect these vulnerabilities before release

---

## Use Case
You are a QA engineer assigned to a security review sprint.

The development team has shipped a new internal build of **OWASP Juice Shop** that must be reviewed across six OWASP Top 10 categories before progressing to UAT.

Your responsibility is to identify vulnerabilities, document evidence, explain security impact, and propose preventive test coverage.

---

# Vulnerability Assessment Tasks

---

# 1. Broken Access Control

## Task 1: Post a product review as another user OR edit another user’s review

### Objective
Verify whether authorization controls prevent impersonation or unauthorized modification of user-generated content.

### Steps Performed
1. Logged in with the first user account and posted a product review.
2. Captured the request and extracted the authorization token associated with the first user.
3. Saved the extracted token for reuse.
4. Logged in with a second user account.
5. Attempted to update the first user’s review by reusing the authorization token captured from the first user.

### Findings
**Observed Behavior:**  
The second user was successfully able to update the review originally created by the first user by using the first user’s authorization token.

**Expected Secure Behavior:**  
Users should only be able to access and modify content that belongs to their own account. The application should validate ownership on the server side and prevent any user from viewing, editing, or performing actions on another user’s reviews, even if a token or request is manipulated.

**Security Impact:**  
This vulnerability allows an authenticated user to impersonate another user by posting or modifying product reviews without proper authorization. An attacker could manipulate product reputation by posting fake positive or negative reviews, damage user trust, spread misleading information, or tamper with another user’s content without consent. In a real-world application, this could lead to reputational damage, fraud, abuse of customer trust, and potential compliance issues due to inadequate access control.

**Root Cause Analysis:**  
The application fails to enforce proper server-side authorization checks to verify that the authenticated user is the legitimate owner of the review being created or modified. The backend appears to trust client-supplied identifiers (such as user ID or review ID) without validating ownership against the active session or authentication token. This is a classic Broken Access Control / Insecure Direct Object Reference (IDOR) issue, where sensitive actions are authorized based on manipulable request parameters instead of server-side access validation.

---

### Screenshot Evidence
![Screenshot](./screenshots/Screenshot%202026-05-14%20111300.png)
![Screenshot](./screenshots/Screenshot%202026-05-14%20111420.png)
---

### QA Test Case Recommendation

**Test Scenario:**  
Verify that users cannot modify or access reviews belonging to other authenticated users.

**Test Objective:**
Ensure the application enforces proper authorization checks and restricts users to performing actions only on resources they own.

---

## Task 2: Access admin section and delete all 5-star reviews

### Hint
URL manipulation may be useful.

### Steps Performed
1. Attempted to log in using an SQL injection payload (' OR 1=1--) in the email field along with a random password.
2. Successfully gained unauthorized access to the admin account.
3. Navigated directly to the administration page using the URL: https://localhost:3000/#/administration
4. Accessed the admin dashboard, where administrative controls were available.
5. Verified that the user could view and delete all 5-star product reviews.

### Findings
**Observed Behavior:**  
The application allowed unauthorized access to the administrator account through SQL injection. After gaining access, the user was able to navigate to the administration panel and perform privileged actions such as viewing and deleting customer reviews.

**Expected Secure Behavior:**  
The application should properly validate and sanitize login inputs to prevent SQL injection attacks. Only authenticated users with valid administrator privileges should be allowed to access the administration section or perform high-privilege actions such as deleting reviews. Unauthorized users should be denied access with appropriate security controls in place.

**Security Impact:**  
This vulnerability allows an attacker to gain unauthorized administrative access to the application, resulting in a complete compromise of privileged functionality. An attacker could view, modify, or delete sensitive application data, manipulate customer reviews, access restricted administrative features, and potentially impact the integrity and availability of the system. In a real-world application, this could lead to data breaches, reputational damage, unauthorized business operations, loss of customer trust, and serious compliance or security incidents.

**Root Cause Analysis:**  
The application appears to be vulnerable to SQL Injection due to improper input validation and the failure to use parameterized queries or secure input handling mechanisms during authentication. User-supplied input is likely being directly incorporated into backend database queries without sanitization, allowing malicious SQL payloads to alter query logic and bypass authentication controls. Additionally, access to sensitive administrative functionality relies solely on successful authentication rather than enforcing stronger privilege validation and secure authorization checks.

---

### Screenshot Evidence
![Screenshot](./screenshots/Screenshot%202026-05-14%20130704.png)
---

### QA Test Case Recommendation
**Scenario:** Verify that unauthorized users cannot bypass authentication or gain access to administrative functionality through malicious input manipulation.

**Test Objective:**
Ensure the application properly validates login inputs, prevents SQL injection attacks, and restricts administrative access to authorized users only.

---

# 2. Security Misconfiguration

---

## Task 1: HTTP Security Headers Inspection

Capture a response using Burp Suite and inspect the following headers.

| Header                    | Present? (Yes/No)                  | Value                            | Security Impact if Missing                                                                 |
|---------------------------|------------------------------------|----------------------------------|--------------------------------------------------------------------------------------------|
| X-Frame-Options           | Yes                                | `SAMEORIGIN`                     | The application may be vulnerable to clickjacking attacks, allowing malicious sites to embed the application in iframes and trick users into unintended actions. |
| Content-Security-Policy   | No                                 | Not Present                      | Missing CSP increases the risk of Cross-Site Scripting (XSS), malicious script injection, and loading content from untrusted sources. |
| Strict-Transport-Security | No                                 | Not Present                      | Without HSTS, users may be vulnerable to SSL stripping or protocol downgrade attacks, exposing traffic over insecure HTTP connections. |
| X-Content-Type-Options    | Yes                                | `nosniff`                        | Browsers may perform MIME type sniffing, increasing the risk of executing malicious files as active content. |
| Permissions-Policy        | No *(Feature-Policy observed instead)* | `Feature-Policy: payment 'self'` | Missing or incomplete browser feature restrictions may expose unnecessary browser capabilities and increase the attack surface. |


---

### Task 2: Directory and File Exposure

### Path 1: `/ftp`

**Response Observed:**  
Navigating to the `/ftp` endpoint exposed a publicly accessible directory listing without requiring any authentication or authorization. The application allowed direct access to internal files and folders that should normally be protected from public access.

**Exposed Files:**  
- `quarantine/`
- `coupons_2013.md.bak`
- `incident-support.kdbx`
- `package-lock.json.bak`
- `acquisitions.md`
- `eastere.gg`
- `legal.md`
- `package.json.bak`
- `announcement_encrypted.md`
- `encrypt.pyc`
- `order_28c3-482148b8bc48d3e9.pdf`
- `suspicious_errors.yml`

**Potential Risk:**  
Public exposure of internal directories and application files introduces significant security risks, as attackers can gather sensitive information that may be used for further exploitation.

Some notable risks include:

- **`incident-support.kdbx`**  
  This appears to be a KeePass database file, which may contain sensitive credentials, internal notes, or operational secrets if compromised.

In a real-world application, such exposure could lead to information disclosure, credential compromise, supply chain exploitation, privilege escalation, or even full system compromise depending on the sensitivity of the exposed files.


### Screenshot
![Screenshot](./screenshots/Screenshot%202026-05-15%20102621.png)

---

### Path 2: `/encryptionkeys`

**Response Observed:**  
Navigating to the `/encryptionkeys` endpoint exposed a publicly accessible directory listing without requiring authentication or authorization. The application allowed direct access to sensitive encryption-related files that should be protected from unauthorized users.

**Exposed Files:**  
- `jwt.pub`
- `premium.key`

**Potential Risk:**  
Public exposure of encryption-related files creates a critical security risk, as attackers may gain access to cryptographic material used by the application.

- **`premium.key`**  
  This is the highest-risk file in this directory. The file appears to contain a cryptographic or application-specific secret key. If this key is used for signing tokens, validating premium access, encrypting sensitive data, or unlocking restricted application functionality, an attacker could potentially forge access, bypass licensing or premium restrictions, manipulate application behavior, or gain unauthorized access to protected features.

**Security Impact:**  
Exposure of cryptographic keys or sensitive application secrets can completely undermine application security controls. If the exposed key is actively used in authentication, authorization, or encryption workflows, attackers may be able to forge tokens, bypass access restrictions, impersonate users, or gain elevated privileges.

In a real-world application, this could result in unauthorized access, privilege escalation, account compromise, feature abuse, or full application compromise depending on how the key is used.


### Screenshot
![Screenshot](./screenshots/Screenshot%202026-05-15%20102740.png)
---

## High-Risk Files That Could Be Used to Compromise the System

| File Name             | Location           | Why It Is Dangerous |
|----------------------|-------------------|---------------------|
| `incident-support.kdbx` | `/ftp`             | This is a KeePass password vault file that may contain sensitive credentials such as usernames, passwords, API keys, database access details, or administrative secrets. If an attacker gains access to this file and successfully retrieves its contents, it could lead to credential compromise, unauthorized access, privilege escalation, or complete system takeover. |
| `premium.key`           | `/encryptionkeys`  | This file appears to contain a cryptographic or application secret key. If used for authentication, authorization, encryption, or premium feature validation, an attacker could potentially forge access, bypass restrictions, manipulate protected functionality, or compromise application security controls. |

---

### QA Test Case Recommendation
**Scenario:** Verify that sensitive directories, internal files, backup files, and cryptographic assets are not publicly accessible without proper authentication and authorization.

**Test Objective:**  
Ensure the application does not expose sensitive internal resources, configuration files, backup files, credential stores, or encryption-related assets through direct URL access.


---

# 3. Software Supply Chain Failures

## Task
Download `package.json.bak` from the exposed directory and identify another outdated vulnerable package.

---

### Steps Performed
1. Downloaded the exposed `package.json.bak` file from the publicly accessible `/ftp` directory.
2. Reviewed the dependency versions listed in the file.
3. Cross-checked the identified packages against known vulnerability databases and security advisories.
4. Identified an outdated dependency with a known denial-of-service vulnerability.

---

### Findings

| Package Name | Current Version | Vulnerability Identified | Risk Level | Reference |
|-------------|----------------|--------------------------|------------|----------|
| `multer` | `~1.3` | Denial of Service (DoS) vulnerability caused by improper handling of malformed multipart requests | High | GHSA-fjgf-rc76-4x9p / CVE-2022-24434 |

---

### Security Impact
The use of an outdated vulnerable dependency introduces supply chain risk into the application. In this case, the vulnerable version of `multer` can be exploited through specially crafted multipart/form-data requests, potentially causing the application to consume excessive resources or crash, leading to denial of service.

In a real-world production environment, attackers could exploit this weakness to disrupt file upload functionality, degrade service availability, or impact application stability.

---

### Root Cause Analysis
The application relies on an outdated third-party dependency (`multer ~1.3`) that contains a publicly disclosed security vulnerability. This indicates inadequate dependency lifecycle management, including a lack of regular dependency updates, vulnerability monitoring, or automated software composition analysis (SCA) as part of the development and CI/CD process.

The exposure of `package.json.bak` further increases the risk by allowing attackers to identify exact dependency versions and map them against known vulnerabilities for targeted exploitation.

### Screenshot Evidence
![Screenshot](./screenshots/Screenshot%202026-05-15%20104222.png)

---

### QA Test Case Recommendation
**Test Scenario:**  
Validate that third-party dependencies used by the application are regularly scanned for known security vulnerabilities as part of the CI/CD pipeline.

**Test Objective:**  
Ensure vulnerable or outdated open-source dependencies are identified and flagged before deployment to production.

---

# 4. Insecure Design

---

## Task 2: Place an order that gives money instead of charging

### Hint
Quantity values may not always be positive integers.

### Steps Performed
1. Added a product to the shopping basket in the OWASP Juice Shop application.
2. Intercepted the basket update request using browser developer tools or an interception proxy.
3. Modified the product quantity value in the request from a positive integer to a negative value.
4. Submitted the manipulated request and proceeded to checkout.
5. Observed the pricing calculation and payment behavior.

---

### Findings

**Observed Behavior:**  
The application allowed the user to submit negative product quantities, which resulted in a negative total amount during checkout. Instead of charging the user, the system effectively credited money back to the user for placing the order.

**Expected Secure Behavior:**  
The application should validate product quantity inputs both on the client side and server side, allowing only positive whole numbers within acceptable limits. Invalid quantities such as negative values, zero (if not allowed), or malformed input should be rejected with an appropriate validation error.

**Security Impact:**  
This vulnerability exposes a serious business logic flaw that allows attackers to manipulate transaction calculations for financial gain. An attacker could exploit this issue to place fraudulent orders, receive unauthorized credits, bypass payment controls, or abuse the checkout workflow. In a real-world e-commerce environment, this could result in direct financial loss, accounting discrepancies, abuse of promotional logic, inventory inconsistencies, and significant business impact.

**Root Cause Analysis:**  
The application fails to enforce proper server-side business validation for product quantity inputs. While the user interface may assume that quantity values will be positive integers, the backend does not independently validate incoming request data before processing pricing calculations. This creates an insecure design flaw where application trust is placed on client-side behavior rather than secure backend validation and business rule enforcement.

---

### Screenshot Evidence
![Screenshot](./screenshots/Screenshot%202026-05-14%20123236.png)
![Screenshot](./screenshots/Screenshot%202026-05-14%20123644.png)

---

### QA Test Case Recommendation
**Test Scenario:**  
Validate that business logic rejects invalid product quantity values during cart updates and checkout.

**Test Objective:**  
Ensure the application enforces business validation rules for transaction-related inputs and prevents manipulation of order calculations.

---

# Core Concept Questions

---

## 1. Which vulnerability category is most likely to already exist in a project you’ve worked on, and why?

**Answer:**  
The vulnerability category most likely to exist in real-world projects I have worked on is **Security Misconfiguration**. This is because configuration-related security gaps are often overlooked during fast-paced development cycles, especially when teams focus heavily on functional delivery. Missing security headers, exposed debug endpoints, verbose error messages, weak access permissions, or unintentionally exposed internal files are common issues that can slip through if explicit security validation is not part of the testing strategy. These vulnerabilities are often introduced not through coding mistakes, but through incomplete deployment or infrastructure hardening.

---

## 2. Why do well-known vulnerabilities continue to ship year after year?

### Developer Perspective
Developers are often focused primarily on implementing business functionality and meeting delivery timelines. Security best practices may not always be deeply embedded into daily development workflows, and not all developers have strong secure coding awareness. In some cases, insecure coding patterns, outdated dependencies, or assumptions about frontend validation create vulnerabilities unintentionally.

### Tester Perspective
Traditional testing teams often prioritize functional validation over security validation unless security testing is explicitly included in scope. Many QA teams may not have specialized security testing skills, tooling, or sufficient time to perform deeper negative testing, abuse case testing, or architecture-level validation. As a result, issues like access control flaws or insecure business logic can remain undetected.

---

## 3. What does A03 (Supply Chain) and A06 (Insecure Design) tell you about where security testing should begin in SDLC?

**Answer:**  
A03 (Software Supply Chain Failures) and A06 (Insecure Design) clearly show that security testing cannot begin only during QA or after development is complete. Security needs to start much earlier in the SDLC—during architecture design, dependency selection, threat modeling, and development planning. If insecure design decisions are made early or vulnerable dependencies are introduced from the start, traditional testing later in the cycle can only detect symptoms rather than prevent the root issue. Effective security must be built into design reviews, secure coding practices, CI/CD dependency scanning, and continuous validation throughout the SDLC.

---

## 4. Three regression test cases you would add to every project

### Test Case 1
**Scenario:**  
Validate authorization boundaries by ensuring users cannot access or modify resources belonging to other users.

**Reason:**  
Broken access control remains one of the most common and high-impact vulnerabilities. Verifying ownership validation helps prevent privilege escalation, unauthorized data access, and IDOR-style issues.

---

### Test Case 2
**Scenario:**  
Validate all user input fields against injection attacks and malformed input.

**Reason:**  
Injection vulnerabilities can lead to authentication bypass, data compromise, or backend manipulation. Input validation and secure query handling should always be part of regression coverage.

---

### Test Case 3
**Scenario:**  
Validate business logic enforcement for critical transactional workflows such as payments, pricing, discounts, and order processing.

**Reason:**  
Business logic flaws are often missed in traditional testing because functionality may appear to work correctly under normal usage. Abuse-case validation helps prevent fraud, pricing manipulation, and financial loss.

---

# Personal Reflection

(5–7 sentences)

**Answer:**  
This exercise really changed the way I look at testing. It made me realize that many security issues won’t show up during regular happy-path functional testing because they often come from unexpected or intentionally malicious user behavior. I was surprised by how easily application workflows can be manipulated when the system blindly trusts user input or assumptions about how features will be used. The business logic issue where entering negative product quantities actually resulted in the application giving money instead of charging was especially eye-opening, because such a simple validation gap could have a major real-world impact. I also found it interesting how something as basic as exposed internal files can give attackers valuable information to plan much bigger attacks without even touching the application logic directly. Overall, this exercise reinforced that good testing is not just about checking whether features work as expected.It’s also about thinking from an attacker’s perspective and questioning how things could be misused. Security testing feels much more like a core part of quality engineering rather than something separate or optional.

---
