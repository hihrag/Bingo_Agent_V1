---
name: flyer-generation
description: Generate a branded Placentia Platinum Bingo flyer image. Use whenever the user asks to make a flyer, draft a flyer, generate event promo art, or design promotional material for a bingo event — whether a regular weekly Tuesday/Sunday or a special event (anniversary, holiday, themed night). Outputs a finished image via the Kie.ai image generation API, using brand assets from business_context.md.
---

# Flyer Generation

## Purpose
Turn a small set of event facts into a finished, branded Placentia Platinum Bingo flyer image. Claude writes the image-generation prompt directly from a structured `FlyerSpec`, validates that every required exact string appears in the prompt, then sends prompt + brand reference images to Kie.ai's Nano Banana Pro model. The result is saved to `flyers/YYYY-MM-DD_occasion.png`.

This skill is the authoritative workflow for flyer creation. Don't paraphrase — follow it step by step.

## Inputs

### Pulled automatically from `business_context.md` (don't ask)
- **hall_name:** "Placentia Platinum Bingo"
- **address:** "901 N. Bradford Ave., Placentia, CA"
- **logo_url:** from the *Brand assets* section
- **avatar_url:** illustrated host portrait, from *Brand assets*
- **sample_flyer_urls:** from *Brand assets*
- **voice / branding rules:** high-energy, payout-forward, never "gaming" or "gambling"; dark purple/navy backgrounds, gold for headline numbers, neon pink/magenta accents, balloon/confetti energy; "PLATINUM" metallic letters with "BINGO" letters on colored bingo balls

### Required from the user every call
Ask clearly if any are missing:
- **Event date** (e.g. "Tuesday, May 5, 2026")
- **Occasion / theme** — "regular Tuesday weekly", "Cinco de Mayo special", "5-year anniversary", etc.
- **Prizes / payouts to feature** — exact dollar amounts, verbatim
- **Special giveaways** (raffle baskets, door prizes, etc.) — verbatim, or "none"

### Phone number
Read `BUSINESS_PHONE` from `.env`. If not set, ask the user once and continue, but don't save it anywhere.

### Auto-filled from event date (single session, not morning/evening)
- **Tuesday** → "Doors open 4pm, Games start 6pm"
- **Sunday** → "Doors open 1pm, Games start 3pm"
- **Special event on a different day, or special hours** → ask the user for the times and override.

### Avatar (illustrated host portrait)
Default **ON**. Suppress only if the user says "without me on it," "no avatar," "no portrait," "skip the avatar," or equivalent.

### Occasion handling
- **Regular weekly (Tuesday/Sunday):** skip seasonal/holiday-specific imagery, but keep the high-energy festive feel: confetti, balloons, payout-forward styling, full brand color intensity. Weekly flyers should still feel exciting, just not themed.
- **Holiday / themed special:** layer in seasonal accents (see seasonal modifier list below). Avatar may pick up a tasteful accessory but identity must not change.
- **Anniversary or milestone:** emphasize the milestone in headline copy; keep the standard brand palette.

## Core Workflow

1. Read `business_context.md` and pull the brand assets (logo, avatar, sample flyer URLs) plus voice rules.
2. Confirm or collect required user inputs (date, occasion, prizes, giveaways, phone if not in `.env`, avatar override if applicable).
3. Build a `FlyerSpec` object (canonical shape below).
4. **Claude writes one detailed image-generation prompt directly** from the `FlyerSpec`, following the prompt-writer rules in this file. (No separate LLM call — Claude is the prompt writer.)
5. Run prompt validation. If any required exact strings are missing, rewrite the prompt with the missing strings inserted verbatim. Retry at most **2** times. If still invalid after 3 total attempts, stop and ask the user to confirm/correct rather than send a bad prompt.
6. Build the reference image array in priority order: avatar (if enabled) → logo → sample flyers.
7. POST to `https://api.kie.ai/api/v1/jobs/createTask` with the validated prompt and reference images.
8. Poll `https://api.kie.ai/api/v1/jobs/recordInfo?taskId=...` until `data.state == "success"` (or `failed`). Reasonable cadence: every 5 seconds, give up after ~3 minutes.
9. Download the result image from the first URL in `data.resultJson.resultUrls`.
10. Ensure `flyers/` exists at the project root; create it if not. Save as `flyers/YYYY-MM-DD_occasion.png` (slugify the occasion: lowercase, hyphens, no spaces).
11. Run the **Quality Gate** checks below and report status to the user.

## FlyerSpec (canonical shape)

```json
{
  "hall_name": "Placentia Platinum Bingo",
  "date": "",
  "phone": "",
  "address": "901 N. Bradford Ave., Placentia, CA",
  "session": "Doors open 4pm, Games start 6pm",
  "prizes": "",
  "special_giveaway": "",
  "occasion": "",
  "notes": "",
  "include_avatar": true,
  "logo_url": "",
  "avatar_url": "",
  "sample_flyer_urls": []
}
```

