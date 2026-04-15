# Data Collection

Use whatever ServiceNow access method is available (REST API, GlideRecord scripts, MCP tools, SDK, etc.) to collect the following data. All queries reference standard ServiceNow tables. Parallelize everything possible.

## Instance Profile

- Get instance version via `gs.getProperty('glide.buildtag')`
- Active user count, users logged in last 30 days (query `sys_user`)

## What's Running (module detection)

Record counts for key tables: `incident`, `problem`, `change_request`, `sc_request`, `sc_req_item`, `kb_knowledge`, `cmdb_ci`, `ast_contract`, `alm_license`, `alm_asset`, `sn_customerservice_case`, `sn_hr_core_case`, `em_alert`, `sys_user`, `sys_user_group`, `task_sla`, `sc_cat_item`, `sp_portal`, `sp_widget`

## Process Health (per active module)

- State distributions — are records flowing through the lifecycle or piling up?
- Priority distributions — is the split reasonable or skewed?
- Assignment coverage — how many records are unassigned?
- Age analysis — how many records are stale (open > 90 days)?

## What's Custom

- Custom u_ tables — which exist in `sys_db_object`, which have data
- Custom u_ fields on OOB tables (query `sys_dictionary`)
- Scoped applications installed (query `sys_app` and `sys_store_app`)

## Automation & Integration Maturity

- Are they using flows (`sys_hub_flow`) or legacy workflows (`wf_workflow`) or both?
- How many integrations exist? What do they connect to?
- REST messages — query `sys_rest_message` for endpoints, auth types
- Transform maps — query `sys_transform_map` for what data flows in

## Who Built This

- Top `sys_created_by` values across config tables, with date ranges
- Recent activity — what changed in last 30 days
- Update sets in progress (query `sys_update_set` where state=in progress)

## What Needs Attention

- Notifications with no recipients (`sysevent_email_action`)
- Active configurations with no triggers (dormant)
- Test/debug artifacts still active
- Custom tables with no security controls (cross-ref `sys_db_object` with `sys_security_acl`)
- Active users past end date
- Stale admin accounts
- Key security properties via `gs.getProperty()`
