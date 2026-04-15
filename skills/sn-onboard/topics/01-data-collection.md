# Data Collection

Use whatever ServiceNow access method is available (REST API, GlideRecord scripts, MCP tools, SDK, etc.) to collect the following data. All queries reference standard ServiceNow tables. Parallelize everything possible.

> **Read the Critical Rules in SKILL.md first.** The filters below are not optional — they exist because raw counts of `sys_script`, `sys_dictionary`, etc. are dominated by OOB content and produce misleading "customization" insights without scope filtering.

## Instance Profile

- Instance version via `gs.getProperty('glide.buildtag')`
- Active user count: `sys_user` where `active=true`
- Users logged in last 30 days: `sys_user` where `last_login_time>javascript:gs.daysAgoStart(30)`
- Admin count: `sys_user` where `active=true^roles=admin`

## What's Running (module detection)

Record counts for key tables: `incident`, `problem`, `change_request`, `sc_request`, `sc_req_item`, `kb_knowledge`, `cmdb_ci`, `ast_contract`, `alm_license`, `alm_asset`, `sn_customerservice_case`, `sn_hr_core_case`, `em_alert`, `sys_user`, `sys_user_group`, `task_sla`, `sc_cat_item`, `sp_portal`, `sp_widget`

Skip modules whose count is 0 or whose plugin returns an error — those plugins aren't installed.

## Process Health (per active module)

For each module with records:

1. **Resolve state labels first.** Query `sys_choice` where `name=<table>^element=state^inactive=false`, sort by `sequence`. Build a `value → label` map. Apply this when describing state distributions — never use the raw integer in the narrative.
2. State distributions: stats grouped by `state`.
3. Priority distributions: stats grouped by `priority` (also resolve via `sys_choice` if reporting labels).
4. Unassigned active records: `assigned_toISEMPTY^active=true` count.
5. Stale open records: `active=true^opened_at<javascript:gs.daysAgoStart(90)` count.

## What's Custom — filter for actual customization

This section's purpose is to characterize what humans on this instance built. **Without scope filtering, every count is dominated by OOB and shipped-app content.**

- **Custom u_ tables:** `sys_db_object` where `nameSTARTSWITHu_`. List name, label, super_class, sys_created_by, sys_created_on. Note: many u_ tables on demo/PDI instances are throwaway test artifacts (timestamp suffixes, `u_test*`, `u_verb_test_*`) — call those out as clutter rather than customization.
- **Custom u_ fields on OOB tables:** `sys_dictionary` where `elementSTARTSWITHu_^sys_scopeNAME=Global` (excludes fields shipped by store apps). For better signal, also exclude `sys_packageDIFFERENT FROM` known OOB packages.
- **Custom apps (intentional):** `sys_app` where `active=true^scopeNOT INglobal`. Exclude scopes belonging to installed store apps.
- **Installed store apps:** `sys_store_app` where `active=true` (these are NOT customization — they're third-party additions).

When you report "X custom configurations exist," scope-filter the underlying table (`sys_script`, `sys_script_client`, `sys_ui_policy`, `sys_ui_action`, `sys_script_include`, `sys_security_acl`) by `sys_scope!=global` or by `sys_package` not in the OOB package list. Without that filter you are reporting the size of the OOB platform.

## Automation & Integration Maturity

- Flows: `sys_hub_flow` where `active=true`, group by `type` (flow vs subflow).
- Legacy workflows: `wf_workflow` where `active=true`.
- REST messages: `sys_rest_message` — fields `name`, `rest_endpoint`, `authentication_type`, `sys_scope`. **Skip rows where `rest_endpoint` is empty** — describe them as "configured but no endpoint set" rather than narrating around them.
- Transform maps: `sys_transform_map` where `active=true`.

When describing integrations: only call something a "real integration" if it has a populated endpoint AND non-default auth (not `no_authentication`). Default-auth + demo URLs (jsonplaceholder, yahoo finance, example.com) are demo content — say so.

## Who Built This

- **Don't read names directly off `sys_created_by` aggregates from `sys_script` etc.** Those tables include OOB rows authored by ServiceNow employees during plugin development.
- Process: aggregate sys_created_by from custom-scope rows only (filter by `sys_scope!=global`), then for each candidate username, query `sys_user` where `user_name=<name>^active=true` AND check `last_login_time` is non-null. Only list users who actually exist on this instance and have logged in.
- Recent activity (last 30 days): scope-filtered count of new BRs / scripts / configurations created.
- Update sets in progress: `sys_update_set` where `state=in progress`.

## What Needs Attention

- Notifications with no recipients: `sysevent_email_action` where `active=true^recipient_fieldsISEMPTY^recipient_groupsISEMPTY^recipient_usersISEMPTY`. Compare to total active notifications for the percentage.
- Dormant active configurations: BRs/flows that are active but have no executions in N days (cross-ref with execution tables if available).
- Test/debug artifacts: u_ tables with timestamp suffixes, names containing `test`, `debug`, `temp`, `verb_test`, `curl_test`.
- Custom tables with no security controls: u_ tables in `sys_db_object` with no matching rows in `sys_security_acl`.
- Active users past end date: `sys_user` where `active=true^u_end_dateISNOTEMPTY^u_end_date<javascript:gs.now()` (or whatever end-date field exists).
- Stale users: `sys_user` where `active=true^last_login_time<javascript:gs.daysAgoStart(90)`.
- Stale admin accounts: same as above with `^roles=admin`.
- Key security properties via `gs.getProperty()`.
