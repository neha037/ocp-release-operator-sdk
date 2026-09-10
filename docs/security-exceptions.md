# Security Exceptions

This file documents known security findings that have been reviewed and accepted.

## Process

1. Identify the finding (CVE, GHSA, or tool-specific ID).
2. Document it in this file with rationale.
3. Add the corresponding suppression to `.trivyignore` or govulncheck config.
4. Get approval from a reviewer listed in `OWNERS`.

## Active Exceptions

*No active exceptions at this time.*

## Format

| ID | Tool | Package | Rationale | Reviewer | Date |
|---|---|---|---|---|---|
| *(example)* CVE-YYYY-NNNNN | govulncheck | example/pkg | Not reachable in our code paths | @reviewer | YYYY-MM-DD |
