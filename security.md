# Security & Data Protection

Same criticality tier as Production Safety in `think-ahead.md`. These rules apply to every plan, feature, and change across all projects.

## Threat Modeling During Planning
- When planning any feature or change, evaluate its security surface before proposing the plan: untrusted input, output rendering, authentication/authorization, data exposure, injection vectors (XSS, SQLi, command injection, SSRF), and secret handling
- For every feature path, define the **abuse behavior** — how it could be misused — alongside the failure behavior required by `think-ahead.md`
- State the security-relevant decisions in the plan, not only after implementation

## Autonomous vs. Authorization-Required
Apply security fixes that are in-scope and low-risk **without asking**:
- Validate and sanitize end-user input
- Escape or encode output to prevent XSS
- Parameterize queries to prevent injection
- Keep secrets and PII out of logs, errors, and analytics
- Apply secure defaults within code you are already writing

**IMPORTANT**: Report and get explicit user confirmation **before** making structural or infrastructure security changes:
- Authentication or authorization model changes
- Cryptography choices (algorithms, key management, hashing)
- Global network, infrastructure, or security-header changes
- Adding a new dependency for security purposes
- Anything already gated by Production Safety or Schema & Contract Changes in `think-ahead.md`

State concretely what changes (before vs. after) and the downstream impact, then wait for confirmation.

## Security Flaws in Existing Code
- **CRITICAL**: When you spot a critical security or data-protection flaw while working on existing code, report it to the user **and** create an observation with `type: security` per `observations.md`
- Set severity proportional to real exploitability and impact — never inflate, never downplay. Calibrate per "Calibrating Findings" in `think-ahead.md`:
  - `high` — exploitable as-is with real impact (reachable stored XSS, exposed PII, auth bypass)
  - `medium` — requires specific conditions or has limited blast radius
  - `low` — defense-in-depth, theoretical, or hard to reach

## Data Protection Compliance (LGPD + TIPA)
Add to review criteria for any change touching personal data: it must comply with **LGPD** (Brazil) and **TIPA** (Tennessee Information Protection Act, US).

- Verify: lawful basis or consent, data minimization, purpose limitation, retention limits, right to access and deletion, encryption of PII at rest and in transit
- **IMPORTANT**: Flag for explicit user confirmation when a change introduces new personal-data collection, a new processing purpose, cross-border data transfer, or third-party data sharing — these compound the Schema & Contract gate in `think-ahead.md` when storage or contracts also change

## Dependency Supply-Chain Gate
- **CRITICAL**: Before adding a **new** dependency (`npm install <pkg>`, editing `package.json`, or the equivalent in any ecosystem), research whether the package is compromised or untrustworthy **before** installing
- Does not apply to installing from an existing committed lockfile (e.g., `npm ci`, or `npm install` that adds no new package)
- Checklist before installing:
  - WebSearch the exact package name and version for "CVE", "malware", "compromised", "supply chain", "typosquat"
  - Confirm it is the official package (correct name, no typosquatting), under active maintenance, with a credible download count and an official repository
  - Prefer established alternatives over obscure packages
- If risk is found or trust cannot be established, stop, report to the user, and do not install
