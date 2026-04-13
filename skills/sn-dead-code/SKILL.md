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

## Data Collection

Use whatever ServiceNow access method is available (REST API, GlideRecord scripts, MCP tools, SDK, etc.) to collect the following data. All queries reference standard ServiceNow tables.

### Phase 1: Discover What's On The Instance

- Get all tables with active business rules, client scripts, UI policies, UI actions
- Get counts of Script Includes, Workflows, Flows, Notifications, Scheduled Jobs
- Identify high-volume tables (incident, task, change_request, etc.) by record count — these are where unnecessary business rule evaluations cost the most

### Phase 2: Performance Killers (Tier 1)
These are actively hurting the instance right now.

**Global Business Rules:**
Query `sys_script` where active=true, collection=global (or table is empty/global).
Fields: name, script, sys_updated_on, sys_updated_by, sys_scope.
Every one of these loads on every page. They should be Script Includes.

**Business Rules Without Conditions on High-Volume Tables:**
For each high-volume table (incident, task, change_request, sc_request, sn_hr_core_case, etc.):
Query `sys_script` where collection=[table], active=true, filter_condition is empty, AND when is 'before' or 'after'.
These fire on every insert/update/delete with no filter — evaluate needlessly.

**Before BRs with Unbounded GlideRecord:**
Query `sys_script` where active=true, when='before', script CONTAINS 'GlideRecord' AND script does NOT contain 'setLimit'.
Before rules block the user — unbounded queries in before rules = worst case performance.

### Phase 3: Provably Dead Code (Tier 2)
Zero references, no trigger, mathematically dead.

**Script Includes Never Called:**
Get all active Script Includes (`sys_script_include` where active=true).
For each, search for the Script Include name/class name across ALL script-containing tables:
- `sys_script` (Business Rules)
- `sys_script_include` (other Script Includes)
- `sys_script_client` (Client Scripts)
- `sys_ui_action` (UI Actions)
- `sys_ui_script` (UI Scripts)
- `sys_processor` (Processors)
- `sys_script_fix` (Fix Scripts)
- `sys_ws_operation` (Web Service Operations)
- `sysauto_script` (Scheduled Jobs)
- `sys_hub_flow` (Flows — check action configs)
- `sys_hub_action_type_definition` (Flow Actions)
- `wf_workflow_version` (Workflow activities)
- `sp_widget` (Portal Widgets — client_script + script fields)
- `sys_ui_page` (UI Pages — processing_script + client_script)
- `sys_transform_script` (Transform Scripts)
- `sysevent_script_action` (Script Actions/Events)
- `sys_security_acl` (ACL scripts)

IMPORTANT: This is computationally expensive. Batch Script Includes and use CONTAINS queries.
Flag any with ZERO references. Note: GlideEvaluator and eval() can call Script Includes dynamically — add a caveat for Script Includes whose name matches common utility patterns.

**Orphaned Catalog Artifacts:**
Find catalog items where active=false. Then query these tables for records pointing at those inactive items that are still active:
- `item_option_new` (Variables) where cat_item IN [inactive items] AND active=true
- `catalog_script_client` where cat_item IN [inactive items] AND active=true
- `catalog_ui_policy` where catalog_item IN [inactive items] AND active=true
- `catalog_ui_policy_action` where ui_policy.catalog_item IN [inactive items]
Also check: active variables/scripts on catalog items not associated with any catalog.

**Workflows With No Trigger:**
Get all published workflow versions (`wf_workflow_version` where published=true).
For each workflow, search for references to the workflow's sys_id or name in:
- `sys_script` (BRs that call Workflow.startFlow)
- `sys_ui_action` (UI Actions that trigger workflows)
- `sys_hub_flow` (Flows that call subflows/workflows)
- `sysauto_script` (Scheduled jobs)
If zero references found, the workflow is published but nothing starts it.

**Notifications With No Recipients:**
Query `sysevent_email_action` where active=true. Check: recipient_fields is empty AND recipient_groups is empty AND recipient_users is empty AND advanced_condition or script doesn't dynamically set recipients. These generate email processing overhead but deliver to nobody.

**UI Policies With No Effect:**
Find UI Policies (`sys_ui_policy` where active=true) that have no associated UI Policy Actions (`sys_ui_policy_action` where ui_policy=[this]) AND script_true is empty AND script_false is empty. These evaluate conditions on every form load but do nothing when matched.

**Scheduled Jobs Always Failing:**
Query `sysauto_script` where active=true. Check `sys_trigger` for last execution results — if last N executions all failed, the job is active but broken. Also: scheduled jobs where run_as user is inactive.

### Phase 4: Orphaned References (Tier 3)
Things pointing at stuff that no longer exists.

**Business Rules on Non-Existent Tables:**
Get all active business rules (`sys_script` where active=true). Check if the collection (table) field references a table that exists in `sys_db_object`. If the table was deleted but the BR remains, it's orphaned.

**ACLs on Non-Existent Tables:**
Similar check for `sys_security_acl` — does the referenced table still exist?

**Broken Reference Fields:**
Check M2M tables for records where one side of the relationship points to a deleted record. Common: `m2m_sp_widget_dependency`, `sc_cat_item_catalog`, etc.

**Transform Maps With No Import Source:**
Query `sys_transform_map` where active=true. Cross-reference with `sys_data_source` — if the data source is inactive or deleted, the transform map is orphaned.