Rules:
- Preserve every text value **exactly** as supplied. Don't reformat phone numbers, addresses, dates, prices, or session times.
- Omit any blank field from the prompt.
- Keep the `FlyerSpec` separate from the generated prompt so corrections can be audited.

## How Claude writes the image prompt

Claude is the prompt writer. Internalize these rules and produce **one** detailed image-generation prompt as a single block of text — no JSON, no markdown, no commentary.

### Style rules
- Bright, saturated background using the Placentia Platinum palette: dark purple/navy base, gold for headline numbers, neon pink/magenta accents.
- Bold, senior-friendly typography with thick shadows or outlines.
- Clean visual hierarchy.
- Fun bingo aesthetic with themed icons (bingo balls, confetti, balloons).
- Oversized, high-contrast text readable on phones.
- Use the logo exactly as provided.
- Use the host avatar exactly as provided when included — same face geometry, proportions, skin tone, eyebrows, jawline, hairstyle, and personality.
- **If `include_avatar` is false, do not reference a host, person, or "me" in the prompt. Lead with the logo and event details only. Do not let the image model invent a generic person.**

### Layout rules
- Vertical flyer, aspect ratio 3:4.
- 2 to 4 boxed information sections maximum. Padding, spacing, breathing room.
- Avoid busy backgrounds behind text. Soft gradients or subtle themed graphics only.
- Balance sections vertically.
- Optimize for mass-text/SMS preview and easy phone readability for an older audience.

### Identity lock (verbatim — keep this language)
> If an avatar or host reference is provided, preserve the same face geometry, proportions, skin tone, eyebrows, jawline, hairstyle, and character personality. Do not simplify, redraw, or change the identity. Seasonal accessories may be overlaid, but the identity must remain the same.

### Text accuracy rule (verbatim)
> The prompt MUST include the exact date, phone number, address, session details, prices, and giveaway text from the input. Copy them verbatim. Do not invent offers, change numbers, change times, change prices, or add unapproved claims.

### Brand direction (verbatim)
> Match the color intensity, typography weight, and layout rhythm of the reference flyers. Keep the logo accurate and readable. Keep the flyer bold, cheerful, and high contrast. Use 2 to 4 clear content blocks, not a crowded collage. Make the date, session, prizes, giveaway, address, and phone immediately readable.

### Voice
- Punchy, payout-forward — let the prize numbers do the talking, no flowery adjectives.
- Never use the words "gaming" or "gambling." It's just **bingo**.
- High-energy and festive across both regular and special flyers.

## Prompt Validation

Before sending the prompt to Kie.ai, verify every non-empty required field appears verbatim in the generated prompt.

**Claude performs this validation directly in conversation. After writing the prompt, explicitly list each required field and confirm its verbatim presence in the prompt text. If any are missing, rewrite the prompt before initiating the API call.**

Required fields (only those with non-empty values in the `FlyerSpec`):
- `date`
- `phone`
- `address`
- `session`
- `prizes`
- `special_giveaway`

Validation logic (reference):

```python
def validate_prompt(generated_prompt: str, flyer_spec: dict) -> tuple[bool, list[str]]:
    fields = ["date", "phone", "address", "session", "prizes", "special_giveaway"]
    missing = []
    for field in fields:
        value = str(flyer_spec.get(field) or "").strip()
        if value and value not in generated_prompt:
            missing.append(field)
    return (len(missing) == 0, missing)
```

If validation fails, rewrite the prompt with this correction note in mind:

> Your prompt is missing required exact strings. You MUST include these strings verbatim:
> {{missing_field_list_with_values}}
> Regenerate one complete image prompt with those exact strings included.

**Retry at most 2 times** (3 total attempts). If still invalid, stop and ask the user to confirm or correct rather than generate a bad flyer.

## Image Generation Request

**If the Kie.ai API call fails with a 4xx error, fetch current Kie.ai documentation before retrying. Do not silently retry a malformed request — confirm request shape against current docs.**

Default settings:

```json
{
  "model": "nano-banana-pro",
  "aspect_ratio": "3:4",
  "resolution": "2K",
  "output_format": "png"
}
```

### Reference image priority order (keep this exact order)
1. Avatar / host image (if `include_avatar` is true)
2. Logo image
3. Sample flyer images

### Kie.ai create-task call

```bash
curl -X POST "https://api.kie.ai/api/v1/jobs/createTask" \
  -H "Authorization: Bearer $KIE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "nano-banana-pro",
    "input": {
      "prompt": "VALIDATED_PROMPT_TEXT",
      "aspect_ratio": "3:4",
      "resolution": "2K",
      "output_format": "png",
      "image_input": ["AVATAR_URL", "LOGO_URL", "SAMPLE_URL_1", "SAMPLE_URL_2"]
    }
  }'
```

