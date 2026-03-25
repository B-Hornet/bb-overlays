# Misdiagnosis Prevention — Damask Tile System

## Purpose
This document captures known failure patterns when AI assistants attempt to
modify or regenerate the damask tile drawing code. Each entry describes the
symptom, root cause, and correct approach.

---

## Failure 1: Motif Drawn at 2× Correct Size

**Symptom:** Motif fills entire tile or overlaps adjacent tiles; pattern looks
like overlapping blobs instead of repeating ornaments.

**Root cause:** Drawing coordinates calculated relative to canvas size instead
of tile size. The motif should fit ~80×100 px inside a 118×112 tile.

**Prevention:**
- Tile dimensions `TW=118, TH=112` are HARDCODED constants, not derived from
  canvas dimensions
- All motif coordinates are relative offsets from tile center (±44 vertical,
  ±34 horizontal max)
- NEVER scale motif based on `canvas.width` or `canvas.height`

---

## Failure 2: Blob/Arc Shapes Instead of Bezier Leaf Silhouettes

**Symptom:** Acanthus wing leaves look like semicircles or teardrops instead of
pointed, organic leaf shapes with concave/convex edges.

**Root cause:** Using `ctx.arc()` or simple triangles for leaf shapes.

**Prevention:**
- Wings MUST use `ctx.bezierCurveTo()` with 4 control points per curve
- Each leaf has: pointed tip, concave inner edge, convex outer edge
- Inner leaf is larger; outer leaf is smaller and splayed outward
- Verify visually: leaves should have an asymmetric, botanical silhouette

---

## Failure 3: Motif Center Not Anchored Before Drawing

**Symptom:** Motif elements scattered across canvas; parts don't align;
pattern looks random.

**Root cause:** Drawing motif parts at absolute coordinates instead of
translating to tile center first.

**Prevention:**
- `drawMotif()` MUST call `ctx.save()` then `ctx.translate(cx, cy)` first
- All subsequent coordinates are relative to (0, 0) = tile center
- `ctx.restore()` at end of each motif

---

## Failure 4: No Vein Strokes on Wing Leaves

**Symptom:** Leaves look flat and solid-colored; missing the organic detail
that sells the damask look.

**Root cause:** Vein strokes omitted or commented out.

**Prevention:**
- Each leaf gets 3 vein strokes: 1 center vein (bezier) + 2 branch veins (lines)
- Stroke style: `colors[4]` (lightest), lineWidth 0.5, globalAlpha 0.5
- Veins must follow the leaf curvature, not just be straight lines across

---

## Failure 5: Gradient Stops Not Matching Theme Color Ramp

**Symptom:** Motif appears monochrome, too bright, or colors don't match
the brand theme.

**Root cause:** Using wrong number of gradient stops, linear instead of
radial gradient, or hardcoded colors instead of theme array.

**Prevention:**
- Every gradient uses `createRadialGradient()` with 5 equidistant stops
- Stops come from `THEMES[themeName]` array, indices 0–4
- Stop positions: 0.0, 0.25, 0.5, 0.75, 1.0
- NEVER use `createLinearGradient()` for motif elements

---

## Failure 6: Brick Offset Missing or Incorrect

**Symptom:** All tile rows line up vertically; pattern looks like a simple
grid instead of a staggered damask layout.

**Root cause:** Missing the `xOffset = (row % 2 === 1) ? TW / 2 : 0` offset
for odd rows.

**Prevention:**
- Tiling loop must check `row % 2`
- Odd rows shift left by half a tile width (`TW / 2 = 59 px`)
- Add +1 extra column to account for offset tiles that extend past canvas edge

---

## General Rules for Editing This Code
1. Read `BB_FLYER_SYSTEM.md` BEFORE making any changes
2. Do NOT refactor the motif drawing into SVG or CSS — it must remain Canvas 2D
3. Do NOT change tile dimensions without updating all motif coordinates
4. Test with at least 3 different themes to verify gradient rendering
5. If the pattern looks wrong, check tile size FIRST — it's the #1 root cause
