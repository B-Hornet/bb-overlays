# BB Flyer System — Damask Background Specification

## Overview
The Beats & Bourbon weekly promo flyer uses a procedurally-generated damask tile
pattern as its background layer. The pattern is drawn on a `<canvas>` element
that sits behind all HTML content inside `.sq-flyer`.

## Canvas Setup
- Element: `<canvas id="bb-bg" width="1080" height="1080">`
- Positioned `absolute`, top-left, full width/height, `z-index: 0`
- All flyer content elements sit at `z-index: 1`

## Tile Dimensions
- **Width:** 118 px (`TW`)
- **Height:** 112 px (`TH`)
- These are absolute pixel values, NOT relative to canvas size

## Tiling Pattern
- **Brick-offset:** Odd rows (row % 2 === 1) are offset by `TW / 2` horizontally
- Tiles cover the full 1080×1080 canvas with +1 overflow column/row
- **Global alpha:** 0.90 applied to entire tile layer

## Motif Anatomy (11 Parts)
Each tile contains one centered ornamental damask motif drawn with Canvas 2D:

| # | Part | Shape | Notes |
|---|------|-------|-------|
| 1 | Crown | 5 bezier petals | Center spike + 2 side + 2 outer |
| 2 | Collar | Ellipse | Below crown |
| 3 | Body (upper) | Narrow rectangle | Column connecting crown to waist |
| 4 | Upper wings | 2 acanthus leaves/side | **Bezier curves, NOT arcs** + vein strokes |
| 5 | Waist | Center ellipse + 2 side ovals | Horizontal accent at motif center |
| 6 | Lower wings | Mirror of upper wings | Same bezier leaf shapes, Y-mirrored |
| 7 | Lower body | Narrow rectangle | Column below waist |
| 8 | Lower collar | Ellipse | Above base |
| 9 | Base | 5 bezier petals | Mirror of crown (Y-flipped) |
| 10 | Center jewel | 3 concentric circles | Outer glow → dark recess → inner dot |
| 11 | Outer oval frame | 2 concentric ellipses | Outermost boundary of motif |

## Acanthus Wing Leaves (Parts 4 & 6)
Each side (left/right) has 2 leaf shapes:
- **Inner leaf:** Larger, closer to body column
- **Outer leaf:** Smaller, splayed outward
- Both drawn with `bezierCurveTo()` — NEVER `arc()` or simple polygons
- Each leaf has **vein strokes**: center vein (bezier) + 2 branch veins (lines)
- Vein strokes: lineWidth 0.5, globalAlpha 0.5, using lightest theme color

## Theme Color Ramps
Each theme provides a 5-stop array `[darkest, dark, mid, light, lightest]`:

| Theme | Stop 0 | Stop 1 | Stop 2 | Stop 3 | Stop 4 |
|-------|--------|--------|--------|--------|--------|
| purple | #1A0A2E | #2D1B4E | #4A2D7A | #6B3FA0 | #8B5CC0 |
| gold | #1A1000 | #3D2B0A | #6B4D1A | #9B7030 | #C49340 |
| teal | #001A1A | #0A2D2D | #1A4D4D | #2A7070 | #3A9090 |
| burgundy | #1A0008 | #2D0A1A | #4D1A2D | #702A40 | #903A55 |
| navy | #000A1A | #0A1A2D | #1A2D4D | #2A4070 | #3A5590 |
| crimson | #1A0000 | #2D0A0A | #4D1A1A | #702A2A | #903A3A |
| emerald | #001A0A | #0A2D1B | #1A4D2D | #2A7040 | #3A9055 |

Colors are applied via `createRadialGradient()` with 5 equidistant stops.

## Theme Mapping from GAS API
The GAS API returns weekly theme names. These map to damask color keys:

| API Theme Name | Damask Color Key |
|---|---|
| Vibes & Vinyl | purple |
| Bass & Bourbon | navy |
| Saturday Soul Sessions | gold |
| The Golden Hour Mix | gold |
| Beats on the Rocks | crimson |
| Late Night Grooves | navy |
| Turntable Therapy | teal |
| The Premium Pour | burgundy |
| Rhythm & Rye | emerald |

## API & Rendering
- `BBDamask.render(themeName)` — main entry point
- Called after schedule data loads; theme extracted from API response
- Fallback: `BBDamask.render('purple')` fires immediately on page load
- The `renderSchedule()` function is monkey-patched to trigger damask render

## Known Failure Modes (What NOT to Do)
1. **Motif at 2× size:** Tile is 118×112, motif fits ~80×100 WITHIN tile
2. **Blob shapes:** Wings MUST use `bezierCurveTo()`, not `arc()`
3. **Unanchored center:** Always `ctx.translate(cx, cy)` before drawing motif
4. **Missing vein strokes:** Each leaf needs center vein + branch veins
5. **Wrong gradient stops:** Must be 5-stop radial, not linear or 2-stop
