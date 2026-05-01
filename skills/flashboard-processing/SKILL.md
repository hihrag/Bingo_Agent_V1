---
name: flashboard-processing
description: Process a handwritten Flashboard sheet from a bingo event. Use whenever the user attaches or references a photo of a Flashboard form (numbered list of game names with payout/profit columns), or asks to log an event, calculate inventory cost, or reconcile event totals. Reads the sheet, looks up game costs in Games_Master, creates a per-event detail tab, appends to Event_Log, and reports totals plus reconciliation.
---

Process a handwritten Flashboard sheet from a bingo event: extract the games, match them to the master list, log the event to the workbook, and reconcile totals.

## Inputs
- Photos to process appear in `./inbox/` as `.jpg`, `.png`, or `.heic`.
- Archive processed photos to `./processed/` with filename prefix `YYYY-MM-DD_` (the event date).

## Workbook structure
- Tab `Games_Master`: columns are `game_name`, `aliases`, `cost_per_unit`, `expected_payout`, `notes`. Aliases are pipe-separated, case-insensitive.
- Tab `Event_Log`: columns are `event_date`, `processed_at`, `total_inventory_cost`, `total_profit_from_sheet`, `games_count`, `unmatched_count`, `reconciliation_status`, `detail_tab_link`, `notes`.
- Per-event tabs: named `Event_YYYY-MM-DD`, columns are `row_number`, `game_name_raw`, `game_name_matched`, `cost_per_unit`, `payout`, `profit`, `match_confidence`, `notes`.

## Workflow when given a Flashboard photo
1. Read the photo: extract the date from the header (top-right), every game/payout/profit from rows 1-31, and the Total Profit box at the bottom.
2. Read `Games_Master` via gws to get the canonical game list and aliases.
3. Match each game name (case-insensitive, whitespace-normalized, single-character edit distance allowed):
   - High confidence (exact or 1-char edit) → auto-accept. Confidence = `high`.
   - Ambiguous → list top 2 guesses and ask the user to confirm before continuing. Confidence = `confirmed` once user picks.
   - No reasonable match → flag as unmatched, do not guess. Confidence = `unmatched`.
