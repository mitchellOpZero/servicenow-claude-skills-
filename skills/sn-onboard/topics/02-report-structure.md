# Report Structure

Compose the report in this exact order. All HTML styling per `shared/report-templates/operatorzero-onboarding-style.md`. Remember the strict Tone rules in SKILL.md — no ServiceNow jargon outside the final reference table.

## 1. Instance at a Glance
Card grid with 6 high-level facts:
- Instance name + version
- Environment type (dev/test/prod — inferred from login activity)
- User activity (active users / recent logins)
- Active modules (plain names: "Incident, Change, Knowledge, CMDB...")
- Customization level (light / moderate / heavy)
- One-sentence vibe ("A lightly customized dev instance with active knowledge management and an approval bottleneck in change management")

## 2. What's Running Here
For each active module (only those with records), write a **3-5 sentence insight block**. NO configuration counts in the narrative.

Focus on:
- **What it does** in plain English (1 sentence)
- **How it's being used** — record volumes, lifecycle health (are things flowing or stuck?)
- **What's been customized** — describe the intent, not the mechanism ("VIP callers get special routing" not "4 business rules handle VIP")
- **The headline** — the one thing someone should know ("42% of incidents are marked critical" / "61% of changes are stuck waiting for approval" / "the service catalog has 500 items but almost no one uses it")

Skip modules with zero records. Mention them in a footnote.

## 3. Who's Been Working on This
Narrative paragraphs (not tables):
- Who did meaningful custom work (exclude system/maintenance accounts)
- When the work happened (timeline feel — "most customization happened in late 2025")
- What they were building (inferred from naming patterns and tables touched)
- How active things are now (recent changes)
- Update sets in progress — what's mid-flight, what's abandoned

## 4. What's Custom
Plain-English assessment:
- Custom tables — how many, which matter (have data), which are leftovers
- Custom fields — which standard areas have been extended
- Custom apps — any scoped applications installed
- Overall assessment: "This instance is [lightly/moderately/heavily] customized. Most of the custom work is in [area]. This is [good/concerning] for long-term maintenance."

## 5. What Needs Attention
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

## 6. How Data Flows In and Out
Describe each integration in plain English:
- What system it connects to (inferred from endpoint/name)
- What it does (sends data, receives data, syncs)
- How secure the connection is (one line)

For data imports: what data comes in, where it goes.

Keep short. If there are no integrations, say "This instance is standalone — no active integrations with external systems."

## 7. Security & Access
Quick health check in plain English:
- Who has admin access and whether that's reasonable
- Whether user accounts are well-maintained
- Whether custom data is properly secured
- Whether key security settings are configured
- One-line bottom line: "This instance has the security posture of a [dev/production] environment."

## 8. Day One Cheat Sheet
The "screenshot this" section. Punchy bullets in four groups:
- **The basics** — 5-6 key facts
- **Things that will surprise you** — 3-4 things that aren't obvious
- **Don't touch until you understand** — 2-3 areas that need context before changes
- **Who to ask** — inferred from contributor data

## 9. Reference (small, at the bottom)
A compact table with the actual numbers — for the developers the manager will pass this to:

| Module | Records | Customizations | Custom Fields | Integrations |
|--------|---------|---------------|---------------|-------------|

This is the ONLY place configuration counts appear. Keep it small. Label columns in plain English ("Customizations" not "Business Rules + Client Scripts + UI Policies").

## What this skill does NOT do

- Does not show scripts or code
- Does not list individual configuration items
- Does not use ServiceNow jargon in the narrative
- Does not provide technical remediation steps
- Does not document process flows
- Configuration counts only appear in the reference table at the bottom

## The Pitch

"New to this instance? Run `/sn-onboard`. Fifteen minutes later you'll know what's here, who built it, what's working, what's stuck, and what needs attention. No ServiceNow expertise required to read it."
