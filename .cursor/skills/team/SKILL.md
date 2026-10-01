---
name: team
description: Smooth the portfolio motion graphics and make the WebGL 3D earth render, spin, and follow scroll correctly. Use when the user invokes /team or asks to improve the planet, globe, earth model, motion graphics, or scroll animation in this portfolio.
disable-model-invocation: true
---

# Team

i want you improve the motion graphic smooth the that the 3d earth model work well

## Scope

All motion and the earth live in `index.html`. Do not add a framework, a bundler, or a second page. Keep the single-file site.

Two module scripts own the motion:

1. Page behaviour: reveal-on-scroll and Lenis (`lenis@1.3.23` from unpkg).
2. WebGL planet: Three `0.143.0` from unpkg, imported as `three` via the import map. The renderer is `THREE.WebGL1Renderer`. Stay on that r143 API (`outputEncoding`, `sRGBEncoding`). Do not upgrade Three unless the user asks.

Earth assets, loaded at runtime:

- `planet.glb`, `planet-lights.glb`, `planet-clouds.png`
- Base: `https://api.getlayers.ai/storage/v1/object/public/public/assets/ascend-d9857ad1f2`

The canvas is `.planet-canvas`: fixed, full viewport, `z-index: 0`, `pointer-events: none`. Page content must stay clickable and scrollable.

## Motion rules

- Drive every animation from one `requestAnimationFrame` loop using a clamped delta (`dt` already capped at `0.05`). Do not add a second rAF loop.
- Scroll position is `0..1` across the document. `sample()` reads `STOPS_X`, `STOPS_Y`, and `STOPS_S` with smoothstep between stops. Follow that curve; do not snap the planet to scroll.
- Ease scroll and transform with the existing critically damped follow (`curP`, then `curX` / `curY` / `curS`). If motion feels late or jittery, change only those rates. Keep movement frame-rate independent.
- Planet spin is `CONFIG.spin` on `planetGroup.rotation.y`, plus scroll travel `curP * Math.PI * 1.6`. Cloud shells spin with `cloud1Spin`, `cloud2Spin`, and `cloud3Spin`.
- Entry rise runs only after `planetLoaded`. Duration is `ENTRY_DUR` (1.9s) with ease-out cubic from `ENTRY_START_Y`.
- `cloudGroup` stays hidden until `planet.glb` loads, then becomes visible. Do not show an empty cloud shell before the mesh exists.
- `prefers-reduced-motion: reduce` must skip Lenis, the typewriter, reveal travel, planet spin, cloud spin, star flicker, and the entry rise. A static earth is correct in that mode.
- OrbitControls stay `enabled: false`. The canvas must not capture pointer or wheel.

## Earth must work

1. Confirm `initScene()` runs and `planet.glb` resolves. A failed load logs `Planet failed to load` and leaves the earth offstage. Fix the loader or the URL before tuning looks.
2. Keep the three-pass render order: torus composer, bloom composer, then final composer. The final pass composites background, flame, bloom, torus, scene, and halo.
3. Cap pixel ratio at 2. On resize, update camera aspect, renderer, all three composers, and bloom resolutions together.
4. If the frame is uneven, simplify cost in this order: lower cloud sphere segments, lower `atmoCount` / `starCount` / `markerCount`, then reduce bloom resolution. Do not remove the earth, the glow, or the scroll path to buy frames.
5. Night lights, ocean flow, atmosphere points, and markers stay attached to the planet group so they spin and scroll with the earth.

## Done when

- The earth mesh is visible, tilted, and spinning smoothly on a normal motion preference.
- Scroll moves the earth along the stop curve without popping or fighting the wheel.
- Clouds appear only after the planet loads, and they orbit at a different rate from the surface.
- Reduced motion shows a still earth and native scroll.
- Text, links, and the CV still receive clicks. The canvas does not block them.
- Check the live page in a browser: load, scroll the full page, resize once, and confirm the console has no planet or WebGL errors.