4. Strip serial numbers and parenthetical annotations before matching, but preserve the raw string in `game_name_raw`.
5. For multi-tier payouts ("500/100" or "50/150/200"), sum them in `payout`, note the breakdown in `notes`.
6. Same game on two rows = two units sold. Each row gets its own line, costs sum naturally.
7. Compute total_inventory_cost = sum of cost_per_unit across matched rows. Compare profit column sum vs. Total Profit box.
   - Reconciliation status: `OK` (zero unmatched, totals match), `MANUAL_REVIEW` (any unmatched), `MISMATCH` (totals don't reconcile).
8. Create a new tab `Event_YYYY-MM-DD` via gws and populate it with one row per game.
9. Append a summary row to `Event_Log` via gws.
10. Move the source photo from `./inbox/` to `./processed/`, renaming with prefix `YYYY-MM-DD_` (event date).
11. Reply with: per-game breakdown (raw → matched + cost), total inventory cost, total profit from sheet, reconciliation status, unmatched games to add to master, and the detail tab name.

## Edge cases
- Crossed-out values: use the un-crossed value.
- Annotations next to names ("Donkey Dash (1000)354"): match on name portion only, preserve full string in `game_name_raw`.
- Empty rows: skip silently.
- Date format on the sheet is M-D-YY (e.g., "4-21-26" = April 21, 2026). Confirm with user only if ambiguous.
- Serial numbers next to game names: ignore for matching.
- Tab name collision: if `Event_YYYY-MM-DD` already exists, ask the user before overwriting.
- Illegible rows: include with confidence = `unmatched`, notes = "illegible". Do not guess.

## gws command reference
Read a range:
`gws sheets spreadsheets.values.get --params '{"spreadsheetId":"<id>","range":"Games_Master!A:E"}'`

Append rows:
`gws sheets spreadsheets.values.append --params '{"spreadsheetId":"<id>","range":"Event_Log!A:A","valueInputOption":"USER_ENTERED"}' --json '{"values":[[...]]}'`

Create a tab:
`gws sheets spreadsheets.batchUpdate --params '{"spreadsheetId":"<id>"}' --json '{"requests":[{"addSheet":{"properties":{"title":"Event_2026-04-21"}}}]}'`

Sheets ranges containing `!` need to be quoted carefully in shell — gws docs cover this if any command fails.

If any gws syntax fails, run `gws sheets --help` or `gws schema sheets.<resource>.<method>` to see the current command shape — gws is pre-1.0 and command shapes can change between versions.

## Lessons learned
*Populate this section after each event with anything that surprised you. Common patterns:*
- *Aliases that needed to be added to master.*
- *Handwriting quirks (specific letters that get confused).*
- *Edge cases in payout/profit columns.*
- *Reconciliation mismatches and their causes.*

### Event 2026-04-21
- **Master typo corrected:** `mee the beavers` → `meet the beavers`. Be alert for similar typos in master entries; flag them rather than silently relying on edit-distance matching.
- **Aliases added** (now auto-match without prompting):
  - `Safari 3 ball` ← `Safari 3 roll`
  - `sure bet` ← `sure boot`
  - `bingo ball bonus` ← `bingo pull`
- **Annotation patterns seen in `game_name_raw`:**
  - Trailing serial: `Down the Stretch 7577`, `River Rat 3043` — strip for matching.
  - Parenthetical denomination + trailing digits: `Donkey Dash (1000)354` — strip both.
  - Trailing single digit: `Pick a Betty 5` — appears to be an annotation (not a serial); strip.
- **Crossed-out values in payout column** are common (row 6 had `520/250` with `250` crossed). Per spec, use the un-crossed value; preserve the crossout in `notes`.
- **Multi-tier payouts** appeared in ~6 rows (`500/100`, `50/150/200`, `800/550`, `55/1000`, `520/250`). Always sum into `payout` and record the breakdown in `notes`.
- **Denomination-suffixed master games** (`leftover 800` vs `Leftovers 1500`, `spare change 800` vs `spare change 1500`) — disambiguate by matching the photo's payout column to the suffix.
- **Reconciliation mismatch root cause:** when profit-column digits are illegible (rows 16 and 18 of this event), the computed sum won't tie to the Total Profit box. Record best-effort values, flag in `notes`, and ask the user to verify rather than guessing. Difference here was $870 ($7,985 computed vs $8,855 box).
- **`gws` quirk:** `gws` auto-loads `.env` from cwd. Don't put `GOOGLE_WORKSPACE_CLI_CREDENTIALS_FILE` in `.env` — gws picks up its own encrypted creds automatically when that var is unset.

### Event 2026-04-26
- **New games added to master** (after first encounter as unmatched):
  - `derby talk` ($48.49) — handwriting can also look like "Donkey Talk"; consider adding alias.
  - `Top Dog` ($81.89) — first time seen on a sheet.
  - When a row is unmatched and the photo's name is plausible as a real game, ask the user to add to master rather than guessing. Then patch the event tab + summary in place.
- **Handwriting pattern: 'Duck' shorthand for 'Ducky'** — `Rubber Duck 2988` matched `rubber ducky` via 1-char edit. Common abbreviation worth remembering.
- **Handwriting pattern: 'Avier' / 'Aviar' for 'Pixie'** — confirmed match. Consider adding `avier` as an alias if it shows up again.
- **Denomination-based matching** worked cleanly: `Left Overs` with payout 1500 → `Leftovers 1500` (vs `leftover 800`). Always check the payout column when a game name has multiple denomination variants in master.
- **Small reconciliation gap ($64.04)** — when the diff is small and ends in cents (`.04`), it's almost certainly a single misread digit on one row, not a structural issue. Worth flagging but not blocking.
- **Bottom-of-sheet notes** appeared on this sheet ("5 payouts for strips 1000 each", "480 IN from Register"). Capture them in `Event_Log.notes` rather than dropping them.
- **Annotations to strip:** `(354)` after `Donkey Dash`, `50-1` after `Dab it 1-2-3`, `2407` and `3098` and `2988` serial numbers. Pattern is consistent with prior event.

### Event 2026-02-15
- **HEIC inputs** are supported per spec but exceed the 256KB Read limit. Pre-convert with `sips -s format jpeg -Z 2000 <in.HEIC> --out <out.jpg>` before reading. Photos are also commonly sideways — rotate with `sips -r -90` (and `-r 180` if still upside-down) before OCR.
- **Color-suffixed Zippers** (`Zippers Orange`, `Zippers Green`, `Zippers Blue`, all payout 500) all map to the single master entry `zipperz` per user — colors are packaging variants, same cost. Treat any `Zippers <color>` reading the same way going forward, no need to re-prompt.
- **Handwriting drift confirmed by user:**
  - `Pancake split` → `banana split` (P/B and -an- shape confusion).
  - `Monopoly` shorthand → `monopoly tic tac` (single-word "Monopoly" maps to the master entry).
  - `Pick a Loser` → `pick a looney` (looney/loser visual drift).
  - `Pixies` → `pixie` (1-char edit, auto-accepted).
- **Denomination-suffixed master games** continued working: `Dab it 123` and `Dab it 1-23` both with payout 1500 → `dab it 123 1500` (vs `dab it 123` $58.85). `Left over` payout 1500 → `Leftovers 1500`.
- **Crossouts on multi-tier values** common again: row 8 (Zingo `1000/[crossed]` and `[crossed]/615`) and row 24 (Classy or sassy `1000/[crossed]` and `[crossed]/715`) and row 16 (Bum out `1199/[crossed]`). Spec rule held: use un-crossed value, note breakdown.
- **`Bum out`** → `burnout` (one word) via space-removal. Consider adding `bum out` as alias to avoid future ambiguity.
- **Master updates after user feedback:**
  - `lightning betty boop` added as alias of existing `betty boop` ($101.66). The handwritten name read at first as "Lightning Bobby Poop"; correct reading is "Lightning Betty Boop". Worth remembering: "Betty Boop" can look like "Botty/Bobby Poop" in cursive.
  - `Lucky Kat` added as new master entry at $104.50.
- **User said to ignore (left as `unmatched` in event tab, no master action):**
  - `Double action` (r19, payout 500) and `X factor` (r20, payout 2000). If they reappear, treat as unmatched again unless user reverses.
- **Small reconciliation gap ($20)** — column sum 14777 vs Total Profit box 14797. Consistent with prior pattern: small whole-dollar diffs almost always a single misread digit.

### Event 2026-04-14
- **User-confirmed mappings worth remembering:**
  - `Bingo Ball Pink` and `Bingo Ball Gold` (both payout 150) → both map to `bingo ball bonus` ($23.71). Same color-variant pattern as Zippers Orange/Green/Blue.
  - `Blazing Balls` (payout 500) → `fire balls` ($55.19). Likely synonym worth treating consistently.
  - `2-Ball` shorthand → `2 ball bingo` ($55.19). Auto-accept on future sheets.
- **User said leave unmatched (no master action):**
  - `Double Dragon` (payout 500). Don't auto-match to `Lucky Dragon`.
- **Date misread risk:** the middle digit of "4-1?-26" was unclear between 8 and 4 — user clarified 4-14-26. When the date digit is ambiguous, ask before creating the tab — back-tracking a tab name is annoying.

### Event 2026-04-12 (multi-page)
- **Multi-page handling:** main sheet (rows 1-31 + Total Profit box) plus a continuation sheet (separate # column starting at 1). User wanted both combined into one Event_YYYY-MM-DD tab. Used `M3..M33` for main rows (M32/M33 = overflow rows written below the form's row 31), and `C1..C11` for continuation rows. This keeps form-position info intact without numeric collisions.
- **Continuation pages typically have no Total Profit box.** Treat the main sheet's box as authoritative for `total_profit_from_sheet`. Computed sums for both pages should still be in notes.
- **Skipped rows** (per user): rows containing things that aren't actual games — annotations, register notes, or scratch work. Mark `match_confidence = "skipped"` in the event tab and exclude from `games_count` / costs / profit sums, but keep the row visible so the audit trail is intact.
- **Denomination defaults for high-tier payouts:** `Dab it 1-2-3` with payout 3000+ (e.g., 1000×3) and `3's a charm` with payout 6000 (= 2×3000) and `Left over` with payout 3000 don't cleanly map to any single existing tier. Default to the highest-tier master entry (e.g., `dab it 123 1500`, `3s a charm 3000`, `Leftovers 1500`) and flag in notes for user verification — these are educated guesses, not high-confidence matches.
- **`Wild Bear Race` → `wild boar race`** via 1-char edit (Bear/Boar). Worth remembering as a recurring handwriting variant.
- **Annotation-only rows:** row 2 of the main sheet had payout-cell annotations like "↑(200) +(400) (31)(2u)" with no game name. Treat row 2 as empty (skip silently per spec).
- **Circled profit values** appear on this sheet (row M3 X Game profit circled; row C7 3's a charm profit circled). Meaning is unclear — possibly indicates a special category or jackpot win. Record the value as-is and note the circle in `notes`; don't second-guess.
- **HEIC photos with multiple sheets photographed together:** the original photo had main + continuation laid side-by-side, rotated 90° in the frame. Need to rotate (`sips -r 90`) once, not -90+180; cropping was easier off the rotated version. Pre-converting and rotating once is the right move before any cropping or OCR.
