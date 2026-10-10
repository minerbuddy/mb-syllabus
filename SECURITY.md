# Security Policy

Thank you for helping keep Mino API and its users safe. This document explains which versions are supported, how to report a vulnerability, what you can expect from us, and how to deploy the project securely.

## Table of Contents

- [Supported Versions](#supported-versions)
- [Reporting a Vulnerability](#reporting-a-vulnerability)
- [What to Include in a Report](#what-to-include-in-a-report)
- [Our Response Process](#our-response-process)
- [Disclosure Policy](#disclosure-policy)
- [Scope](#scope)
- [Safe Harbor](#safe-harbor)
- [Known Security Limitations](#known-security-limitations)
- [Secure Deployment Guide](#secure-deployment-guide)
- [Data Handling and Privacy](#data-handling-and-privacy)
- [Safety-Related Content Issues](#safety-related-content-issues)
- [Dependency Security](#dependency-security)

---

## Supported Versions

Security fixes are applied to the latest release on the `main` branch only.

| Version | Supported |
|---------|-----------|
| 2.1.x (latest) | Yes |
| Older than 2.1.0 | No |

If you run an older version, please upgrade before reporting an issue.

## Reporting a Vulnerability

**Please do not report security vulnerabilities through public GitHub issues, discussions, or pull requests.**

Use one of these private channels:

1. **GitHub private vulnerability reporting** (preferred). Go to the repository's **Security** tab and choose **Report a vulnerability**. This option must be enabled in the repository settings by a maintainer.
2. **Email:** **`[ADD SECURITY CONTACT EMAIL HERE]`**

If you would like to encrypt your report, ask for a PGP key in your first message and we will provide one.

## What to Include in a Report

The more detail you give, the faster we can help. Please include:

- A clear description of the vulnerability and its potential impact
- The affected endpoint, file, or configuration (for example `POST /query/` or `app/services/gemini.py`)
- Step-by-step instructions to reproduce, with example requests or a small proof of concept
- The version or commit hash you tested
- Your environment (OS, Python version, hosting platform, model name)
- Any suggested fix or mitigation, if you have one

**Do not include real API keys or other people's personal data in your report.** If you found an exposed credential, tell us where it is, but do not attach the credential itself.

## Our Response Process

These are our target timelines. Maintainers are a small team, so they are goals rather than guarantees:

| Stage | Target |
|-------|--------|
| Acknowledge your report | within 3 business days |
| Initial assessment and severity rating | within 7 days |
| Status updates | at least every 14 days until resolved |
| Fix for critical or high severity issues | as quickly as possible, ideally within 30 days |
| Fix for medium or low severity issues | within the next regular release |

When a report is accepted, we will:

1. Confirm the issue and determine which versions are affected
2. Develop and test a fix privately
3. Release the fix and publish an advisory crediting the reporter (unless you prefer to stay anonymous)

If we decide a report is not a vulnerability, we will explain why.

## Disclosure Policy

We follow **coordinated disclosure**:

- Please give us a reasonable amount of time to fix the issue before sharing details publicly. We ask for up to **90 days** from the date of your report, and will aim to be much faster.
- We will tell you when the fix is released, and you are welcome to publish your own write-up after that.
- If an issue is being actively exploited, we will work with you to speed up the process and warn users.

## Scope

### In scope

- Exposure or leakage of secrets such as `GOOGLE_API_KEY` (through responses, logs, or error messages)
- Bypassing the CORS allow-list in a way that lets untrusted origins read responses
- Vulnerabilities in request handling for `POST /query/`, including malformed or oversized input that crashes or exhausts the service
- Ways to abuse the API to consume the owner's Gemini quota in unintended ways
- Information disclosure through error events or stack traces
- Insecure defaults in `render.yaml` or the documented configuration
- Vulnerable dependencies that are actually exploitable in this project

### Out of scope

- Vulnerabilities in third-party services (Google Gemini API, Render, GitHub). Please report these to the respective vendor
- Social engineering, phishing, or physical attacks against maintainers or users
- Denial-of-service attacks that rely on sending massive traffic volumes
- Automated scanner output without a demonstrated, exploitable impact
- Missing security headers or best practices with no real security impact
- Issues that need a compromised device or a malicious browser extension
- The model giving a wrong or incomplete answer (see [Safety-Related Content Issues](#safety-related-content-issues) for how to report this)
- Attempts to make the model reveal its system prompt. The prompt is public in this repository, so it is not a secret

## Safe Harbor

We support good-faith security research. If you follow this policy, we will not pursue or support legal action against you for your research. To stay within safe harbor, please:

- Only test against your own deployment or a local copy, not against other people's production systems or data
- Avoid privacy violations, data destruction, and disruption of service
- Do not access, modify, or keep data that does not belong to you
- Stop testing and report immediately if you encounter sensitive data
- Give us reasonable time to respond before any public disclosure

This policy does not give permission to attack systems operated by third parties.

## Known Security Limitations

The current version (2.1.0) is a simple backend and does **not** yet include several protections. If you deploy it publicly, be aware of the following:

| Limitation | Risk | Recommended mitigation |
|------------|------|------------------------|
| No authentication on `/query/` | Anyone who can reach the URL can use your Gemini quota | Put the API behind an API gateway, or add token-based authentication |
| No rate limiting | Quota exhaustion, unexpected costs, abuse | Add rate limiting at the proxy or application level |
| No maximum size for `query` or `image_base64` | Memory exhaustion, high latency, high cost | Enforce request size limits (at the proxy and in the Pydantic model) |
| Raw exception text is sent to the client in `error` events | May reveal internal details | Return a generic error message and log details server-side |
| Invalid base64 images are skipped silently | Confusing behavior, hard to audit | Return a validation error instead |
| CORS only protects browsers | Non-browser clients can still call the API freely | CORS is not a substitute for authentication |
| Dependencies are not version-pinned | Unexpected or vulnerable versions may be installed | Pin versions and review updates |
| Synchronous streaming call inside an async route | A few slow requests can block others, which makes denial of service easier | Use the async Gemini client (`client.aio`) |
| No automated tests or security scanning | Regressions may go unnoticed | Add tests and CI checks |

We track improvements to these items in the [Roadmap](README.md#roadmap).

## Secure Deployment Guide

If you run your own instance, please follow these practices.

### Secrets

- Store `GOOGLE_API_KEY` only in environment variables or your host's secret manager. **Never commit it** to Git, and make sure `.env` is in `.gitignore`.
- In `render.yaml`, do not hard-code the key. Mark it as a secret that is set in the dashboard.
- Restrict the key in Google AI Studio or Google Cloud (for example, by API and, if possible, by usage limits or budget alerts).
- **Rotate the key immediately** if you think it was exposed, and review your usage for anything unexpected.

### Network and CORS

- Always serve the API over **HTTPS**. Render provides this by default.
- Set `ALLOWED_ORIGINS` to the exact URLs of your frontend. Never use `*`, especially because the app enables `allow_credentials`.
- Treat CORS as a browser protection only. Add authentication for real access control.

### Abuse protection

- Add rate limiting per IP or per user (for example at a reverse proxy or API gateway).
- Set request body size limits so oversized images and queries are rejected early.
- Set a spending cap or alerts on your Google account so abuse cannot run up a large bill.

### Application hardening

- Run with a non-reload production command (`uvicorn app.main:app --host 0.0.0.0 --port 8000`). Do not run with `reload` in production.
- Keep Python and all dependencies up to date, and pin versions in `requirements.txt`.
- Do not log full request bodies, since they may contain user images or personal details.
- Monitor logs for repeated failures, unusual traffic spikes, and error events.

### If you add retrieval (RAG) later

- Treat all retrieved documents as **untrusted input**. They may contain instructions that try to manipulate the model (prompt injection).
- Control who can add documents to the knowledge base, and record where each document came from.
- Never place secrets in documents or prompts that could be returned to users.

## Data Handling and Privacy

- Mino API as written does **not** store questions, images, or answers in a database or on disk.
- Each question and any attached image is **sent to the Google Gemini API** for processing. Their terms and data-handling practices apply, so review them before sending sensitive data.
- Your hosting provider may keep request logs. Check its retention settings.
- Users should be told not to submit personal, confidential, or company-sensitive information in their questions or images.

If you add storage (chat history, user accounts, uploaded documents), you become responsible for protecting that data. Please update this policy and your privacy notice accordingly.

## Safety-Related Content Issues

Mino gives information about mining safety, regulations, and operations. A wrong or misleading answer on topics such as ventilation, gas monitoring, blasting, or electrical safety could contribute to real harm.

If you find a case where Mino gives dangerous or clearly incorrect safety guidance:

- Open a regular issue (this is not a security vulnerability) with the question you asked, the answer you received, and what the correct information is, with an official source if possible.
- If the problem comes from a weakness in the system prompt, say so. Please see the section on changing the system prompt in [CONTRIBUTING.md](CONTRIBUTING.md).

Users should always follow their site's official safety procedures and the latest DGMS circulars. Mino's answers are **not** a replacement for qualified professionals or official regulations.

## Dependency Security

The project currently depends on `fastapi`, `uvicorn[standard]`, `google-genai`, `python-dotenv`, and `pydantic`. We recommend:

- Enabling **GitHub Dependabot** alerts and security updates for the repository
- Regularly running a dependency audit, for example `pip install pip-audit` followed by `pip-audit`
- Pinning versions and reviewing changelogs before upgrading
- Reporting vulnerable dependencies through the channels above if you believe they are exploitable in this project

## Acknowledgments

We are grateful to everyone who reports issues responsibly. With your permission, we will credit you in the security advisory for the fix.

---

*Last updated: October 2026*
