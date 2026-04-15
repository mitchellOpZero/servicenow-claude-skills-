# Data Collection

Use whatever ServiceNow access method is available (REST API, GlideRecord scripts, MCP tools, SDK, etc.) to collect the following data. All queries reference standard ServiceNow tables.

## Phase 1: Discover What's On The Instance

- Get all tables with active business rules, client scripts, UI policies, UI actions
- Get counts of Script Includes, Workflows, Flows, Notifications, Scheduled Jobs
- Identify high-volume tables (incident, task, change_request, etc.) by record count — these are where unnecessary business rule evaluations cost the most

## Phase 2: Performance Killers (Tier 1)
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

## Phase 3: Provably Dead Code (Tier 2)
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

## Phase 4: Orphaned References (Tier 3)
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

## Phase 5: Suspicious (Lower Confidence)
These can't be proven dead but are worth flagging.

**Empty Script Bodies:**
Business Rules, Client Scripts, UI Actions where active=true AND script field is empty or contains only comments. These evaluate but do nothing.

**Fields Empty 95%+ of the Time:**
For custom fields (u_ prefix) on high-volume tables: count total records vs records where field is not empty. If <5% populated, the field may be unused clutter on forms.

**Active Configs By Inactive Users:**
Scheduled jobs, business rules, or flows where sys_created_by or run_as maps to an inactive user. May indicate abandoned work.
