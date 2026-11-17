# Dionysus Configuration Reference

Dionysus keeps every server yaml config in one `configs/` directory:

| File | Purpose |
|---|---|
| `configs/bukkit.yml` | Bukkit core settings (permissions file, chunk GC, spawn limits) |
| `configs/spigot.yml` | Spigot settings (view distance, merge radius, messages, growth modifiers) |
| `configs/paper.yml` | Paper settings (light queue, entity activation, anti-xray, timings) |
| `configs/commands.yml` | Command aliases and fallbacks |
| `configs/help.yml` | Custom help topics |
| `configs/dionysus.yml` | Dionysus' own options (documented below) |

Files left in the server root on an old layout are moved into `configs/` automatically on the first start. Nothing is overwritten: a file is only migrated when the target does not already exist, and files whose location you set explicitly on the command line (`--spigot-settings`, `--paper-settings`, `--bukkit-settings`, `--commands-settings`) are never touched.

`server.properties`, `eula.txt`, `log4j2.xml` and the JSON data files (`ops.json`, `whitelist.json`, `banned-players.json`, `banned-ips.json`, `usercache.json`) stay in the server root. The permissions file location remains configurable through `settings.permissions-file` in `configs/bukkit.yml`.

---

## `configs/dionysus.yml`

Every option is optional. Unless stated otherwise the default keeps vanilla behaviour, so an unmodified file changes nothing.

### Performance toggles

| Option | Default | Description |
|---|---|---|
| `crop.optimize-growth` | `false` | Roll crop/stem growth with one `nextFloat()` call instead of `nextInt(bound)`. The growth probability is statistically identical. |
| `pathfinding.optimize` | `false` | Skip A* pathfinding for entities with no player within `pathfinding.max-player-distance` blocks. Invisible to players; useful on mob farms and in remote areas. |
| `pathfinding.max-player-distance` | `32.0` | Radius in blocks used by the above. |
| `scheduled-ticks.optimize` | `false` | Drain the scheduled-tick set with a single `pollFirst()` per entry, hoist the time lookup and avoid per-entry position allocations. |
| `hopper.optimize` | `false` | Skip the hopper suction entity scan when the scanned slices provably contain no candidates. Minecart chests and hoppers are always scanned for. |
| `light-updates-max-ms-per-tick` | `10` | Soft time budget per tick for the lighting queue. This is a soft limit: every tick at least 64 entries per chunk are processed anyway, so a backlog from mass block changes (TNT, explosions) always drains. `0` or negative means no time limit. |

### Behaviour changes

| Option | Default | Description |
|---|---|---|
| `egg-can-knockback` | `true` | Thrown eggs knock players back on hit (0.4 strength). **This changes vanilla combat behaviour**; set to `false` for pure vanilla. |

### I/O and startup

| Option | Default | Description |
|---|---|---|
| `async-json-list-save` | `true` | Save ops/whitelist/ban JSON lists on a background thread. The JSON snapshot is still built on the main thread; pending writes are flushed on shutdown. |

### Status commands

| Option | Default | Description |
|---|---|---|
| `tpsbar.enabled` | `true` | Allow `/tpsbar`. |
| `tpsbar.tick-interval` | `20` | Ticks between refreshes (20 = once per second). |
| `tpsbar.title` | `<tps> TPS, <mspt> MSPT, <ping>ms ping` | Boss bar title. Placeholders: `<tps>`, `<mspt>`, `<ping>`. |
| `tpsbar.fill-mode` | `TPS` | What the bar progress represents: `TPS` or `MSPT`. |
| `tpsbar.progress-color.good` | `GREEN` | Bar colour at TPS ≥ 19 (or MSPT < 40). |
| `tpsbar.progress-color.medium` | `YELLOW` | Bar colour at TPS ≥ 15 (or MSPT < 50). |
| `tpsbar.progress-color.low` | `RED` | Bar colour below those thresholds. |
| `tpsbar.text-color.good` | `GREEN` | Reserved for title colouring; 1.12.2 boss bar titles do not render colour codes, so the bar colour carries the signal. |
| `tpsbar.text-color.medium` | `YELLOW` | See above. |
| `tpsbar.text-color.low` | `RED` | See above. |
| `rambar.enabled` | `true` | Allow `/rambar`. |
| `rambar.tick-interval` | `20` | Ticks between refreshes (20 = once per second). |
| `rambar.title` | `<used>/<xmx> (<percent>)` | Boss bar title. Placeholders: `<allocated>`, `<used>`, `<xmx>`, `<xms>`, `<percent>`. Values use B/K/M/G/T formatting. |
| `rambar.progress-color.good` | `GREEN` | Bar colour below 50% heap usage. |
| `rambar.progress-color.medium` | `YELLOW` | Bar colour below 75% heap usage. |
| `rambar.progress-color.low` | `RED` | Bar colour at or above 75% heap usage. |
| `ping-command.enabled` | `true` | Allow `/ping [player]`. |

### Experimental

| Option | Default | Description |
|---|---|---|
| `use-io-uring` | `false` | **Do not enable this unless you know what it is.** Switches netty to the experimental io_uring transport on Linux instead of epoll. Requires kernel io_uring support (the native library is bundled). This changes the socket channel implementation that packet-manipulation plugins such as ProtocolLib may assume, and io_uring is far less battle-tested than epoll. |

### Power saving

| Option | Default | Description |
|---|---|---|
| `pause-server-when-empty` | `false` | Freeze all world ticking (time, weather, entities, tile entities, mob spawning, chunk unload) while no players are online, and resume the moment a player joins. Server-level work keeps running while paused: network and login handling, the Bukkit scheduler, autosave and console commands. World time and loaded chunks are frozen in place — chunk unloading is also paused, so loaded chunks stay in memory until a player joins. Console log lines mark the pause/resume transitions. |

---

## Commands

| Command | Permission | Description |
|---|---|---|
| `/dionysus [reload\|version]` | `bukkit.command.dionysus` | Reload `dionysus.yml` or print the version. |
| `/tpsbar [on\|off\|toggle]` | `dionysus.command.tpsbar` | Toggle a boss bar showing live TPS/MSPT/ping. |
| `/rambar [on\|off\|toggle]` | `dionysus.command.rambar` | Toggle a boss bar showing heap usage. |
| `/ping [player]` | `dionysus.command.ping` | Show your own connection latency. |
| `/ping <player>` | `dionysus.command.ping.other` | Show another player's latency. |
| `/tps` | `bukkit.command.tps` | TPS over 1m/5m/15m, plus a 5-second average and the last tick duration. |

---

## JVM flags

| Flag | Effect |
|---|---|
| `-DDionysus.enableFMA=true` | Accelerate entity distance calculations with fused multiply-add. **Off by default.** The numeric result may differ from vanilla in the last bit (~4e-16 relative), which is only observable at exact distance-threshold boundaries. |
| `--add-modules=jdk.incubator.vector` | Enables the SIMD optimisations. Only used on Java 17 and 18; on other versions Dionysus logs that SIMD is unavailable and continues without it. |

---

## Notes on defaults

Options that change observable behaviour or that are experimental default to `false`: crop growth RNG, pathfinding skip, scheduled-tick loop, hopper scan skip, and io_uring. Notes:

- The three loop/RNG optimisations (crop, scheduled ticks, hopper) are behaviour-equivalent by construction; they are opt-in so that a bug in one cannot affect servers that did not ask for it.
- `egg-can-knockback` is the one behavioural default (`true`) that differs from vanilla. Set it to `false` if you want stock combat.
- `light-updates-max-ms-per-tick` is active by default and only tunes a budget; it cannot change game state.
