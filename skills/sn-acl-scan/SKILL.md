# ServiceNow ACL Security Scanner

Scan a ServiceNow instance for the same access control misconfigurations that security researchers found on 70% of enterprise instances. Every finding references the published research that documented the vulnerability — AppOmni, Obsidian Security, Varonis, and the associated CVEs. This is the report you hand to your security team or CISO.

## Arguments
- No arguments required — always scans the whole instance
- Optional: `html` (default) or `md` for output format

## Core Principle

**This isn't an audit checklist — it's a vulnerability scan backed by published security research.**

Every finding category maps to a real-world data exposure documented by security research firms. The report doesn't say "best practice says you should do X." It says "this specific misconfiguration caused data leaks on 70% of tested enterprise instances (AppOmni, 2023) and yours has it too."

## Research Sources (cite in every report)

These are referenced throughout the report. Each finding category ties back to one or more:

| Source | Finding | Link |
|--------|---------|------|
| AppOmni (Aaron Costello, 2023) | ~70% of instances tested had ACL misconfigs leaking data | https://appomni.com/ao-labs/servicenow-knowledge-bases-data-exposures-uncovered/ |
| AppOmni (2024) | 1,000+ enterprise instances exposing KB articles with PII and credentials | https://appomni.com/ao-labs/servicenow-knowledge-bases-data-exposures-uncovered/ |
| BleepingComputer | 45% of enterprise instances leaking KB data — PII, system details, active credentials | https://www.bleepingcomputer.com/news/security/over-1-000-servicenow-instances-found-leaking-corporate-kb-data/ |
| Dark Reading | Thousands of ServiceNow KB instances exposed | https://www.darkreading.com/cloud-security/servicenow-kb-instances-expose-corporate-data |
| CIO.com | "Warning to ServiceNow admins: Fix your ACLs now" — CVE-2025-3648 | https://www.cio.com/article/4019819/warning-to-servicenow-admins-fix-your-access-control-lists-now.html |
| Threatpost | "Most ServiceNow Instances Misconfigured, Exposed" | https://threatpost.com/most-servicenow-instances-misconfigured-exposed/178827/ |
| ServiceNow (BusinessWire) | Official acknowledgment of widespread misconfiguration | https://www.businesswire.com/news/home/20220309005387/en/ |

## Critical Rules

### Tie every finding to published research
- Don't just say "you have 330 empty ACLs." Say "you have 330 ACLs with no role restriction and no condition — the same misconfiguration AppOmni found leaking data on 70% of enterprise instances."
- Include the source link for each finding category.
- This transforms the report from "we think this is bad" to "researchers proved this is bad and you have it."

### Classify by exploitability, not just count
- **Critical**: Unauthenticated users can access data RIGHT NOW (guest/public access to sensitive tables)
- **High**: Authenticated users can access data they shouldn't (empty ACLs on restricted tables)
- **Medium**: Missing controls that could be exploited if combined with other weaknesses
- **Low**: Best practice gaps that increase attack surface

### Show what's actually exposed, not just what's misconfigured
- Don't just count empty ACLs. Show which TABLES those ACLs protect and whether those tables have sensitive data.
- An empty ACL on `sys_user` (user records with emails, phone numbers) is critical. An empty ACL on `sys_documentation` is noise.
- Prioritize findings by the sensitivity of the data at risk.

## Data Collection

Use whatever ServiceNow access method is available (REST API, GlideRecord scripts, MCP tools, SDK, etc.) to collect the following data. All queries reference standard ServiceNow tables.

### Phase 1: Instance Security Profile

Retrieve these system properties using `gs.getProperty()`:
- `glide.buildtag` — instance version
- `glide.security.use_csrf_token` — CSRF protection
- `glide.cms.catalog.public_access` — public catalog access
- `glide.ui.session_timeout` — session timeout
- `glide.basicauth.required.scriptedprocessor` — basic auth on processors
- `glide.script.block.client.globals` — client script security
- `glide.ui.escape_all_script` — XSS protection
- `glide.security.strict.updates` — strict update security

### Phase 2: Empty ACLs (THE MAIN VULNERABILITY)
This is the #1 finding from AppOmni — ACLs with no Required Role and no Condition.

