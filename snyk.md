# agents.md — Snyk-first engineering guardrails (IntelliJ + CI)

## Goal (non-negotiable)
Ship changes that pass Snyk in the IDE *and* in CI:
- Shift left: catch issues while coding (Snyk JetBrains plugin)
- Avoid pipeline delays: no “fix-forward” after PR
- Reduce app risk: secure defaults, minimal attack surface

This file applies to any AI coding agent producing code, configs, tests, build files, or IaC in this repo.

---

## Definition of Done (must satisfy all)
1. **No new High/Critical findings** in:
   - Snyk Open Source (dependencies)
   - Snyk Code (source code)
   - Snyk IaC (Terraform/K8s/CloudFormation/etc.), if applicable
2. Security-sensitive changes include:
   - tests covering the secure behavior
   - safe defaults enabled (see “Secure defaults”)
3. If something cannot be fixed immediately:
   - provide a minimal-risk mitigation
   - document exact follow-up work
   - use ignores only when explicitly allowed (see “Ignores policy”)

---

## Required local checks (tell the developer exactly what to run)
When you finish a change, include these commands in your final response:

### Dependencies (SCA)
- `snyk test --all-projects --severity-threshold=high`

### Code (SAST)
- `snyk code test --severity-threshold=high`

### IaC (only if IaC changed or exists)
- `snyk iac test --severity-threshold=high`

If the repo uses wrappers, prefer:
- `./mvnw test` or `./gradlew test` before Snyk runs

---

## How to respond as an agent (output format)
When proposing changes, output:
1. **What changed** (bullets)
2. **Why it’s secure** (bullets tied to risk)
3. **How to verify** (commands above + any app-specific steps)
4. **Snyk risk notes**:
   - expected prior findings you did NOT touch
   - any remaining risk + mitigation

---

## Fix strategy (prioritize lowest-risk + lowest-churn)
### A) Dependency vulnerabilities (Open Source)
1. Prefer upgrading to a fixed version (minimal major bumps).
2. Prefer vendor-maintained/BOM-managed versions when available.
3. Avoid “exclude the transitive dependency” unless you understand impact.
4. Never introduce:
   - wildcard / floating versions
   - unpinned plugin versions
   - EOL libraries when a supported option exists

### B) Code vulnerabilities (Snyk Code)
1. Fix the root cause (don’t suppress).
2. Prefer secure framework primitives over custom security logic.
3. When multiple fixes exist:
   - pick the simplest that reduces exposure AND maintains behavior
   - add tests to prevent regression

### C) IaC findings
1. Reduce exposure first (public access, overly broad ingress/egress, no encryption).
2. Prefer least privilege and secure-by-default settings.
3. Document any intentional exposure with justification.

---

## Ignores policy (strict)
- **Default: do not ignore. Fix.**
- If you must ignore a dependency/IaC vulnerability:
  - require a written reason
  - set an expiry date (no permanent ignores unless approved)
  - scope the ignore narrowly (path/module)
- **Do not use `.snyk` ignores for Snyk Code findings** (code issues must be fixed or handled via org-level policy per team standards).

---

## Secure defaults checklist (use these patterns)
### Secrets & credentials
- Never hardcode secrets/tokens/keys.
- Load secrets from environment/secret manager.
- Do not log secrets or sensitive headers.

### Crypto & tokens
- Use modern primitives (no MD5/SHA1 for security, no custom crypto).
- Use secure random for secrets.
- Enforce token validation (issuer/audience/expiry) where relevant.

### Web/API security (Java/Spring typical)
- Validate inputs (length/format/ranges) at the boundary.
- Avoid SQL injection: use parameterized queries / ORM safely.
- Avoid SSRF: allowlist hosts, block link-local/metadata IP ranges.
- Avoid XSS: output encode; don’t render untrusted HTML.
- Avoid deserialization of untrusted data.
- Enforce authz checks server-side (don’t rely on client/UI).
- Prefer least-privilege roles/scopes.

### Transport & headers
- Enforce TLS; never downgrade security settings.
- Cookies: `Secure`, `HttpOnly`, and appropriate `SameSite`.
- Add or preserve security headers where applicable (HSTS, etc.).

### Logging
- No sensitive data in logs (tokens, passwords, PII).
- Use structured logs where possible; include request IDs, not payload dumps.

---

## Build file guardrails (Maven/Gradle)
- Prefer BOMs / dependencyManagement to centralize versions.
- Pin plugin versions (no implicit/latest).
- Keep dependency upgrades intentional and explained in PR notes.
- If changing versions, include “Why this version” + “What Snyk it fixes”.

---

## “Stop the line” conditions (do not proceed without fixing)
- New High/Critical issue introduced
- Any new use of insecure crypto or disabling TLS/cert validation
- Hardcoded secret or credential-like string
- Injection risk (SQL/command/template) without safe parameterization
- Broad network exposure in IaC (0.0.0.0/0) without explicit justification

---

## Optional but recommended repo additions (suggest, don’t force)
- A script: `./security-check.sh` running tests + Snyk commands
- Pre-commit hook calling the script for changed modules
- A short `SECURITY.md` describing severity thresholds and ignore approval flow
