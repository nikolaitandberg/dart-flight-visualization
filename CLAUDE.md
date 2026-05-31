# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Single-file static web app: `index.html` contains all HTML, CSS, and JavaScript. No build step, no dependencies, no server required — open directly in a browser.

## Architecture

All code lives in `index.html` as one IIFE (`(function() { ... })()`). Key logical sections in order:

- **Constants** — `PARTICLE_COUNT`, `TRAIL_LENGTH`, `BASE_SPEED`, `FIN_STRENGTH`, `WAKE_AMPLITUDE`, etc.
- **`D` (dart dimensions)** — global object set by `initDartDims(cx, cy)` on resize. All geometry (barrel size, fin length, tip length) lives here.
- **`computeFlowVelocity(x, y, params, time)`** — the aerodynamic core. Returns `{vx, vy}` as a sum of: free-stream flow, per-fin dipole perturbations (strength ∝ `sin(θ)`), barrel/tip stagnation, and wake turbulence. This is what drives all particle motion and the pressure heatmap.
- **`isInsideDart(x, y, finAngleDeg)`** — collision detection for particle respawning and heatmap masking.
- **`drawDart(ctx, finAngleDeg)`** — renders tip, barrel, and four fins. In-plane fins spread by `sin(θ/2) * finLen`; out-of-plane fins foreshorten by `cos(θ/2)`.
- **Pressure heatmap** — offscreen canvas at `1/HEATMAP_SCALE` resolution. Recomputed (not every frame) when `heatmapDirty = true`. Uses Bernoulli approximation: `p ∝ 1 − |v|² / (U² * 2.5)`.
- **`Particle` class** — holds position, trail history array, and a per-particle speed multiplier. `step()` does Euler integration; `draw()` colors trails by speed deviation from free-stream.
- **`drawFinDiagram(canvas, finAngleDeg)`** — small cross-section schematic in the sidebar showing the current fin arrangement.
- **`frame()`** — rAF loop: increment `time`, maybe recompute heatmap, clear canvas, draw heatmap → flow arrows → particles → dart.

## Fin angle parameterization

The slider controls the dihedral angle θ (degrees, 10–90). At θ=90° fins form a cross. The dipole strength of each fin scales with `sin(θ)`, so flow deflection and wake width both grow nonlinearly with the slider.
