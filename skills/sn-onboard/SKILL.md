# ServiceNow Instance Orientation

Generate a concise, conversational orientation guide for someone new to a ServiceNow instance. This is the "here's what you need to know" document — written for a manager, new stakeholder, or anyone taking ownership. 15-minute read, no fluff.

## Arguments
- No arguments required — always scans the whole instance
- Optional: `html` (default) or `md` for output format

## Core Principle

**Insights, not inventory.**

Nobody cares that there are 54 active business rules on the incident table. What they care about is: "The incident process has custom VIP routing and an unfinished AI categorization feature. Priority is miscalibrated — 42% of tickets are marked critical."

Every piece of data you collect should be translated into what it MEANS, not what it COUNTS. Configuration counts belong in a small reference table at the bottom, not in the narrative.

## What Makes a Good Insight

| Bad (inventory) | Good (insight) |
|---|---|
| "54 active business rules on incident" | "The incident process has significant customization, mostly around VIP handling" |
| "79 BRs on change_request" | "Change management uses the full model framework — Normal, Standard, and Emergency paths are all configured with approval gates" |
| "22 active client scripts" | "The incident form has custom behavior — VIP callers get visual indicators and certain fields auto-populate from the caller record" |
| "197 notifications have no recipients" | "About half the email notifications on this instance aren't reaching anyone" |
| "116 dormant business rules" | "There are configurations that have been parked — active but not actually running. Some look like abandoned experiments" |
| "5,630 total active BRs" | "This is a standard-sized instance with moderate customization" |

The insight is what the configuration tells you about how the instance is being used and what state it's in.

## Tone

Conversational and direct. Like a knowledgeable colleague briefing you over coffee. Use "you" and "your." Short sentences. No jargon without a plain-English explanation. No ServiceNow technical terms without context.

Never say: "business rule", "client script", "UI policy", "UI action", "ACL", "sys_id", "GlideRecord"
Instead say: "custom configuration", "customization", "form behavior", "field rules", "buttons/actions", "security controls", "record ID"

**"Automation" vs "Configuration":** Only use "automation" for things that genuinely run on their own — flows, workflows, scheduled jobs. Business rules, client scripts, and UI policies are configurations on a table, not automations. Say "custom configuration" or "customization" instead.

The ONLY place technical terms appear is in the small reference table at the bottom (for the developers the manager will hand this to).

## Data Collection

Use whatever ServiceNow access method is available (REST API, GlideRecord scripts, MCP tools, SDK, etc.) to collect the following data. All queries reference standard ServiceNow tables. Parallelize everything possible.

### Instance Profile

- Get instance version via `gs.getProperty('glide.buildtag')`
- Active user count, users logged in last 30 days (query `sys_user`)

### What's Running (module detection)

Record counts for key tables: `incident`, `problem`, `change_request`, `sc_request`, `sc_req_item`, `kb_knowledge`, `cmdb_ci`, `ast_contract`, `alm_license`, `alm_asset`, `sn_customerservice_case`, `sn_hr_core_case`, `em_alert`, `sys_user`, `sys_user_group`, `task_sla`, `sc_cat_item`, `sp_portal`, `sp_widget`

### Process Health (per active module)

- State distributions — are records flowing through the lifecycle or piling up?
- Priority distributions — is the split reasonable or skewed?
- Assignment coverage — how many records are unassigned?
- Age analysis — how many records are stale (open > 90 days)?

### What's Custom

- Custom u_ tables — which exist in `sys_db_object`, which have data
- Custom u_ fields on OOB tables (query `sys_dictionary`)
- Scoped applications installed (query `sys_app` and `sys_store_app`)

### Automation & Integration Maturity

- Are they using flows (`sys_hub_flow`) or legacy workflows (`wf_workflow`) or both?
- How many integrations exist? What do they connect to?
- REST messages — query `sys_rest_message` for endpoints, auth types
- Transform maps — query `sys_transform_map` for what data flows in

### Who Built This

- Top `sys_created_by` values across config tables, with date ranges
- Recent activity — what changed in last 30 days
- Update sets in progress (query `sys_update_set` where state=in progress)

### What Needs Attention

- Notifications with no recipients (`sysevent_email_action`)
- Active configurations with no triggers (dormant)
- Test/debug artifacts still active
- Custom tables with no security controls (cross-ref `sys_db_object` with `sys_security_acl`)
- Active users past end date
- Stale admin accounts
- Key security properties via `gs.getProperty()`

## Document Structure

