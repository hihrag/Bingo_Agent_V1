---
name: monthly-reminders
description: Draft monthly schedule reminder texts for the Tuesday and Sunday bingo crews. Use whenever the user asks to draft monthly reminders, send out the month's schedule, remind crews of their dates for the month, or anything similar — typically at the start of a month. Outputs two ready-to-paste text messages, one per crew chat.
---

Draft monthly schedule reminder texts for the Tuesday and Sunday crews, ready for Hrag to copy and paste into the respective group chats.

## Inputs
- **Month and year.** Default: the current month. The user may say "May," "next month," "April 2026," etc. — interpret reasonably. If genuinely ambiguous, ask once for clarification.

## Workflow
1. Read `business_context.md` for crew rosters and event days. Crew membership is not included in the messages — the rosters are just confirmation that the Tuesday and Sunday crew chats are the right destinations.
2. Compute every Tuesday and every Sunday in the target month.
3. Draft two messages using the exact template below, one for each crew chat.
4. Output both messages clearly labeled, with no preamble or commentary.

## Message template (use verbatim)
```
Hey guys happy month of [Month], you're scheduled for the following [Tuesdays/Sundays] this month
• [Day, Month Date]
• [Day, Month Date]
• [Day, Month Date]
• [Day, Month Date]
Let me know if you need to call off any dates now
```

## Format rules (strict)
- Greeting is always "Hey guys happy month of [Month Name]" — same for both crews. Don't personalize to crew name. Don't substitute synonyms ("Hey team," "Hi everyone," etc.).
- Use "Tuesdays" or "Sundays" matching the crew being addressed.
- Dates: full day name, full month name, date — e.g., `Tuesday, May 5` or `Sunday, May 31`. One per line, bulleted with `•`.
- Call-off line is verbatim: `Let me know if you need to call off any dates now`. No paraphrasing.
- No sign-off, no closer, no "thanks team," no name. Message ends after the call-off line.
- No call times in the message (crew already knows).
- No emojis.
- No phone numbers.

## Output structure
```
For Tuesday crew chat:
[Tuesday message]

For Sunday crew chat:
[Sunday message]
```

No preamble before the drafts. No commentary after. The user copies each block directly into the respective group chat.

## Edge cases
- **5 Tuesdays or 5 Sundays in the month:** list all of them.
- **Past months:** draft anyway — the user may be testing or backfilling.
- **A month with crew schedule changes (TBD):** if `business_context.md` ever adds an exceptions section, honor it. Otherwise, assume default crews work all dates.

## Lessons learned
*(Empty until first real use. Append observations after each month's draft — wording adjustments, format tweaks, anything Hrag corrected.)*
