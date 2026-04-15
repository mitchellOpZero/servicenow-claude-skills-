# Data Collection

Use whatever ServiceNow access method is available (REST API, GlideRecord scripts, MCP tools, SDK, etc.) to collect the following data. All queries reference standard ServiceNow tables.

## Phase 1: Instance Security Profile

Retrieve these system properties using `gs.getProperty()`:
- `glide.buildtag` — instance version
- `glide.security.use_csrf_token` — CSRF protection
- `glide.cms.catalog.public_access` — public catalog access
- `glide.ui.session_timeout` — session timeout
- `glide.basicauth.required.scriptedprocessor` — basic auth on processors
- `glide.script.block.client.globals` — client script security
- `glide.ui.escape_all_script` — XSS protection
- `glide.security.strict.updates` — strict update security

## Phase 2: Empty ACLs (THE MAIN VULNERABILITY)
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

## Phase 3: Tables Without ANY ACLs

Get all tables from `sys_db_object` that have records. Cross-reference with `sys_security_acl` to find tables with ZERO ACLs. These have no access control whatsoever.

Filter to tables that actually matter:
- Have records (not empty schema tables)
- Are not system/internal tables
- Focus on custom tables (u_ prefix) — these are most commonly unprotected
- Check task-extended tables, cmdb-extended tables

## Phase 4: Knowledge Base Exposure (AppOmni's primary finding)

Query `kb_knowledge_base` for all active knowledge bases (fields: title, active, kb_version).

For each KB:
- Check if User Criteria allows public/guest access
- Check if "Can Read" user criteria is set or empty
- Check if any ACL on `kb_knowledge` restricts access
- Count published articles per KB

Check if KB articles are accessible via the Simple List widget without authentication — this is the exact exploit path AppOmni documented. Query `sp_widget` where id='widget-simple-list' and check if it exposes kb_knowledge.

Check `gs.getProperty('glide.knowman.block_access_with_no_user_criteria')` — if false, KBs without user criteria are publicly accessible.

## Phase 5: Service Portal Exposure

Query `sp_portal` for all active portals (fields: title, url_suffix, login_page, public). Check which portals allow unauthenticated access.

Query `sp_page` where public=true. Check for pages that don't require authentication.

For public portal pages, check which widgets are rendered and what data those widgets expose. Focus on: Simple List, Data Table, Record List widgets that may expose table data to unauthenticated users.

## Phase 6: Client-Callable Script Includes

Query `sys_script_include` where client_callable=true AND active=true (fields: name, api_name, access, sys_scope).

For each client-callable Script Include:
- Check if a corresponding ACL exists on sys_script_include.[name]
- If no ACL → any authenticated user can call it via GlideAjax
- Check if the script accesses sensitive data (GlideRecord queries on sensitive tables)
- This is a server-side code execution vector

## Phase 7: Guest User and Public Access Scope

Check the guest user account (`sys_user` where user_name='guest') — is it active? What roles does it have?

Check `sys_user_role` for the 'public' role — what tables/features does this role grant access to?

Check for ACLs that explicitly grant access to 'public' or no role — these are intentionally or accidentally public-facing.

## Phase 8: Security Properties Audit

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

## Phase 9: Instance Scan Security Findings

Pull from existing `scan_finding` data — these are ServiceNow's own security checks:
- Deactivated Security Controls
- Empty ACLs (ServiceNow's own check)
- Tables without ACLs
- Public portal pages
- UI Pages without ACLs
- Client-callable Script Includes without ACLs
- Users with local passwords

Group by priority. These validate our own findings with SN's built-in tooling.
