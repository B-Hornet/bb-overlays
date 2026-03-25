# Beats & Bourbon — Flyer Production System
# Created: 2026-03-25
# Updated: 2026-03-25 — Post-FlyerTemplate session fixes
# Source: Purple Reign session — 10 version iteration postmortem + FlyerTemplate debug session

---

## THE PRIME RULE
Calculate the full y-budget BEFORE placing a single element.
Never stack elements downward and hope they fit.
Map every block top-to-bottom first, verify total ≤ canvas height, then draw.

---

## CANVAS SPEC
- Size: 1080×1080px (square social), 1920×1080px (landscape stream)
- Safe margin: 72px left/right, 40px top/bottom minimum
- Always verify: last text baseline + font descender + 40px ≤ canvas height

---

## Y-BUDGET TEMPLATE (1080×1080 square)

Block                        | Y range      | Notes
-----------------------------|--------------|---------------------------
Reeboot Radio Presents       | y=55         | 20px font, top anchor
Accent bars start            | y=68         |
BEATS & baseline             | y=178        | 130px font
BOURBON baseline             | y=306        | 130px font
Tagline baseline             | y=346        | 22px font
EST. 2023 baseline           | y=370        | 16px font
Accent bars end              | y=280        |
Rule 1                       | y=400        |
Premium Sound Menu           | y=436        | 20px font
Lineup row 1                 | y=462        |
Lineup rows (7 × 58px step)  | y=462→810    | last baseline y=810
Rule 2                       | y=840        |
Twitch block                 | y=852→876    |
Rule 3                       | y=910        |
Date baseline                | y=978        | 76px font, top≈902 ✓
Footer baseline              | y=1040       | 13px font
Canvas bottom                | y=1080       | 40px clearance ✓

Verify before every build:
- Date top (978-76=902) > Rule 3 (910)? → YES ✓
- Footer (1040) + 13px + descender ≈ 1058 < 1080? → YES ✓

---

## TWO-COLOR TEXT RULE (non-negotiable)

Color              | Hex       | Used for
-------------------|-----------|------------------------------------------
Near-white         | #F2ECFF   | Titles (BEATS &, BOURBON), DJ names, Date
Warm gold          | #F2C050   | Times, subheads, tagline, labels, rules
Dim gold           | #C49040   | EST. 2023, footer, secondary metadata
Twitch dot only    | #BF94FF   | ● bullet dot ONLY — not the text
Accent bar         | #9050D8   | Left/right vertical bars flanking title

### ⚠️ CRITICAL — ERRORS MADE IN SESSION, NEVER REPEAT:

**ERROR 1 — Theme ramp used for text colors (caught 2026-03-25)**
`gold` and `dim` were pulled from `theme.r[3]` and `theme.r[1]`.
On Purple Reign, r[3]=#B878E8 and r[1]=#3A1468 — both vanish on dark purple.
THE FIX: Text colors are HARDCODED CONSTANTS. Never pulled from theme ramp.
The theme ramp is for the DAMASK MOTIF ONLY. Not text. Not ever.

```javascript
// ✅ CORRECT — constants, always
const white = '#F2ECFF';
const gold  = '#F2C050';  // NOT #D4A84B — too mudby on purple
const dim   = '#C49040';  // NOT #A07830

// ❌ WRONG — ramp values bleed into text
const gold = theme.r[3];  // becomes purple on Purple Reign
const dim  = theme.r[1];  // becomes near-black on Purple Reign
```

**ERROR 2 — Twitch text in #BF94FF (caught 2026-03-25)**
"● LIVE ON TWITCH" rendered entirely in Twitch purple.
Purple text on dark purple background = invisible.
THE FIX: Split into two fillText calls.

```javascript
// ✅ CORRECT
ctx.fillStyle = '#BF94FF'; ctx.fillText('●', W/2 - 100, 866);  // dot only
ctx.fillStyle = white;     ctx.fillText('LIVE ON TWITCH', W/2 + 18, 866);  // white text

// ❌ WRONG
ctx.fillStyle = '#BF94FF';
ctx.fillText('● LIVE ON TWITCH', W/2, 866);  // entire line purple = invisible
```

NEVER use purple text on purple background — it disappears.
NEVER use near-white for metadata — it competes with titles.
Gold on dark purple = maximum contrast. Always default to gold for small text.

---

## TYPOGRAPHY SYSTEM

