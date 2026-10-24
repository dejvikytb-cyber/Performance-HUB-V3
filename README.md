# Performance Hub V3

A client-side Fabric performance control console for Minecraft Java 1.21.11 (Java 21). This version goes beyond vanilla option presets by adding conservative render-path filters and measured diagnostics.

## Features

### Render path
- **Entity occlusion culling**: after vanilla's own frustum / renderer visibility result passes, tests three sight lines to non-player entities. An entity is skipped only when all three lines hit opaque block collision. Results are cached by entity, camera position/orientation, entity block and configured tick interval. Player entities and unusually large/custom entities are intentionally never hidden.
- **Block-entity culling**: optionally stops submitting distant block entities and uses a separate, cached, per-tick-budgeted set of ray tests to skip block entities hidden behind opaque blocks. This only changes the local render; it does not unload or stop ticking the block entity.
- **Capped culling work**: the per-client-tick occlusion check budget bounds raycast cost to prevent the culler itself becoming a frame-time problem.

### Particle pressure control
- **Particle distance culling**: rejects newly created particles beyond the configured camera radius before they enter the engine.
- **Particle admission budget**: caps accepted particle spawns per client tick, reducing future particle tick and render work.
- **Nearby particle reserve**: distant cosmetic particles cannot consume the reserved share of the budget.
- **Adaptive budget**: gradually lowers the cap when high P95 frame time coincides with particle pressure, and recovers gradually when frames are healthy.

### Diagnostics and UI
- Modern dark control panel (Right Shift).
- Cached performance overlay (F8): FPS, recent frame-time P95, client tick duration, Java heap, GC count, culling and particle counters, plus a small frame-time graph.
- Formatting/metrics strings refresh on a configurable interval, rather than being rebuilt every frame.
- HUD text fitting is cached, and the frame graph is downsampled to at most 52 buckets on the metrics refresh interval, reducing repeated text measurements and graph draw calls.
- Profiles: Competitive, Balanced, Visual. No silent first-run changes by default.
- Settings live in `config/performancehub.json`.

## Important behavior and limitations

- All culling switches affect **local rendering/visual effects only**. They do not modify packets, remote server behavior, collision, mob AI, redstone, or server ticks.
- The tick metric is **client tick duration**, not remote server TPS.
- Entity occlusion culling uses CPU raycasts. The bounded check budget and cache are deliberate: the feature should not replace one bottleneck with another. If a modded entity renders with unusual geometry, disable entity occlusion in the Culling tab.
- Block-entity culling can make distant or fully occluded chests, signs and other block entities disappear earlier than vanilla. Increase the range or disable either BE culling switch if a modded BE is incorrectly hidden.
- Particle budgeting intentionally drops cosmetic effects at high load. Increase the cap or disable the feature if you prefer visual fidelity.
- V3 does **not** replace OpenGL with Vulkan/DirectX. Minecraft's renderer and compatible renderer mods own that pipeline; switching APIs is not a one-setting toggle.
- Sodium, Lithium, ImmediatelyFast and other rendering / simulation optimizers may provide broader optimizations. Performance Hub is designed as a configurable companion rather than a replacement for all of them.

## Build

Requires JDK 21 and network access to resolve Gradle/Fabric/Minecraft dependencies.

```bash
gradle build --no-daemon
```

Output: `build/libs/performance-hub-3.0.0.jar`.

GitHub Actions builds the project on pushes, pull requests and manual workflow runs.
