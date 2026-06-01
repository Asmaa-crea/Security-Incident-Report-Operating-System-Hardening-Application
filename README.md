# Security Incident Report: Operating System Hardening Application

## Course & Project Context
* **Program:** Professional Cybersecurity Training
* **Course:** Course 3 - Network Security & Operating System Protection
* **Activity Name:** Application of Operating System Hardening Techniques (تطبيق تقنيات تقوية نظام التشغيل)

---

## Project Overview
This repository contains a comprehensive, professional Security Incident Report documenting a web application compromise and unauthorized redirection scenario. As a Cybersecurity Analyst, I investigated a security breach affecting the corporate website `yummyrecipesforme.com`, analyzed network traffic artifacts, and formulated production-grade mitigation strategies to harden the host infrastructure.

### Raw Forensic Logs
* **Analyzed Network Traffic Log:** [(https://docs.google.com/document/d/1umT0tOfrpnROG9mFq-SAw4TdF-RR_RihowgrYDlotek/edit?tab=t.0)]

---

## Section 1: Identify the Network Protocols Involved in the Incident

Based on the forensic analysis of the provided network traffic signatures, two primary application-layer network protocols were utilized to facilitate connections and execute the malicious redirection sequence:

### 1. DNS (Domain Name System)
* **Purpose & Function:** DNS is responsible for translating human-readable Fully Qualified Domain Names (FQDNs) into machine-readable IP addresses. 
* **Evidence in Log:**
  * At `14:18:32.192571`, the asset initiated an authorized DNS query (`A? yummyrecipesforme.com.`) to locate the web hosting server, which resolved to `203.0.113.22`.
  * Later, at `14:20:32.192571`, an unauthorized background DNS query requested the IP address for the malicious domain (`A? greatrecipesforme.com.`), which resolved to `192.0.2.17`.

### 2. HTTP (Hypertext Transfer Protocol)
* **Purpose & Function:** HTTP operates at the application layer to transfer web requests and resources (such as HTML pages, scripts, and media) over a reliable TCP connection on port 80.
* **Evidence in Log:**
  * At `14:18:36.786589`, after completing the standard TCP three-way handshake (`[S]`, `[S.]`, `[.]`), the browser issued an unencrypted HTTP `GET / HTTP/1.1` request to the authentic but compromised host, which delivered the injected malicious JavaScript payload.
  * At `14:25:29.576590`, the browser established a secondary HTTP connection to the unauthorized external malicious server (`greatrecipesforme.com.http`) via an HTTP `GET /` request, resulting in extensive rogue data transfer.

---

## Section 2: Incident Documentation

### Incident Timeline & Event Sequence
1. **Initial Vector (Brute-Force Attack):** An unauthorized threat actor launched an automated brute-force attack against the administrative portal of `yummyrecipesforme.com`. Due to a critical lack of identity controls, the attacker successfully guessed the portal credentials because the system was improperly left configured with its **factory default administrator password**.
2. **Unauthorized Access & Code Injection:** Upon gaining access, the attacker altered the web source code, injecting a malicious JavaScript function directly into the index page. To maintain persistence, the attacker changed the administrator password to lock out internal webmasters.
3. **Drive-By Download Execution:** When users visited the homepage, the injected JavaScript executed a fraudulent prompt deceiving visitors into downloading a critical "browser update" executable malware binary.
4. **Malicious Redirection & System Degradation:** Upon execution of the dropped payload, the malware altered the users' local browser configurations, triggering an unauthorized redirection chain through DNS/HTTP queries to a rogue clone site (`greatrecipesforme.com`). This resulted in immediate local endpoint performance degradation (system slowness).

### Extracted Network Signatures
* **Legitimate Hosting Server IP:** `203.0.113.22` (Port 80)
* **Malicious External Server IP:** `192.0.2.17` (Port 80)
* **Connection Profile:** Standard TCP Three-Way Handshake captured prior to each HTTP transmission.

---

## Section 3: Recommended Remediation & OS Hardening Strategy

To mitigate the root cause of this security incident, prevent similar account takeovers, and systematically apply operating system and application hardening baselines, the following security controls must be deployed immediately:

### Enforcing Multi-Factor Authentication (MFA) & Portal Hardening Archetype

1. **Mandatory Multi-Factor Authentication (MFA):**
   * Enforce MFA for all administrative and privileged management accounts. Utilizing secondary verification factors (such as TOTP via an authenticator app) guarantees that even if a password is stolen or brute-forced, the account remains secure.
2. **Account Lockout & Rate Limiting Controls:**
   * Configure strict rate-limiting on all authentication endpoints. Implement an automated account lockout policy that temporarily disables access (e.g., for 30 minutes) after 5 consecutive failed login attempts within a rolling window. This invalidates automated brute-forcing toolsets.
3. **Absolute Elimination of Factory Default Credentials:**
   * Establish a strict system-hardening baseline policy requiring unique, high-entropy passphrase creation immediately upon initialization of any enterprise service, CMS platform, or operating system portal. Default vendor passwords must be entirely restricted.
4. **Network Access Control Lists (ACLs):**
   * Harden the administrative panel interface by placing it behind a corporate Virtual Private Network (VPN) or applying strict IP whitelisting via an Access Control List (ACL) to reduce the external exposure surface.