Element                   | Font                          | Size  | Weight | Tracking
--------------------------|-------------------------------|-------|--------|----------
Reeboot Radio Presents    | Georgia                       | 20px  | 400    | 7px
BEATS & / BOURBON         | Georgia                       | 130px | bold   | 0px
Tagline                   | Georgia                       | 22px  | 500    | 5px
EST. 2023                 | Georgia                       | 16px  | 400    | 4px
Premium Sound Menu        | Georgia                       | 20px  | 500    | 8px
Times (lineup)            | Courier New monospace         | 27px  | bold   | 1px
DJ Names (lineup)         | Georgia                       | 27px  | bold   | 0.5px
Live on Twitch            | Georgia                       | 22px  | 600    | 6px
DATE                      | Arial Black / Arial sans      | 76px  | 900    | 8px
Footer                    | Georgia                       | 13px  | 400    | 4px

KEY RULE — Date font:
- Use Arial Black (900 weight) for the full date string as ONE fillText call
- Never split date into separate parts with different fonts — spacing collision
- Double space between words: 'SAT  APR  05' not 'SAT APR 05'
- Georgia is for brand words. Arial Black is for punchy date numbers.

---

## GAS TEMPLATE GUARD RULES
### ⚠️ CRITICAL — ALWAYS APPLY TO FlyerTemplate.html

**RULE 1 — JSON.parse must have try/catch**
`<?= scheduleJson ?>` is only substituted when served by GAS HtmlService.
In browser preview, it's a literal string — JSON.parse crashes.
ALWAYS wrap in try/catch with fallback sample data.

```javascript
// ✅ CORRECT
let SCHEDULE, THEME_NAME;
try {
  SCHEDULE   = JSON.parse('<?= scheduleJson ?>');
  THEME_NAME = '<?= themeName ?>';
} catch(e) {
  SCHEDULE   = { date: 'SAT  APR  05', lineup: [...] }; // fallback
  THEME_NAME = 'purple';
}

// ❌ WRONG — crashes in browser
const SCHEDULE = JSON.parse('<?= scheduleJson ?>');
```

**RULE 2 — google.script.run must be guarded**
`google.script.run` only exists inside GAS HtmlService sandbox.
In browser preview, `google` is undefined — calling it throws ReferenceError.
ALWAYS check existence before calling.

```javascript
// ✅ CORRECT
if (typeof google !== 'undefined' && google.script) {
  google.script.run.receiveFlyerPng(dataUrl);
} else {
  // browser preview fallback — show canvas + download button
}

// ❌ WRONG — ReferenceError in browser
google.script.run.receiveFlyerPng(dataUrl);
```

---

## LAYOUT RULES

1. CENTER everything — no left-alignment unless it's a landscape/split layout
2. RAISE the header — first text element at y=55, no dead air at top
3. ACCENT BARS — two vertical bars flanking title block (left x=72, right x=W-77)
   both 5px wide, same color as accent, spanning header height
4. SCHEDULE centered as a unit:
   - Time column right-aligns to W/2-20
   - Pipe separator at W/2
   - DJ name left-aligns from W/2+20
5. TWITCH block — centered unit: ● dot in #BF94FF, text in near-white
6. DATE — centered, Arial Black, ONE string, double-spaced words
7. FOOTER — centered, smallest text, 40px+ above canvas bottom

---

## THEME COLOR SYSTEM

Each theme replaces ONLY the damask motif color ramp.
Background gradient, vignette, text colors, and layout stay identical.

Theme         | Base bg          | Motif ramp (dark→light)
--------------|------------------|------------------------------------------
Classic       | #1C0B02→#0C0500  | #41220D → #9D6632 → #EFC884 → #FDF4E0
Midnight      | #02060E→#010308  | #0A1A3A → #1A4080 → #6090D0 → #C0D8F8
Crimson       | #0E0202→#050001  | #3A0808 → #901818 → #D04040 → #F8C0C0
Sunset        | #0E0802→#050300  | #3A1A04 → #A04010 → #E07030 → #F8C090
Purple Reign  | #06010E→#02000A  | #1A0632 → #5C28A0 → #B878E8 → #E0C0F8

Damask motif uses 5-stop radial gradient per element.
Text colors NEVER change with theme — they are always the constants above.

---

## BACKGROUND LAYER STACK (in order)

