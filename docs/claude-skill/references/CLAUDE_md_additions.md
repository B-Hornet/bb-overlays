# CLAUDE.md Additions for bb-overlays

The following content should be appended to CLAUDE.md in the bb-overlays repo.

---

## Damask Background System

### Critical Files
- `promo.html` — Contains the `BBDamask` IIFE module in a `<script>` block
- `docs/claude-skill/references/BB_FLYER_SYSTEM.md` — Full spec
- `docs/claude-skill/references/misdiagnosis-prevention.md` — Known failure modes

### Before Editing Damask Code
1. READ `docs/claude-skill/references/BB_FLYER_SYSTEM.md` first
2. READ `docs/claude-skill/references/misdiagnosis-prevention.md` second
3. The tile is 118×112 px — do NOT change this
4. All motif shapes use bezierCurveTo — do NOT replace with arc()
5. The motif has 11 distinct parts — do NOT simplify or merge them
6. Each part uses a 5-stop radial gradient — do NOT use linear gradients

### Architecture
- `BBDamask` is a self-contained IIFE module (no build system, no dependencies)
- It renders onto `<canvas id="bb-bg">` which sits behind all HTML content
- `BBDamask.render(themeName)` is the only public API
- Theme is extracted from GAS API response via a name→color mapping table
- Fallback theme is 'purple'

### DO NOT
- Run `initialSetup()` in the GAS project — it wipes all sheets
- Run `clasp push` — the owner handles deployment manually
- Replace Canvas 2D drawing with SVG, CSS gradients, or image files
- Change tile dimensions without reading the full spec
- Remove vein strokes from acanthus wing leaves
