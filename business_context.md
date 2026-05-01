# Bingo Business Context

Persistent facts about Placentia Platinum Bingo. All skills should read this file when they need context about the operation. This is a living document — update it as things change.

## Business basics
- **Business name:** Placentia Platinum Bingo
- **Operates under:** Youth Help Center (charity bingo)
- **Operator:** Hrag

## Locations
**Placentia Women's Club**
- Address: 901 N. Bradford Ave., Placentia, CA
- Sole venue for all events.

## Events
Recurring schedule, year-round.
- **Tuesdays:** doors 4pm, games 6pm, wraps ~10pm
- **Sundays:** doors 1pm, games 3pm, wraps ~7:30pm

## Crews
**Tuesday runner crew (rotates):** Joey, Bella, Lisa, Chris
**Sunday runner crew (rotates):** Erik, Karo, Thomas, Bella
**Standby runners (called in for call-offs or extra help):** Marty, Hanu, Tina

**Permanent staff (work both Tuesdays and Sundays, all events):**
- Aram — caller
- Joy — accountant
- Shannon — host
- Mike — manager
- Luis — day porter / maintenance
- Security guard (different person per day, not tracked by name)

**All staff are volunteers and work for tips. No payroll. No tip tracking.**

## Group chats
- **Tuesday runners chat** — for Tuesday crew
- **Sunday runners chat** — for Sunday crew
- **Management + permanent workers chat** — for permanent staff

**Monthly schedule reminders go to:** Tuesday runners chat and Sunday runners chat only. Permanent staff don't need monthly reminders since they work every event.

**Standbys (Marty, Hanu)** — text them individually when needed. Hrag handles the actual texting.

## Vendors
- **Bingo West** — preferred vendor. Delivers; cheaper pricing.
- **Marathon** — used for variety and better game selection.

Default to Bingo West unless the order needs something only Marathon carries.

## Recurring expenses
Paid by Hrag.

**Venue (paid at start of each month for that month's dates):**
- Tuesdays: $1,200/day
- First Tuesday of every month: $1,500
- Sundays: $1,200/day
- Storage unit (Placentia Women's Club): $200/month flat

**Other monthly:**
- Liability insurance: monthly

**Variable:**
- ATM vendor: charged based on transaction volume per event
- Supplies: coffee cups, trash bags, etc.

**Quarterly:**
- Security guard: billed every 3 months

**Other recurring services:**
- TXT180 — text messaging service for player flyer blasts. Open to alternatives if substantially cheaper, but not actively shopping. Don't proactively flag alternative services.

## Branding and voice
**Voice:** friendly, high-energy, payout-forward. Punchy — let prize amounts do the talking, not adjectives. Cool/festive tone consistent across regular and special flyers. Casual with crew and staff (they're volunteers and friends).

**Word choice:** "bingo" is just "bingo." Never "gaming," never "gambling."

**Visual identity:** dark purple/navy backgrounds, gold for headline numbers, neon pink/magenta accents, balloon/confetti festive energy. Logo: "PLATINUM" in metallic letters with "BINGO" letters on colored bingo balls. Illustrated portrait of Hrag with a microphone is a recurring brand element on flyers.

**Always include on player-facing materials:**
- Doors open / games start times
- Address (901 N. Bradford Ave., Placentia, CA)
- Phone: 714 number only (business line for reservations)

**Flyers** are designed in Canva. A separate `brand_assets.md` will be added later with logo files, color codes, the illustrated portrait, and design templates.

## Brand assets

These URLs are used by skills that generate visual content (e.g., flyer generation). Local masters live in `brand_assets/`. If a URL ever breaks, re-upload from the local masters and update this section.

- **Logo URL:** https://raw.githubusercontent.com/hihrag/Bingo_Agent_V1/main/brand_assets/logo.jpeg
- **Illustrated host avatar URL:** https://raw.githubusercontent.com/hihrag/Bingo_Agent_V1/main/brand_assets/avatar.png
- **Sample flier URLs:**
  - https://raw.githubusercontent.com/hihrag/Bingo_Agent_V1/main/brand_assets/sample1.jpeg
  - https://raw.githubusercontent.com/hihrag/Bingo_Agent_V1/main/brand_assets/sample2.jpeg
  - https://raw.githubusercontent.com/hihrag/Bingo_Agent_V1/main/brand_assets/sample3.jpg

## Notes for the agent
- **Default behavior:** act first, show results. Draft, generate, propose — don't ask permission for minute details. Show me a result and I'll redirect if needed.
- **Money:** no special routing required. Joy handles books, but the agent doesn't need to involve her in normal workflows.
- **Texts/communications:** the agent always drafts and Hrag sends from his own phone. Never include or store actual phone numbers in this file or any skill output. The agent doesn't need them.
- **This file is a living document.** Expect it to grow and shift. When something changes (new staff, new vendor, schedule shift), update it.
