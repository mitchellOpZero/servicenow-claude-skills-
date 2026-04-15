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

## Tone (strict — voice is the entire point of this skill)

Conversational and direct. Like a knowledgeable colleague briefing you over coffee. Use "you" and "your." Short sentences. No jargon without a plain-English explanation. No ServiceNow technical terms without context.

Never say: "business rule", "client script", "UI policy", "UI action", "ACL", "sys_id", "GlideRecord"
Instead say: "custom configuration", "customization", "form behavior", "field rules", "buttons/actions", "security controls", "record ID"

**"Automation" vs "Configuration":** Only use "automation" for things that genuinely run on their own — flows, workflows, scheduled jobs. Business rules, client scripts, and UI policies are configurations on a table, not automations. Say "custom configuration" or "customization" instead.

The ONLY place technical terms appear is in the small reference table at the bottom (for the developers the manager will hand this to).

## How to use this skill

This skill is a router. Read each topic file only when you reach that phase.

| Phase | Read this |
|---|---|
| Collect instance data | `topics/01-data-collection.md` |
| Compose the 9-section report + apply scope rules | `topics/02-report-structure.md` |
| Apply HTML styling | `../../shared/report-templates/operatorzero-onboarding-style.md` |

## Output

Save as `instance-orientation.html` in the current working directory. Open in browser.
