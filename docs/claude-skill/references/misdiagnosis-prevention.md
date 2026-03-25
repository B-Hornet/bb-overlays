# Misdiagnosis Prevention Log — Reference Guide
# Created: 2026-03-05
# Updated: 2026-03-25 — BB Flyer session errors added
# Purpose: Track failed fix attempts so they are NEVER repeated

## Core Principle
**If you can't explain the problem in one sentence, you don't understand it enough to fix it.**

## Visual Bug Protocol
1. **REQUIRE** a screenshot before any fix attempt
2. **IDENTIFY** the exact property from the visual
3. **GREP/READ** the file to confirm the property exists and is set
4. **FIX** only that property — one change at a time
5. **VERIFY** with visual confirmation

---

## Logged Misdiagnoses

### "Double Spacing" in Chat (2026-03-05)
**Symptom:** Words in chat messages had large gaps between them
**Root cause:** `textAlign: TextAlign.justify` on Text widgets

| Attempt | What Was Tried | Time Wasted | Why Wrong |
|---------|---------------|-------------|-----------|
| 1 | Removed `height: 1.3` from Text.rich | 30 min | Line height controls vertical spacing, not horizontal word gaps |
| 2 | Reduced bubble padding 16→8px | 30 min | Padding is outside the text, doesn't affect word distribution |
| 3 | Added `height: 1.1` to 4 TextStyles | 30 min | Same wrong property, applied to more widgets |
| 4 | Global M3 theme override in main.dart | 45 min | Material 3 line height defaults don't cause horizontal spread |
| 5 | Scoped Theme() wrapper on root widget | 30 min | Same wrong approach, scoped instead of global |
| 6 | Searched for wordSpacing | 15 min | grep confirmed it was never set anywhere |

**Total time wasted:** ~3 hours
**Time to fix with screenshot:** 10 seconds

**Key insight:** `TextAlign.justify` distributes space between words to fill line width.
The "orphan" last line (left-aligned) is the telltale sign.

---

### BB Flyer — Text Colors Invisible on Purple Theme (2026-03-25)
**Symptom:** Gold times and footer text invisible on Purple Reign flyer
**Root cause:** `const gold = theme.r[3]` — theme ramp bled into text constants

| Attempt | What Was Tried | Why Wrong |
|---------|---------------|-----------|
| 1 | Wrote drawTextOverlay pulling gold/dim from theme.r[] | Spec violation — text colors are NEVER from theme ramp |

**Time wasted:** One iteration
**Root cause in one sentence:** Theme ramp values for Purple Reign are purple tones —
using them as text colors makes text invisible on a dark purple background.

**Fix:**
```javascript
// ALWAYS hardcoded — never theme.r[x]
const white = '#F2ECFF';
const gold  = '#F2C050';  // was #D4A84B — too muddy
const dim   = '#C49040';  // was #A07830
```

**Rule added to spec:** Theme ramp (`theme.r[]`) is for damask motif ONLY.
Text colors are spec constants. If you see `theme.r[` in drawTextOverlay — STOP.

---

### BB Flyer — "LIVE ON TWITCH" Invisible (2026-03-25)
**Symptom:** Twitch line completely invisible on Purple Reign flyer
**Root cause:** Entire `"● LIVE ON TWITCH"` string rendered in `#BF94FF` (light purple)
on a dark purple background — insufficient luminance contrast

**Fix:** Split into two fillText calls:
```javascript
// ● dot = Twitch brand purple (decorative signal only)
ctx.fillStyle = '#BF94FF';
ctx.fillText('●', W/2 - 100, 866);

// Text = near-white (always readable on any dark background)
ctx.fillStyle = '#F2ECFF';
ctx.fillText('LIVE ON TWITCH', W/2 + 18, 866);
```

**Rule:** Never render a full text line in a color that shares the same hue family
as the background. Purple text on purple bg = invisible, always.

---

### BB Flyer — GAS Template Crashes in Browser (2026-03-25)
**Symptom 1:** `Uncaught SyntaxError: Unexpected token '<', "<?= scheduleJson ?>" is not valid JSON`
**Symptom 2:** `Uncaught ReferenceError: google is not defined`

Both errors happen when FlyerTemplate.html is opened directly in a browser
instead of being served by GAS HtmlService.

**Root cause 1:** `JSON.parse('<?= scheduleJson ?>')` — the `<?= ?>` tag is only
substituted when GAS serves the template. In a browser it's a literal string.

**Root cause 2:** `google.script.run` only exists in the GAS sandbox.
In a browser, `google` is undefined.

**Fix — ALWAYS apply to any GAS HtmlService template:**
```javascript
// Rule 1: try/catch on template variables
let SCHEDULE, THEME_NAME;
try {
  SCHEDULE   = JSON.parse('<?= scheduleJson ?>');
  THEME_NAME = '<?= themeName ?>';
} catch(e) {
  SCHEDULE   = { ...fallback sample data... };
  THEME_NAME = 'purple';
}

// Rule 2: guard google.script.run
if (typeof google !== 'undefined' && google.script) {
  google.script.run.receiveFlyerPng(dataUrl);
} else {
  // browser preview — show canvas + download button
}
```

**Rule:** Every GAS HtmlTemplate file must be previewable in a browser.
If it crashes on direct open, it's not production ready.

---

## Red Flags — Stop and Re-analyze

1. Same ID repeated 3+ times in logs (container reuse)
2. User says "this worked before you changed it"
3. Your fix creates NEW problems
4. You're on attempt 3+ without progress
5. You can't explain root cause in one sentence
6. You changed 3+ files and can't isolate which helped
7. You deleted code instead of understanding why it existed
8. **You're fixing a visual bug without having seen a screenshot**
9. **NEW: You're pulling theme ramp values into text color variables (BB flyer)**
10. **NEW: You wrote a GAS template without try/catch + google guard (BB flyer)**

---

## Decision Tree for Visual Bugs

```
User reports visual issue
  → Do you have a screenshot?
    → NO: Ask for one. Do NOT attempt any fix.
    → YES:
      → Can you identify the exact property from the visual?
        → YES: read the file, confirm it's set, fix it — ONE change
        → NO: Ask for a more specific screenshot (zoom in, dev tools)
```

## Decision Tree for GAS Template Files

```
Writing a new HtmlService template?
  → Does it use <?= template variables ?>?
    → YES: Wrap ALL JSON.parse calls in try/catch with fallback data
  → Does it call google.script.run?
    → YES: Guard with: if (typeof google !== 'undefined' && google.script)
  → Test: open the file directly in a browser
    → Crashes? → Not production ready. Fix the guards first.
    → Renders? → Ready to deploy to GAS.
```