Query `sys_security_acl` where active=true. Check for records where:
- No required roles (`sys_security_acl_role` has no entries for this ACL)
- No condition
- No script
These default to ALLOW ALL — including guest users.

For each empty ACL, get the table/object it protects and categorize by data sensitivity:
- Critical: User data (sys_user, sys_user_group), credentials, tokens
- High: Business data (incident, change_request, customer records)
- Medium: Configuration data (sys_properties, sys_script)
- Low: Reference data (sys_choice, sys_documentation)

### Phase 3: Tables Without ANY ACLs

Get all tables from `sys_db_object` that have records. Cross-reference with `sys_security_acl` to find tables with ZERO ACLs. These have no access control whatsoever.

Filter to tables that actually matter:
- Have records (not empty schema tables)
- Are not system/internal tables
- Focus on custom tables (u_ prefix) — these are most commonly unprotected
- Check task-extended tables, cmdb-extended tables

### Phase 4: Knowledge Base Exposure (AppOmni's primary finding)

Query `kb_knowledge_base` for all active knowledge bases (fields: title, active, kb_version).

For each KB:
- Check if User Criteria allows public/guest access
- Check if "Can Read" user criteria is set or empty
- Check if any ACL on `kb_knowledge` restricts access
- Count published articles per KB

Check if KB articles are accessible via the Simple List widget without authentication — this is the exact exploit path AppOmni documented. Query `sp_widget` where id='widget-simple-list' and check if it exposes kb_knowledge.

Check `gs.getProperty('glide.knowman.block_access_with_no_user_criteria')` — if false, KBs without user criteria are publicly accessible.

### Phase 5: Service Portal Exposure

Query `sp_portal` for all active portals (fields: title, url_suffix, login_page, public). Check which portals allow unauthenticated access.

Query `sp_page` where public=true. Check for pages that don't require authentication.

For public portal pages, check which widgets are rendered and what data those widgets expose. Focus on: Simple List, Data Table, Record List widgets that may expose table data to unauthenticated users.

### Phase 6: Client-Callable Script Includes

Query `sys_script_include` where client_callable=true AND active=true (fields: name, api_name, access, sys_scope).

For each client-callable Script Include:
- Check if a corresponding ACL exists on sys_script_include.[name]
- If no ACL → any authenticated user can call it via GlideAjax
- Check if the script accesses sensitive data (GlideRecord queries on sensitive tables)
- This is a server-side code execution vector

### Phase 7: Guest User and Public Access Scope

Check the guest user account (`sys_user` where user_name='guest') — is it active? What roles does it have?

Check `sys_user_role` for the 'public' role — what tables/features does this role grant access to?

Check for ACLs that explicitly grant access to 'public' or no role — these are intentionally or accidentally public-facing.

### Phase 8: Security Properties Audit

Check these critical security properties via `gs.getProperty()`:
- `glide.security.use_csrf_token` (CSRF protection)
- `glide.ui.security.allow_coexistence` (mixed content)
- `glide.basicauth.required.scriptedprocessor` (API auth)
- `glide.authenticate.sso.redirect.url` (SSO config)
- `glide.security.strict.updates` (strict update mode)
- `glide.script.block.client.globals` (client security)
- `glide.ui.escape_all_script` (XSS prevention)
- `glide.ui.escape_text` (text escaping)
- `glide.cms.catalog.public_access` (public catalog)
- `glide.knowman.block_access_with_no_user_criteria` (KB security — THE AppOmni finding)
- `glide.security.file.mime_type.validation` (file upload security)
- `glide.ui.session_timeout` (session management)
- `glide.login.home` (login redirect)

For each: current value, recommended value, risk if misconfigured.

### Phase 9: Instance Scan Security Findings