### 1. Instance at a Glance
Card grid with 6 high-level facts:
- Instance name + version
- Environment type (dev/test/prod — inferred from login activity)
- User activity (active users / recent logins)
- Active modules (plain names: "Incident, Change, Knowledge, CMDB...")
- Customization level (light / moderate / heavy)
- One-sentence vibe ("A lightly customized dev instance with active knowledge management and an approval bottleneck in change management")

### 2. What's Running Here
For each active module (only those with records), write a **3-5 sentence insight block**. NO configuration counts in the narrative.

Focus on:
- **What it does** in plain English (1 sentence)
- **How it's being used** — record volumes, lifecycle health (are things flowing or stuck?)
- **What's been customized** — describe the intent, not the mechanism ("VIP callers get special routing" not "4 business rules handle VIP")
- **The headline** — the one thing someone should know ("42% of incidents are marked critical" / "61% of changes are stuck waiting for approval" / "the service catalog has 500 items but almost no one uses it")

Skip modules with zero records. Mention them in a footnote.

### 3. Who's Been Working on This
Narrative paragraphs (not tables):
- Who did meaningful custom work (exclude system/maintenance accounts)
- When the work happened (timeline feel — "most customization happened in late 2025")
- What they were building (inferred from naming patterns and tables touched)
- How active things are now (recent changes)
- Update sets in progress — what's mid-flight, what's abandoned

### 4. What's Custom
Plain-English assessment:
- Custom tables — how many, which matter (have data), which are leftovers
- Custom fields — which standard areas have been extended
- Custom apps — any scoped applications installed
- Overall assessment: "This instance is [lightly/moderately/heavily] customized. Most of the custom work is in [area]. This is [good/concerning] for long-term maintenance."

### 5. What Needs Attention
**Cap at top 10.** Each finding gets:
- A short title (no technical terms)
- Where / what it affects
- Why it matters — one sentence, business impact
- What to do — one sentence

Group by urgency:
- 🔴 Fix soon — things that are broken or expose data
- 🟡 Clean up — things that add clutter or confusion  
- 🔵 Be aware — not broken but worth knowing about

End with: "For a full technical assessment, run `/sn-tech-debt-scan`."

### 6. How Data Flows In and Out
Describe each integration in plain English:
- What system it connects to (inferred from endpoint/name)
- What it does (sends data, receives data, syncs)
- How secure the connection is (one line)

For data imports: what data comes in, where it goes.

Keep short. If there are no integrations, say "This instance is standalone — no active integrations with external systems."

### 7. Security & Access
Quick health check in plain English:
- Who has admin access and whether that's reasonable
- Whether user accounts are well-maintained
- Whether custom data is properly secured
- Whether key security settings are configured
- One-line bottom line: "This instance has the security posture of a [dev/production] environment."

### 8. Day One Cheat Sheet
The "screenshot this" section. Punchy bullets in four groups:
- **The basics** — 5-6 key facts
- **Things that will surprise you** — 3-4 things that aren't obvious
- **Don't touch until you understand** — 2-3 areas that need context before changes
- **Who to ask** — inferred from contributor data

### 9. Reference (small, at the bottom)
A compact table with the actual numbers — for the developers the manager will pass this to:

| Module | Records | Customizations | Custom Fields | Integrations |
|--------|---------|---------------|---------------|-------------|

This is the ONLY place configuration counts appear. Keep it small. Label columns in plain English ("Customizations" not "Business Rules + Client Scripts + UI Policies").

## HTML Styling

Same light-mode base as other opZero reports:
- Background: #f8f9fb
- Cards: #ffffff, border: 1px solid #d1d9e0, box-shadow: 0 1px 3px rgba(0,0,0,0.04)
- Headers: #1f2328
- Body text: #424a53, line-height 1.8 for narratives
- Secondary: #656d76
- Accent: #0969da
- Max content width: 780px
- Generous whitespace between sections
- Module blocks: card-style with colored left borders
- Attention items: 🔴🟡🔵 emoji indicators
- Cheat sheet: distinct background (#f0f4f8), compact
- Reference table: small font, muted styling, clearly labeled as a technical reference
- Sticky top nav
- No code blocks anywhere

## Output

Save as `instance-orientation.html` in the current working directory. Open in browser.

## What This Skill Does NOT Do

- Does not show scripts or code
- Does not list individual configuration items
- Does not use ServiceNow jargon in the narrative
- Does not provide technical remediation steps
- Does not document process flows
- Configuration counts only appear in the reference table at the bottom

## The Pitch

"New to this instance? Run `/sn-onboard`. Fifteen minutes later you'll know what's here, who built it, what's working, what's stuck, and what needs attention. No ServiceNow expertise required to read it."
