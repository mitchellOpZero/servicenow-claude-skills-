# Report Structure

Compose the report in this exact order. All HTML styling per `shared/report-templates/operatorzero-report-base.md` plus the tier-badge extension below.

## 1. Dead Code Summary
Hero card with headline numbers:
- **Performance Drags** — active findings hurting the instance now (Tier 1 count)
- **Provably Dead** — zero references, no trigger, mathematically dead (Tier 2 count)
- **Orphaned References** — pointing at things that don't exist (Tier 3 count)
- **Suspicious** — can't prove dead, worth investigating (lower confidence count)

One-paragraph plain-English verdict with estimated cleanup effort.

## 2. Performance Killers — Fix These First
For each Tier 1 finding:
- What it is (plain English)
- Why it hurts (loads every page / fires on every transaction / blocks users)
- How often it fires (estimated from table volume)
- What to do (migrate to Script Include / add condition filter / add setLimit)

These aren't dead code — they're alive and actively wasting resources.

## 3. Provably Dead Code
For each category in Tier 2:
- Count of findings
- Evidence methodology ("searched N script tables for class name references, found zero")
- Top findings with names and last-modified dates
- Recommended action (deactivate, delete, or investigate)

## 4. Orphaned References
For each category in Tier 3:
- What's broken (the reference target doesn't exist)
- Impact (clutter vs potential errors)
- Recommended action

## 5. Suspicious — Needs Investigation
Lower-confidence findings with caveats:
- Empty script bodies
- Barely-used custom fields
- Configs by inactive users
- Each with a note on why it's suspicious but not proven dead

## 6. Cleanup Priority Matrix
The "screenshot this" section:

| Priority | Category | Count | Impact | Effort |
|----------|----------|-------|--------|--------|
| 1 | Global BRs | N | Performance | Low — migrate to SI |
| 2 | Unconditioned BRs on busy tables | N | Performance | Low — add conditions |
| 3 | Orphaned catalog artifacts | N | Clutter + confusion | Medium — script cleanup |
| 4 | Dead Script Includes | N | Clutter + upgrade | Low — deactivate |
| ... | ... | ... | ... | ... |

## 7. Reference
Technical detail at the bottom:
- Full list of dead Script Includes with names and zero-reference proof
- Full list of orphaned catalog artifacts with parent item names
- Tables searched for cross-references (complete list)
- Data collection methodology

## Dead-code-specific style additions

On top of the shared base:
- Tier badges: Tier 1 red `#dc2626` (performance), Tier 2 orange `#ea580c` (dead), Tier 3 yellow `#ca8a04` (orphaned), Blue `#2563eb` (suspicious)

## What this skill does NOT do

- Does not modify any records or configurations
- Does not deactivate or delete anything — only identifies
- Does not claim code is dead based on age alone (age proves nothing)
- Does not show script code in the narrative (reference section only)
- Does not use ServiceNow jargon in narrative sections
- Does not guarantee completeness — dynamic invocations (eval, GlideEvaluator) can't be statically detected
- Does not flag OOB/protected code as dead — focuses on customer customizations

## The Pitch

"How much dead code is on your ServiceNow instance? Run `/sn-dead-code`. It cross-references every script, workflow, and configuration to find what's active but provably doing nothing — zero references, no triggers, orphaned artifacts, and rules firing needlessly on every transaction. No existing tool does this."
