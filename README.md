# No Debug Lag

A Fabric mod for Minecraft **1.21.10** (Fabric Loader **0.19.3**+) that
permanently disables the F3 debug screen's expensive per-frame work and
replaces it with a tiny overlay showing only **facing direction**,
**coordinates**, and **FPS**.

## Why this happens
Opening F3 doesn't just draw text — every single frame `DebugHud` rebuilds
a long list of strings (chunk data, entity counts, memory stats, light
values, frame/tick charts, packet size/ping charts) and only then draws
it. That recalculation, not the drawing, is what costs FPS.

## What this mod does
1. **Blocks it permanently.** A mixin cancels `DebugHud.drawLeftText` /
   `drawRightText` at the very start, so none of that scanning ever runs —
   no toggle key, on by default, stays off until you change it yourself.
2. **Draws a minimal replacement.** A second mixin injects at the tail of
   `InGameHud.render` and draws up to three plain text lines — direction,
   coordinates, FPS — using data Minecraft already has on hand each frame
   (`player.getHorizontalFacing()`, `player.getX/Y/Z()`,
   `client.getCurrentFps()`). No scanning, no charts, effectively free.
3. **Everything is configurable from Mod Menu.** If you have
   [Mod Menu](https://modrinth.com/mod/modmenu) installed, open it → No
   Debug Lag → you can toggle each of the three overlay lines, toggle the
   F3 blocking itself, and pick which screen corner it renders in. Settings
   save to `config/nodebuglag.json`.

Mod Menu isn't required to play — it's only used for the settings screen.
Without it, the mod just runs with its defaults (F3 blocked; direction,
coordinates and FPS all shown, top-left).

## Building
1. Install JDK 21.
2. Open the folder in IntelliJ IDEA — Loom will fetch Minecraft 1.21.10,
   Yarn `1.21.10+build.2`, Fabric API, and (compile-only) Mod Menu
   automatically on first sync (needs internet). Or from a terminal:
   `gradle wrapper` once, then `./gradlew build`.
3. Output jar: `build/libs/nodebuglag-1.1.0.jar`.
4. Requires **Fabric Loader 0.19.3+** and **Fabric API 0.138.4+1.21.10** in
   your mods folder. Mod Menu 16.0.1 is optional, only for the config screen.

## Version-compatibility notes
- `DebugHud`'s methods (`drawLeftText`, `drawRightText`) and
  `InGameHud.render(DrawContext, RenderTickCounter)` were verified against
  **Yarn 1.21.10+build.2** and cross-checked against **1.21.11+build.1** —
  both still use the classic single-phase `DrawContext` rendering path.
- Heads up for future ports: Minecraft's newer "26.x" branch is moving HUD
  rendering to a two-phase extract/draw architecture with different types
  (`GuiGraphicsExtractor`, `DeltaTracker`), and Fabric is dropping Yarn
  mappings in favor of Mojang's official mappings after 1.21.11. If you
  port this past 1.21.11, expect to rewrite `InGameHudMixin` for the new
  API and re-map class/method names — same "crash tells you the exact
  broken method name" situation as before applies to `DebugHudMixin` too.
