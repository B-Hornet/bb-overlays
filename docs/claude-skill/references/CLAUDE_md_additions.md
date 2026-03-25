# CLAUDE.md — BB Flyer System Additions
# Append these sections to the existing CLAUDE.md in the repo
# Updated: 2026-03-25

---

## FLYER SYSTEM — FlyerTemplate.html

### Text Color Rule (INVIOLABLE)
Text colors in `drawTextOverlay()` are HARDCODED CONSTANTS. Never theme ramp values.

```javascript
// ✅ ALWAYS — these exact hex values, every theme
const white = '#F2ECFF'; // titles, DJ names, date, Twitch text
const gold  = '#F2C050'; // times, subheads, labels, rules
const dim   = '#C49040'; // EST. 2023, footer, secondary metadata
```

If you see `theme.r[` anywhere inside `drawTextOverlay()` — stop and fix it.
That's a spec violation. The theme ramp is for the damask motif ONLY.

Previous wrong values that caused muddy text on Purple Reign:
- `#D4A84B` — too warm/muddy on purple bg → replaced with `#F2C050`
- `#A07830` — too dark on purple bg → replaced with `#C49040`

### Twitch Block Rule
Two separate `fillText` calls — never one combined string in Twitch purple.

```javascript
// ✅ CORRECT
ctx.fillStyle = '#BF94FF'; ctx.fillText('●', W/2 - 100, 866);   // dot only
ctx.fillStyle = '#F2ECFF'; ctx.fillText('LIVE ON TWITCH', W/2 + 18, 866);

// ❌ WRONG — invisible on purple themes
ctx.fillStyle = '#BF94FF';
ctx.fillText('● LIVE ON TWITCH', W/2, 866);
```

### GAS Template Guards (REQUIRED on every HtmlService template)

```javascript
// Guard 1: JSON.parse on template variables
let SCHEDULE, THEME_NAME;
try {
  SCHEDULE   = JSON.parse('<?= scheduleJson ?>');
  THEME_NAME = '<?= themeName ?>';
} catch(e) {
  // fallback for browser preview
  SCHEDULE   = { date: 'SAT  APR  05', lineup: [...], ... };
  THEME_NAME = 'purple';
}

// Guard 2: google.script.run
if (typeof google !== 'undefined' && google.script) {
  google.script.run.receiveFlyerPng(dataUrl);
} else {
  // browser preview — show canvas + download button
}
```

Without Guard 1: `SyntaxError: Unexpected token '<'` on browser open.
Without Guard 2: `ReferenceError: google is not defined` on browser open.

Rule: every GAS HtmlTemplate must open directly in a browser without errors.
If it crashes on direct open, the guards are missing.

### Damask Performance Rule
Pre-render the motif to an offscreen canvas ONCE, then stamp with drawImage().
Never call the motif draw function per-cell on the main canvas.

```javascript
// ✅ CORRECT — build once, stamp many (fast)
const tile = buildTile();
for (let row...) for (let col...)
  ctx.drawImage(tile, x, y);

// ❌ SLOW — redraws all bezier paths per cell
for (let row...) for (let col...)
  drawMotif(ctx, x, y, ramp);
```

### Pre-Export Checklist
Before saving or attaching any flyer PNG:

- [ ] `const white/gold/dim` are hardcoded hex, not `theme.r[x]`
- [ ] Twitch: two fillText calls — dot purple, text white
- [ ] Date: Arial Black, ONE fillText, double spaces between words
- [ ] Y-budget verified — last element + 40px ≤ 1080
- [ ] All 5 themes tested — text readable on Classic AND Purple Reign
- [ ] File opens in browser without console errors (guards in place)

---

## THEME SYSTEM

5 themes rotate weekly. NO GREEN — DJs use green screens.
Approved: Classic, Midnight, Crimson, Sunset, Purple Reign.

Theme switching ONLY changes:
- Damask motif radial gradient color ramp
- Background base gradient
- Top bloom color
- Accent bar color

Theme switching NEVER changes:
- Text colors (white/gold/dim — always the hardcoded constants)
- Layout or y-budget
- Typography rules
- Ornamental rule colors
