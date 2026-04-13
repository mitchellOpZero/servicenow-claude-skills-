# OperatorZero ServiceNow Skills for Claude Code

Free ServiceNow instance scanning tools built as [Claude Code](https://claude.ai/claude-code) skills.

Drop these into your Claude Code skills directory, connect to a ServiceNow instance, and run them. Each skill scans your instance and generates a professional HTML report.

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

### 1. Copy skills to your Claude Code skills directory

```bash
git clone https://github.com/mitchellOpZero/servicenow-claude-skills-.git
cp -r servicenow-claude-skills-/skills/* ~/.claude/skills/
```

### 2. Connect to your ServiceNow instance

These skills need a way to query your ServiceNow instance. They're tool-agnostic — use whatever connection method works for your setup:

- **ServiceNow REST API** — Claude Code can call the Table API, Scripted REST APIs, or GlideRecord via the Fluent API
- **Anthropic SDK** — build a connection layer with the Claude SDK
- **Any MCP server** that provides ServiceNow table query and script execution capabilities

The skill files describe WHAT data to collect (specific tables, fields, and query patterns). Claude Code figures out HOW to collect it based on whatever tools are available in your environment.

### 3. Run a skill

```
/sn-onboard
/sn-dead-code
/sn-acl-scan
```

Each generates an HTML report and opens it in your browser.

## Requirements

- [Claude Code](https://claude.ai/claude-code) (CLI, desktop app, or IDE extension)
- A ServiceNow instance (dev, test, or prod)
- A way to query the instance from Claude Code (REST API, SDK, MCP server, etc.)
- Admin or read-access role on the instance

## How It Works

Each skill is a `SKILL.md` file — a detailed prompt that tells Claude Code exactly what data to collect, how to analyze it, and how to format the output. When you run a skill:

1. Claude Code reads the SKILL.md instructions
2. Queries your ServiceNow instance using available tools
3. Analyzes the data (cross-referencing tables, scoring findings, categorizing by severity)
4. Generates a styled HTML report
5. Opens it in your browser

No plugins, no store apps, no installation on your ServiceNow instance. Read-only — nothing is modified.

## Sample Reports

The `sample-reports/` directory contains example outputs from real instance scans so you can see what the reports look like before running them yourself.

## More Skills

These 3 are open source. We have 50+ more at [operatorzero.ai](https://operatorzero.ai) covering license optimization, upgrade readiness, SLA validation, CMDB integrity, notification auditing, form performance profiling, change compliance, storage analysis, and more.

operatorZero is an AI-native ServiceNow partner — we build tools that give platform teams superpowers. If you're spending days on assessments that should take minutes, come talk to us.

## Contributing

Found a bug or have an idea for a check? Open an issue or PR.

## License

MIT