Pull from existing `scan_finding` data — these are ServiceNow's own security checks:
- Deactivated Security Controls
- Empty ACLs (ServiceNow's own check)
- Tables without ACLs
- Public portal pages
- UI Pages without ACLs
- Client-callable Script Includes without ACLs
- Users with local passwords

Group by priority. These validate our own findings with SN's built-in tooling.

## Document Structure

### 1. Security Risk Summary
Hero card with critical numbers:
- **Critical Exposures** — data accessible to unauthenticated users right now
- **High Risk** — empty ACLs on sensitive tables
- **Security Properties** — misconfigured out of recommended total
- **Instance Scan Findings** — SN's own security check results

One-paragraph verdict with severity assessment.

### 2. Research Context — Why This Matters
Brief section explaining the published research:
- AppOmni found 70% of instances leaking data through these exact misconfigurations
- 1,000+ enterprise instances exposed KB articles with PII and credentials
- CVE-2025-3648 — ServiceNow acknowledged the issue
- Links to all sources

This section establishes credibility — "we're checking for vulnerabilities that have been publicly documented and exploited."

### 3. Empty Access Controls — The Primary Vulnerability
The main finding section. For each empty ACL:
- Which table/resource it protects
- Data sensitivity classification (Critical/High/Medium/Low)
- What an attacker could access
- Reference: "This is the misconfiguration AppOmni found on 70% of tested instances"

Group by sensitivity. Show the worst ones first.

### 4. Knowledge Base Exposure
Specific to the AppOmni KB research:
- Which KBs are accessible without authentication
- Whether User Criteria is properly configured
- Whether the `block_access_with_no_user_criteria` property is set
- Article count at risk
- Reference: "AppOmni found 1,000+ instances exposing KB articles with active credentials"

### 5. Service Portal Attack Surface
- Public portals and pages
- Widgets exposing data to unauthenticated users
- The Simple List widget exploit path (AppOmni's documented attack)

### 6. Client-Callable Script Includes
- Script Includes callable via GlideAjax without ACL protection
- What data they can access
- Server-side code execution risk

### 7. Security Properties Assessment
Table showing each property, current value, recommended value, and risk:
| Property | Current | Recommended | Risk |
|----------|---------|-------------|------|

Color-coded: red for dangerous values, green for correct.

### 8. Instance Scan Validation
Cross-reference our findings with ServiceNow's own Instance Scan:
- Matching findings (our scan agrees with SN's)
- Additional findings (things we caught that SN didn't)
- Establishes dual validation

### 9. Remediation Priority
Ordered action plan:
1. Fix critical exposures (guest access to sensitive data)
2. Add roles to empty ACLs on sensitive tables
3. Set `glide.knowman.block_access_with_no_user_criteria` to true
4. Review and restrict public portal pages
5. Add ACLs to client-callable Script Includes
6. Fix security property misconfigurations
7. Review and resolve Instance Scan findings

Each with effort estimate and impact rating.

### 10. Reference
- Full list of empty ACLs with tables and sensitivity
- Full list of tables without ACLs
- Security properties detail
- Research source links with descriptions
- Methodology notes

## HTML Styling

Match the operatorzero.ai report style:
- Font: Inter, system fonts
- Background: #f5f6fa
- Cards: #ffffff, border-radius: 8px, box-shadow: 0 2px 12px rgba(0,0,0,0.08)
- Headers: #111827, 18px weight 600
- Body text: #6b7280 for muted, #111827 for emphasis, 13px, line-height: 1.6
- Accent blue: #2563eb
- Severity colors: Critical #dc2626, High #ea580c, Medium #ca8a04, Low #16a34a
- Max content width: 850px
- Cover: linear-gradient(135deg, #1e3a8a, #1d4ed8)
- Numbered sections with blue badge squares
- Research citations: distinct callout style with source link
- Sticky top nav
- No code blocks anywhere

## Output

Save as `acl-scan.html` in the current working directory. Open in browser.

## What This Skill Does NOT Do

- Does not attempt to exploit any vulnerability
- Does not access data through the vulnerabilities it finds
- Does not modify any ACLs or security configurations
- Does not test from an unauthenticated perspective (runs as admin)
- Does not guarantee completeness — complex ACL inheritance and scripted conditions may have edge cases
- Does not replace a formal penetration test

## The Pitch

"Security researchers found 70% of ServiceNow instances leaking data through misconfigured access controls. Run `/sn-acl-scan` to check if yours is one of them. Every finding references the published research — hand the report to your CISO."