**Widgets Not On Any Page:**
Get all `sp_widget` records. Check if any `sp_instance` record references them (`sp_instance.sp_widget`=[widget sys_id]). Widgets with zero instances aren't used on any portal page. Caveat: widgets can be loaded dynamically via spUtil.get() — flag but note caveat.

### Phase 5: Suspicious (Lower Confidence)
These can't be proven dead but are worth flagging.

**Empty Script Bodies:**
Business Rules, Client Scripts, UI Actions where active=true AND script field is empty or contains only comments. These evaluate but do nothing.

**Fields Empty 95%+ of the Time:**
For custom fields (u_ prefix) on high-volume tables: count total records vs records where field is not empty. If <5% populated, the field may be unused clutter on forms.

**Active Configs By Inactive Users:**
Scheduled jobs, business rules, or flows where sys_created_by or run_as maps to an inactive user. May indicate abandoned work.

## Document Structure

### 1. Dead Code Summary
Hero card with headline numbers:
- **Performance Drags** — active findings hurting the instance now (Tier 1 count)
- **Provably Dead** — zero references, no trigger, mathematically dead (Tier 2 count)  
- **Orphaned References** — pointing at things that don't exist (Tier 3 count)
- **Suspicious** — can't prove dead, worth investigating (lower confidence count)

One-paragraph plain-English verdict with estimated cleanup effort.

### 2. Performance Killers — Fix These First
For each Tier 1 finding:
- What it is (plain English)
- Why it hurts (loads every page / fires on every transaction / blocks users)
- How often it fires (estimated from table volume)
- What to do (migrate to Script Include / add condition filter / add setLimit)

These aren't dead code — they're alive and actively wasting resources.

### 3. Provably Dead Code
For each category in Tier 2:
- Count of findings
- Evidence methodology ("searched N script tables for class name references, found zero")
- Top findings with names and last-modified dates
- Recommended action (deactivate, delete, or investigate)

### 4. Orphaned References  
For each category in Tier 3:
- What's broken (the reference target doesn't exist)
- Impact (clutter vs potential errors)
- Recommended action

### 5. Suspicious — Needs Investigation
Lower-confidence findings with caveats:
- Empty script bodies
- Barely-used custom fields
- Configs by inactive users
- Each with a note on why it's suspicious but not proven dead

### 6. Cleanup Priority Matrix
The "screenshot this" section:

| Priority | Category | Count | Impact | Effort |
|----------|----------|-------|--------|--------|
| 1 | Global BRs | N | Performance | Low — migrate to SI |
| 2 | Unconditioned BRs on busy tables | N | Performance | Low — add conditions |
| 3 | Orphaned catalog artifacts | N | Clutter + confusion | Medium — script cleanup |
| 4 | Dead Script Includes | N | Clutter + upgrade | Low — deactivate |
| ... | ... | ... | ... | ... |

### 7. Reference
Technical detail at the bottom:
- Full list of dead Script Includes with names and zero-reference proof
- Full list of orphaned catalog artifacts with parent item names
- Tables searched for cross-references (complete list)
- Data collection methodology

## HTML Styling

Match the operatorzero.ai report style:
- Font: Inter, system fonts
- Background: #f5f6fa
- Cards: #ffffff, border-radius: 8px, box-shadow: 0 2px 12px rgba(0,0,0,0.08)
- Headers: #111827, 18px weight 600
- Body text: #6b7280 for muted, #111827 for emphasis, 13px, line-height 1.6
- Accent blue: #2563eb
- Tier colors: Red #dc2626 for Tier 1 (performance), Orange #ea580c for Tier 2 (dead), Yellow #ca8a04 for Tier 3 (orphaned), Blue #2563eb for suspicious
- Max content width: 850px
- Stats values: 28px weight 700
- Cover: linear-gradient(135deg, #1e3a8a, #1d4ed8)
- Numbered sections with blue badge squares
- Sticky top nav
- No code blocks anywhere

## Output

Save as `dead-code.html` in the current working directory. Open in browser.

## Performance Notes

The Script Include cross-reference check is the most expensive operation — it searches every script-containing table for each Script Include name. On large instances:
- Batch the searches (search for multiple names in one query where possible)
- Use CONTAINS queries, not exact match (class names may appear in various contexts)
- Set reasonable limits — if there are 2,000+ Script Includes, sample or prioritize customer-created ones (sys_policy is empty, not 'protected' or 'read')
- Focus on customer Script Includes first (not OOB) — OOB ones are ServiceNow's responsibility

## What This Skill Does NOT Do

- Does not modify any records or configurations
- Does not deactivate or delete anything — only identifies
- Does not claim code is dead based on age alone (age proves nothing)
- Does not show script code in the narrative (reference section only)
- Does not use ServiceNow jargon in narrative sections
- Does not guarantee completeness — dynamic invocations (eval, GlideEvaluator) can't be statically detected
- Does not flag OOB/protected code as dead — focuses on customer customizations

## The Pitch

"How much dead code is on your ServiceNow instance? Run `/sn-dead-code`. It cross-references every script, workflow, and configuration to find what's active but provably doing nothing — zero references, no triggers, orphaned artifacts, and rules firing needlessly on every transaction. No existing tool does this."
