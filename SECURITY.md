# Security Policy

## Supported Versions

Currently, AI SOC Copilot is in the Pre-MVP phase. Only the latest branch is supported with security updates.

| Version | Supported          |
| ------- | ------------------ |
| MVP (`main`) | :white_check_mark: |
| Pre-MVP | :x:                |

## Reporting a Vulnerability

We take the security of AI SOC Copilot and the data it handles extremely seriously. If you believe you have found a security vulnerability, please report it to us directly.

**Do not report security vulnerabilities through public GitHub issues.**

Instead, please report them via email to: `security@aisoccopilot.com` *(Placeholder)*.

Please include the following details in your report:
- A description of the vulnerability and its impact.
- Detailed steps to reproduce the issue.
- Any relevant logs, screenshots, or Proof of Concept (PoC) code.

### Response Expectations

- We will acknowledge receipt of your vulnerability report within **48 hours**.
- We will provide a status update and estimated timeline for a patch within **5 business days**.
- Once the vulnerability is patched, we will coordinate public disclosure with you.

## Security Best Practices (Scope)

This repository adheres strictly to Secure Coding practices and relies on community vigilance to maintain its integrity. 
- **Secrets:** We use automated SAST tools to block the accidental commit of secrets.
- **Dependencies:** We routinely monitor `requirements.txt` and `pubspec.yaml` via Dependabot.
- **Validation:** All external inputs (like JSON uploads) are heavily validated and sanitized.

For more detailed architectural security information, please read our [Security Architecture](docs/09-Security-Architecture/Security_Architecture.md).