Read `KIE_API_KEY` from `.env`. Do not hard-code it. Strip whitespace if present.

### Polling

```bash
curl "https://api.kie.ai/api/v1/jobs/recordInfo?taskId=TASK_ID" \
  -H "Authorization: Bearer $KIE_API_KEY"
```

Parse `data.state`:
- `success` → use the first URL in `data.resultJson.resultUrls`
- `failed` → surface the error to the user
- anything else → keep polling

Cadence: ~5 second interval, give up after ~3 minutes.

### Saving the output
- Ensure `flyers/` exists at the project root (`mkdir -p flyers/`).
- Filename: `flyers/YYYY-MM-DD_<occasion-slug>.png` — slug is lowercase, hyphens for spaces, alphanumerics only.
  - Examples: `flyers/2026-05-05_cinco-de-mayo.png`, `flyers/2026-05-12_regular-tuesday.png`
- Download via `curl -L -o <path> <url>`.
- If a file at that path already exists, append `_v2`, `_v3`, etc.

## Brand Rules

When sample flyers exist (they do — see `business_context.md`), inject these constraints into the prompt:

> Brand direction:
> - Match the color intensity, typography weight, and layout rhythm of the reference flyers.
> - Keep the logo accurate and readable.
> - Keep the flyer bold, cheerful, and high contrast.
> - Use 2 to 4 clear content blocks, not a crowded collage.
> - Make the date, session, prizes, giveaway, address, and phone immediately readable.

### Seasonal avatar modifiers (verbatim list)
- **Christmas:** santa hat and red scarf
- **Easter:** bunny ears or pastel egg accent
- **Halloween:** light spooky costume accessory
- **St. Patrick's Day:** leprechaun hat or shamrock accent
- **Valentine's Day:** hearts or rose accent
- **Summer:** sunglasses or beach hat
- **Thanksgiving:** pilgrim hat or harvest accent
- **Cinco de Mayo:** tasteful fiesta color accents, papel picado, marigold/orange/green details

Do **not** alter the avatar face or identity. Accessories overlay only.

For regular Tuesday/Sunday weekly flyers, skip seasonal accents entirely — use the standard Placentia Platinum brand palette.

## Quality Gate

Before reporting done:

- Confirm the saved file exists at `flyers/YYYY-MM-DD_<occasion>.png` and is non-empty.
- Confirm the final prompt contained every required exact string.
- If you can visually inspect, do — flag likely text rendering errors. Image models often distort phone numbers, addresses, and prices.
- If text is wrong, regenerate with a stricter correction prompt. **Do not edit the source `FlyerSpec` to make a bad output match — fix the generation, not the truth.**
- Save the prompt and the Kie.ai response alongside the image if the user asks for it.

Return to the user:
- Local path to the final image (markdown link).
- One-line status.
- Any known text risks (e.g. "the model occasionally garbles phone numbers — please double-check 714-XXX-XXXX is correct in the rendered image").
- The exact prompt used, if requested.

## Common Failure Modes

- **Missing exact date/phone/address/prices in the generated prompt:** retry prompt-writing before image generation.
- **Overcrowded flyer:** reduce to 2 to 4 content boxes.
- **Tiny unreadable text:** emphasize senior-friendly oversized typography.
- **Avatar identity drift:** strengthen identity lock language and ensure avatar is **first** in the reference image array.
- **Logo drift:** strengthen logo preservation language and keep logo **second** in the reference image array.
- **Wrong event details rendered in the image:** regenerate. Do not manually edit source data unless the user confirms the source was wrong.
- **Kie.ai job stuck or failed:** surface the raw error and the `taskId` so the user can retry or check the dashboard.

## Minimal end-to-end pseudocode

```python
flyer_spec = build_flyer_spec_from_business_context_and_user_inputs()

prompt = claude_writes_prompt(flyer_spec)
for attempt in range(2):  # 2 retries after the initial write
    valid, missing = validate_prompt(prompt, flyer_spec)
    if valid:
        break
    prompt = claude_rewrites_prompt(flyer_spec, missing)
else:
    raise ValueError("Prompt validation failed after 3 attempts — ask user to confirm inputs")

image_inputs = [
    flyer_spec["avatar_url"] if flyer_spec["include_avatar"] else None,
    flyer_spec["logo_url"],
    *flyer_spec.get("sample_flyer_urls", []),
]
image_inputs = [x for x in image_inputs if x]

task_id = kie_create_task(prompt, image_inputs)
result_url = kie_poll_until_done(task_id)

ensure_dir("flyers/")
out_path = f"flyers/{date_iso}_{slug(occasion)}.png"
download(result_url, out_path)

run_quality_gate(out_path, prompt, flyer_spec)
return out_path
```

## Lessons learned
*(Empty until first real use. Append observations after each generation — what worked, what the model garbled, what wording the user corrected.)*
