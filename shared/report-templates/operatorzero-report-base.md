# OperatorZero Report Style — Standard

Shared visual spec for HTML reports produced by skills in this library. Skills should reference this file rather than duplicating styling instructions. For the lighter narrative variant used by onboarding-style skills, see `operatorzero-onboarding-style.md`.

## Typography
- Font: Inter, system fonts
- Body text: 13px, line-height 1.6
- Muted: `#6b7280`
- Emphasis / headers: `#111827`
- Header weight: 600, size 18px
- Stat values: 28px weight 700

## Layout
- Background: `#f5f6fa`
- Cards: `#ffffff`, `border-radius: 8px`, `box-shadow: 0 2px 12px rgba(0,0,0,0.08)`
- Max content width: 850px
- Sticky top nav with section links
- Numbered sections with blue badge squares
- Tables: alternating row shading, compact padding
- No code blocks anywhere in the rendered report

## Color Tokens
- Accent blue: `#2563eb`
- Cover gradient: `linear-gradient(135deg, #1e3a8a, #1d4ed8)` with report title and instance name

## Severity Palette
| Tier | Color | Use |
|---|---|---|
| Critical | `#dc2626` | systemic failures, broken at 100% |
| High | `#ea580c` | high error rate, urgent migration |
| Medium | `#ca8a04` | dormant, orphaned, low-priority risk |
| Low | `#16a34a` | healthy, well-configured |

Render severity as pill badges (red/orange/yellow/green).

## Skill-specific extensions

Skills add their own badges or visual elements in their own `topics/` files and reference this base. Examples:

- **sn-acl-scan** — research citation callouts with source links
- **sn-dead-code** — tier badges (Tier 1 red performance, Tier 2 orange dead, Tier 3 yellow orphaned, Blue suspicious)