1. Radial gradient base — theme color, bright center-top, dark corners
2. Damask tile — globalAlpha 0.90, brick-offset pattern, 118×112px tile
3. Top bloom — radial glow, theme color, upper center, alpha ~0.30
4. Vignette — dark radial, crushes corners to near-black
5. Bottom fade — linear gradient, last 30% of canvas height
6. Film grain — getImageData pixel noise ±10, applied once after all layers
7. Text overlay — fixed color constants, never theme ramp values

---

## DAMASK TILE SYSTEM

Tile size: 118×112px
Tiling: brick offset — odd rows offset by TW/2
Alpha: 0.90

Performance pattern: pre-render motif to offscreen canvas once, then
drawImage() stamp across the grid. DO NOT call drawMotif() per-cell on
the main canvas — too slow at 1080px.

```javascript
// ✅ CORRECT — build once, stamp many
const tile = buildTile(); // offscreen canvas
for (let row...) for (let col...)
  ctx.drawImage(tile, x, y);

// ❌ SLOW — redraws motif paths per cell
for (let row...) for (let col...)
  drawMotif(ctx, x, y, ramp);
```

---

## PRE-FLIGHT CHECKLIST (run before every export)

STRUCTURE
□ Y-budget calculated top-to-bottom before any drawing
□ Last element baseline + descender + 40px ≤ 1080
□ No dead air at top — first element within 60px of canvas top
□ Date is ONE fillText call in Arial Black, not split pieces
□ Double spaces in date string: 'SAT  APR  05'

TYPOGRAPHY
□ Near-white (#F2ECFF) only on: title, DJ names, date, LIVE ON TWITCH text
□ Gold (#F2C050) only on: times, subheads, labels, small text, rules
□ Twitch purple (#BF94FF) only on: ● dot, NOT the full text line
□ No purple text anywhere else in the layout
□ Schedule times right-aligned, DJ names left-aligned, pipe at center

TEXT COLOR GUARD — check before every render function
□ `const white`, `const gold`, `const dim` are HARDCODED HEX — not `theme.r[x]`
□ If you see `theme.r[` anywhere in drawTextOverlay — STOP, it's wrong

GAS TEMPLATE GUARDS — check before deploying FlyerTemplate.html
□ JSON.parse wrapped in try/catch with fallback schedule data
□ google.script.run guarded with typeof check
□ Canvas visible in browser preview mode (not just GAS mode)

LAYOUT
□ All elements centered (textAlign='center' or symmetric left/right)
□ Accent bars present on both sides of title block
□ Twitch: ● dot (#BF94FF) + text (near-white) — two separate fillText calls
□ Three ornamental rules: after header, after schedule, before date

BACKGROUND
□ Damask full bleed — no solid panel blocking texture
□ Vignette darkens corners, center stays readable
□ Film grain applied as final layer before text
□ Theme color correct — motif ramp matches requested theme

READABILITY TEST
□ Schedule times legible at 50% zoom?
□ DJ names legible at 50% zoom?
□ Date punchy and readable at thumbnail size?
□ No text disappearing into background?
□ "LIVE ON TWITCH" visible? (was invisible when purple-on-purple)

---

## ORNAMENTAL RULE FUNCTION

drawRule(y) — always centered, always gold diamond + dots:
- Left line: x=80 → W/2-22
- Right line: W/2+22 → W-80
- Diamond: centered at W/2, rotated 45°, gold stroke
- Flanking dots: W/2±22, radius 2.5, gold fill

---

## SESSION DEBRIEF — FlyerTemplate.html Debug Session
Date: 2026-03-25
Errors fixed: 4

### Errors Made (log for prevention)

| Error | Root Cause | Fix |
|-------|-----------|-----|
| Gold/dim text invisible on purple | `gold=theme.r[3]`, `dim=theme.r[1]` — theme ramp bled into text | Hardcode `#F2C050` and `#C49040` always |
| "LIVE ON TWITCH" invisible | Full line in `#BF94FF` — purple on purple | Split: dot=#BF94FF, text=#F2ECFF |
| Black screen in browser | No try/catch on `JSON.parse('<?= ?>')` | Wrap in try/catch with fallback data |
| ReferenceError: google not defined | `google.script.run` called without guard | Wrap in `typeof google !== 'undefined'` check |

### Key Rule Reinforced
The two-color text rule is INVIOLABLE. Text colors are constants.
Theme ramp = motif only. If you're pulling `theme.r[x]` in drawTextOverlay,
you are making a mistake.
