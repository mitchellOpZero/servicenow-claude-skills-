# OperatorZero ServiceNow Skills for Claude Code

Free ServiceNow instance scanning tools built as [Claude Code](https://claude.ai/claude-code) skills. Designed to pair with the official **[ServiceNow SDK](https://www.servicenow.com/docs/r/application-development/servicenow-sdk/servicenow-sdk-landing.html)** and its `fluent` plugin (released April 15, 2026 as part of ServiceNow's *Build Anywhere* initiative).

- **The fluent plugin** lets you *build* on ServiceNow from Claude Code (Business Rules, Tables, Flows, Script Includes, etc.).
- **These skills** let you *audit and understand* what's already on your instance.

Together they cover the full developer loop: see what's there → fix or extend it.

## Skills

### `/sn-onboard` — Instance Orientation
New to an instance? Get a plain-English orientation guide: what's running, who built it, what's customized, what needs attention. Written for a manager or new stakeholder — no ServiceNow expertise required to read it.

**What it covers:**
- Active modules and how they're being used
- Process health (are things flowing or stuck?)
- Who built the customizations and when
- What needs attention (capped at top 10 findings)
- Integration inventory
- Security posture
- Day one cheat sheet

[View sample report](sample-reports/instance-orientation.html)

---

### `/sn-dead-code` — Dead Code Scanner
Find every script, workflow, and configuration that's active but provably doing nothing. Cross-references every script table on the instance to prove zero references — no existing tool does this.

**What it finds:**
- **Performance killers** — Global Business Rules loading on every page (ServiceNow's own docs say not to use them — they ship 79 OOB), Before rules without conditions on busy tables firing needlessly on every transaction
- **Provably dead code** — Script Includes with zero references across all script tables, workflows nothing triggers, notifications sending to nobody, orphaned catalog artifacts on inactive items, empty business rules
- **Orphaned references** — Business rules on tables that don't exist, widgets not placed on any page

Every finding is backed by evidence — zero references, non-existent targets, empty script bodies. Not "this looks old." Provably dead.

[View sample report](sample-reports/dead-code.html)

---

### `/sn-acl-scan` — ACL Security Scanner
Check your instance for the same access control misconfigurations that security researchers found on 70% of enterprise instances. Every finding references the published research.

**What it checks:**
- **Empty ACLs** — no required role, no condition, no script (defaults to allow everyone including guests). This is the exact vulnerability [AppOmni documented](https://appomni.com/ao-labs/servicenow-knowledge-bases-data-exposures-uncovered/).
- **Tables with zero ACLs** — no access control whatsoever
- **Client-callable Script Includes without ACLs** — callable via GlideAjax by any authenticated user
- **Knowledge Base exposure** — the vulnerability that exposed PII and credentials on [1,000+ enterprise instances](https://www.bleepingcomputer.com/news/security/over-1-000-servicenow-instances-found-leaking-corporate-kb-data/)
- **Public portal pages** — pages accessible without authentication
- **Guest user status** — is the unauthenticated access path active?
- **Security properties** — 11 critical properties checked against recommended values

Every finding ties back to published research: [AppOmni](https://appomni.com/ao-labs/servicenow-knowledge-bases-data-exposures-uncovered/), [CVE-2025-3648](https://www.cio.com/article/4019819/warning-to-servicenow-admins-fix-your-access-control-lists-now.html), [BleepingComputer](https://www.bleepingcomputer.com/news/security/over-1-000-servicenow-instances-found-leaking-corporate-kb-data/), [Threatpost](https://threatpost.com/most-servicenow-instances-misconfigured-exposed/178827/).

[View sample report](sample-reports/acl-scan.html)

---

## Installation

### 1. Install the ServiceNow SDK + fluent plugin (recommended)

The cleanest way to give Claude Code an authenticated path into your ServiceNow instance is the official SDK and its `fluent` Claude Code plugin.

**Install the SDK** (Node 20+ required):
```bash
npm install -g @servicenow/sdk@latest
npx @servicenow/sdk --version   # confirm 4.6.0 or higher
```

**Install the fluent plugin in Claude Code:**
```
/plugin marketplace add servicenow/sdk
/plugin install fluent
/reload-plugins
```

This gives Claude Code two skills out of the box:
- `now-sdk-setup` — environment bootstrap
- `now-sdk-explain` — JIT documentation lookup over the SDK's ~180 topic catalog (Business Rules, Tables, Flows, ACLs, Script Includes, etc.)

Authenticate to your instance once via `npx @servicenow/sdk deploy` — credentials are cached locally.

> **Other connection methods** also work — these skills are tool-agnostic. They describe *what* data to collect; Claude Code figures out *how* using whatever's available (REST API, GlideRecord scripts, MCP servers, etc.).

### 2. Install these audit skills

```bash
git clone https://github.com/mitchellOpZero/servicenow-claude-skills-.git
cp -r servicenow-claude-skills-/skills/* ~/.claude/skills/
cp -r servicenow-claude-skills-/shared ~/.claude/skills/_shared    # report templates referenced by the skills
```

### 3. Run a skill

```
/sn-onboard
/sn-dead-code
/sn-acl-scan
```

Each generates an HTML report and opens it in your browser.

## Requirements

- [Claude Code](https://claude.ai/claude-code) (CLI, desktop app, or IDE extension)
- Node.js 20+ (for the ServiceNow SDK)
- A ServiceNow instance running Zurich or later (or a free PDI)
- Admin or read-access role on the instance

## How It Works — Architecture

These skills follow the **router + topic corpus** pattern that ServiceNow itself uses for the new `now-sdk-explain` skill. Each skill is split into:

```
skills/<skill>/
├── SKILL.md                    ← thin router (~60 lines): trigger,
│                                 Core Principle, Critical Rules, and
│                                 a routing table pointing to topics
└── topics/
    ├── 01-data-collection.md   ← all sn_script collection phases
    └── 02-report-structure.md  ← report sections + scope + pitch

shared/report-templates/        ← cross-skill HTML style specs
├── operatorzero-report-base.md
└── operatorzero-onboarding-style.md
```

When a skill triggers, only `SKILL.md` (~2KB) lands in context. The agent reads each topic file **only when it reaches that phase**. This keeps per-trigger context cost ~80% lower than monolithic skill files and makes report templates reusable across skills.

When you run a skill:

1. Claude Code reads the slim `SKILL.md` router and the relevant phase topics
2. Queries your ServiceNow instance (via the SDK, REST API, MCP server, or whatever's available)
3. Analyzes the data (cross-referencing tables, scoring findings, categorizing by severity)
4. Generates a styled HTML report using the shared template
5. Opens it in your browser

No plugins, no store apps, no installation on your ServiceNow instance. Read-only — nothing is modified.

## Companion Tools

| Need | Use |
|---|---|
| Build Business Rules, Tables, Flows, Script Includes | [ServiceNow SDK + `fluent` plugin](https://www.servicenow.com/docs/r/application-development/servicenow-sdk/servicenow-sdk-landing.html) |
| Look up SDK / Fluent API docs from Claude Code | `now-sdk-explain` (auto-triggers) |
| Audit what's already on the instance | These skills (`/sn-onboard`, `/sn-dead-code`, `/sn-acl-scan`) |

## Sample Reports

The `sample-reports/` directory contains example outputs from real instance scans so you can see what the reports look like before running them yourself.

## More Skills

These 3 are open source. We have 50+ more at [operatorzero.ai](https://operatorzero.ai) covering license optimization, upgrade readiness, SLA validation, CMDB integrity, notification auditing, form performance profiling, change compliance, storage analysis, and more — all built on the same router + corpus pattern.

operatorZero is building tools that give platform teams superpowers. If you're spending days on assessments that should take minutes, come talk to us.

## Contributing

Found a bug or have an idea for a check? Open an issue or PR.

## License

MIT
