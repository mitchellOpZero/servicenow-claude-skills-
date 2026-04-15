# Report Structure

Compose the report in this exact order. Each numbered section maps to one card/section in the rendered output. All HTML styling per `shared/report-templates/operatorzero-report-base.md` plus the citation-callout extension noted below.

## 1. Security Risk Summary
Hero card with critical numbers:
- **Critical Exposures** — data accessible to unauthenticated users right now
- **High Risk** — empty ACLs on sensitive tables
- **Security Properties** — misconfigured out of recommended total
- **Instance Scan Findings** — SN's own security check results

One-paragraph verdict with severity assessment.

## 2. Research Context — Why This Matters
Brief section explaining the published research:
- AppOmni found 70% of instances leaking data through these exact misconfigurations
- 1,000+ enterprise instances exposed KB articles with PII and credentials
- CVE-2025-3648 — ServiceNow acknowledged the issue
- Links to all sources

This section establishes credibility — "we're checking for vulnerabilities that have been publicly documented and exploited."

## 3. Empty Access Controls — The Primary Vulnerability
The main finding section. For each empty ACL:
- Which table/resource it protects
- Data sensitivity classification (Critical/High/Medium/Low)
- What an attacker could access
- Reference: "This is the misconfiguration AppOmni found on 70% of tested instances"

Group by sensitivity. Show the worst ones first.

## 4. Knowledge Base Exposure
Specific to the AppOmni KB research:
- Which KBs are accessible without authentication
- Whether User Criteria is properly configured
- Whether the `block_access_with_no_user_criteria` property is set
- Article count at risk
- Reference: "AppOmni found 1,000+ instances exposing KB articles with active credentials"

## 5. Service Portal Attack Surface
- Public portals and pages
- Widgets exposing data to unauthenticated users
- The Simple List widget exploit path (AppOmni's documented attack)

## 6. Client-Callable Script Includes
- Script Includes callable via GlideAjax without ACL protection
- What data they can access
- Server-side code execution risk

## 7. Security Properties Assessment
Table showing each property, current value, recommended value, and risk:
| Property | Current | Recommended | Risk |
|----------|---------|-------------|------|

Color-coded: red for dangerous values, green for correct.

## 8. Instance Scan Validation
Cross-reference our findings with ServiceNow's own Instance Scan:
- Matching findings (our scan agrees with SN's)
- Additional findings (things we caught that SN didn't)
- Establishes dual validation

## 9. Remediation Priority
Ordered action plan:
1. Fix critical exposures (guest access to sensitive data)
2. Add roles to empty ACLs on sensitive tables
3. Set `glide.knowman.block_access_with_no_user_criteria` to true
4. Review and restrict public portal pages
5. Add ACLs to client-callable Script Includes
6. Fix security property misconfigurations
7. Review and resolve Instance Scan findings

Each with effort estimate and impact rating.

## 10. Reference
- Full list of empty ACLs with tables and sensitivity
- Full list of tables without ACLs
- Security properties detail
- Research source links with descriptions
- Methodology notes

## ACL-scan-specific style additions

On top of the shared base:
- Research citations: distinct callout style with source link

## What this skill does NOT do

- Does not attempt to exploit any vulnerability
- Does not access data through the vulnerabilities it finds
- Does not modify any ACLs or security configurations
- Does not test from an unauthenticated perspective (runs as admin)
- Does not guarantee completeness — complex ACL inheritance and scripted conditions may have edge cases
- Does not replace a formal penetration test

## The Pitch

"Security researchers found 70% of ServiceNow instances leaking data through misconfigured access controls. Run `/sn-acl-scan` to check if yours is one of them. Every finding references the published research — hand the report to your CISO."
