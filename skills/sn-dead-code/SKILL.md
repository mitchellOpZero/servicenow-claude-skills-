# ServiceNow Dead Code Scanner

Scan a ServiceNow instance and find code and configurations that are active but provably doing nothing — burning evaluation cycles, cluttering troubleshooting, or pointing at things that no longer exist. Everything in this report is verifiable: zero references, no trigger path, impossible conditions, broken targets.

## Arguments
- No arguments required — always scans the whole instance
- Optional: `html` (default) or `md` for output format

## Core Principle

**Prove it's dead, or don't call it dead.**

Every finding must be backed by evidence: zero references across all script fields, a target table that doesn't exist, a condition that can never match, a workflow with no trigger. Age alone proves nothing — a business rule written 3 years ago could be running perfectly every day. If you can't prove it's dead, it goes in a "suspicious" section with lower confidence, not the main findings.

## Critical Rules

### Only report what you can prove
- "Zero references" means you searched every script-containing table and found nothing. List what you searched.
- "No trigger path" means no business rule, UI action, flow, or scheduled job starts this workflow.
- "Orphaned" means the parent record is inactive or deleted, but child records are still active.
- Never say "likely unused" in Tier 1 — say "zero references found across [N] script tables" or "target table [X] does not exist."

### Discover, don't assume
- Don't hardcode which tables or modules to check — discover what's active on the instance first.
- Scan all script-containing tables for cross-references, not just the obvious ones.
- Every instance is different — the checks are universal but the findings are unique.

### Separate performance impact from clutter
- Global Business Rules loading on every page = active performance drag
- Unused Script Include sitting in a table = zero runtime cost, clutter only
- The report should clearly distinguish between "this is hurting you right now" and "this is just taking up space"

### Group by impact, not by technical type
- Don't show "12 dead Script Includes, 8 orphaned catalog scripts, 3 broken workflows"
- Show "Tier 1: 4 findings actively hurting performance, Tier 2: 23 findings provably dead, Tier 3: 11 orphaned references"

## Performance Notes (constraints on running this skill)

The Script Include cross-reference check is the most expensive operation — it searches every script-containing table for each Script Include name. On large instances:
- Batch the searches (search for multiple names in one query where possible)
- Use CONTAINS queries, not exact match (class names may appear in various contexts)
- Set reasonable limits — if there are 2,000+ Script Includes, sample or prioritize customer-created ones (sys_policy is empty, not 'protected' or 'read')
- Focus on customer Script Includes first (not OOB) — OOB ones are ServiceNow's responsibility

## How to use this skill

This skill is a router. Read each topic file only when you reach that phase.

| Phase | Read this |
|---|---|
| Run all 5 collection phases | `topics/01-data-collection.md` |
| Compose 7 report sections + apply scope rules | `topics/02-report-structure.md` |
| Apply HTML styling | `../../shared/report-templates/operatorzero-report-base.md` |

## Output

Save as `dead-code.html` in the current working directory. Open in browser.
