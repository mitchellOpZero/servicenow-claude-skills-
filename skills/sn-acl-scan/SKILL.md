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

## How to use this skill

This skill is a router. Read each topic file only when you reach that phase.

| Phase | Read this |
|---|---|
| Run all 9 collection phases | `topics/01-data-collection.md` |
| Compose 10 report sections + apply scope rules | `topics/02-report-structure.md` |
| Apply HTML styling | `../../shared/report-templates/operatorzero-report-base.md` |

## Output

Save as `acl-scan.html` in the current working directory. Open in browser.
