# 🔐 AISec Plus — Week 1: Map the Threat

**Researcher:** Daniel Ossai
**Repo:** `aisecplus-week01-danielossai`
**Submitted:** Week 1 · AISec Plus Bootcamp

---

## 📌 Incident Name & Date

**EchoLeak (CVE-2025-32711)**
Discovered: January 2025 · Disclosed: June 2025 · Patched: June 2025

---

## 📝 What Happened

Researchers at Aim Security discovered a critical zero-click indirect prompt injection vulnerability in Microsoft 365 Copilot. By sending a single crafted email — with no user interaction required — an attacker could cause Copilot to silently access internal files and exfiltrate their contents to an attacker-controlled server.

The victim didn't need to click anything. They didn't need to open an attachment. Simply having the email in their inbox was enough for Copilot to process hidden malicious instructions and begin leaking sensitive enterprise data.

Microsoft assigned it **CVE-2025-32711** with a CVSS score of **9.3 (Critical)**. It is the first documented case of prompt injection being weaponized for concrete data exfiltration in a production AI system. Microsoft patched it server-side; no exploitation in the wild was confirmed.

---

## ⚙️ How the Attack Worked

The attack chained four distinct bypasses to achieve silent data exfiltration:

**Step 1 — XPIA Classifier Bypass**
Microsoft 365 Copilot uses Cross-Prompt Injection Attack (XPIA) classifiers to detect and block malicious instructions. The attacker bypassed this by writing the prompt in natural language that appeared directed at the human recipient — never mentioning AI, Copilot, or any technical trigger words. The classifier never flagged it.

**Step 2 — Link Redaction Bypass**
Copilot normally redacts external links to prevent data from leaving the M365 environment. The attacker circumvented this using reference-style Markdown formatting, which Copilot rendered without triggering the redaction filter.

**Step 3 — Auto-Fetched Image Exploit**
Copilot's auto-fetch behaviour for embedded images was abused to initiate outbound HTTP requests to an attacker-controlled server — carrying exfiltrated data in the request parameters.

**Step 4 — Content Security Policy (CSP) Bypass via Teams Proxy**
Microsoft Teams operates as a trusted proxy within M365's content security policy. The attacker routed the exfiltration request through this allowed proxy, bypassing the CSP entirely.

**The result:** Full privilege escalation across LLM trust boundaries, with no user interaction. The payload was pure text — no code, no malware signatures. Copilot was behaving exactly as designed: processing input and responding helpfully. Traditional defenses like antivirus, firewalls, and static file scanning were completely ineffective.

---

## 🗺️ Threat Mapping

| Framework | Classification |
|---|---|
| **NIST Family** | **Abuse** (weaponizing a functioning model) + **Privacy** (unauthorized data exfiltration) |
| **OWASP LLM01** | Prompt Injection — hidden instructions hijacked Copilot's behaviour |
| **OWASP LLM02** | Sensitive Information Disclosure — internal files exfiltrated without user consent |
| **OWASP LLM08** | Vector & Embedding Weaknesses — RAG architecture exploited to access internal data |
| **MITRE ATLAS** | Check [atlas.mitre.org/studies](https://atlas.mitre.org/studies) for the documented case study |

> **Key insight:** This is a textbook **indirect prompt injection** — the malicious instruction came from an external source (email), not from the user. The LLM could not distinguish between trusted developer context and untrusted external content.

---

## 💥 The Impact

EchoLeak represents a structural risk, not just a one-off bug:

- **Immediate risk:** Any organisation with M365 Copilot enabled was potentially vulnerable to silent data exfiltration via a single crafted email
- **Scope:** Works across Word, PowerPoint, Outlook, and Teams — any surface where Copilot processes content
- **Novelty:** First confirmed zero-click exploit against a production AI agent; the first time prompt injection achieved real data exfiltration in a live enterprise system
- **Broader implication:** The structural attack surface applies to any LLM-based assistant with access to multiple internal data sources — not just Microsoft's implementation
- **Confirmed exploitation:** None in the wild; Microsoft patched server-side before known abuse occurred

---

## 🛡️ Prevention & Defense

**What Microsoft did:**
- Server-side patch deployed (no customer action required)
- DLP tags introduced to block Copilot from processing external emails
- New M365 Roadmap feature restricting Copilot's access to emails

**What the broader lesson is:**
- **Scoped data access** is the real fix — limit what Copilot can reach before a vulnerability can abuse it
- **Treat external content as untrusted** — emails, documents, and web content should never be in the same trust boundary as system instructions
- **Separate instruction and data channels** — the LLM should never be able to confuse a user's email with an operator command
- **Output sandboxing** — filter and validate what the model is allowed to send outbound before it leaves the system
- Patching individual CVEs is not sufficient; the architectural pattern (LLM + RAG + broad data access) must be hardened by design

---

## 🔗 Sources

1. The Hacker News — Zero-Click AI Vulnerability Exposes Microsoft 365 Copilot Data Without User Interaction
   https://thehackernews.com/2025/06/zero-click-ai-vulnerability-exposes.html

2. Hack The Box — Inside CVE-2025-32711 (EchoLeak): Prompt injection meets AI exfiltration
   https://www.hackthebox.com/blog/cve-2025-32711-echoleak-copilot-vulnerability

3. Checkmarx — EchoLeak (CVE-2025-32711): Attack chain analysis
   https://checkmarx.com/zero-post/echoleak-cve-2025-32711-show-us-that-ai-security-is-challenging/

4. Sentra — EchoLeak: What the Microsoft Copilot Prompt Injection Vulnerability Means for Your Data
   https://sentra.io/blog/copilot-echoleak-prompt-injection

5. arXiv — EchoLeak: The First Real-World Zero-Click Prompt Injection Exploit in a Production LLM System
   https://arxiv.org/abs/2509.10540

---

*AISec Plus · Week 1 · We are all growing. Show up. Build. Improve.*
