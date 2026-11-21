# GameSpice 2D

2D movement and world for a predicted, server-authoritative game: a pure motor, sanitised commands, an analytic arena body, a spatial grid for eating and interest, a deterministic pellet scatter, 2D replication descriptors and an administrator's noclip, freeze and speed.

Read the family [CLAUDE.md](../../CLAUDE.md) first; this file is only what is specific to the 2D layer. Classes are `GS2D*`, because `class_name` is global.

**The distributable is `addons/gs_2d/`.** It requires gs-core and gs-camera (both symlinked, `.gitignore`; `addons/gs_camera`) and names no class outside them: `GS2DCameraRig` takes its zoom and clamp arithmetic from `GSCameraZoom2D`. game-hungario is the one game on it.

## Read before

| Read | Before |
| --- | --- |
| [docs/design.md](docs/design.md) | anything: the pure motor, sanitising, mass order, the scatter's indices, the flat body, the grid, replication, admin modifiers, the config, what is not here |
| [docs/history.md](docs/history.md) | the half-range hash, the doubled grid, the config nothing read |

## Layout

| Path | What |
| --- | --- |
| `core/` | `GS2DCommand` (aim, reach, move, buttons; `sanitise()`), `GS2DState`, `GS2DTunables` (`blob()`), `GS2DMassRules`, `GS2DAdminModifiers` |
| `motion/` | `GS2DMotor` (`simulate()`, pure), `GS2DBody` (the seam), `GS2DBodyFlat` (analytic circles and a rectangle), `GS2DBodyPhysics` (unpredicted entities only) |
| `world/` | `GS2DGrid`, `GS2DScatter`, `GS2DArena` |
| `net/gs_2d_net_sync.gd` | `GS2DNetSync`: what to replicate, as data; 2D codecs, `WORLD_EXTENT`, `ADMIN_BITS` |
| `nodes/` | `GS2DController`, `GS2DSampler` (resolves the pointer), `GS2DCameraRig` |
| `gs_2d_config.gd` | `GS2DConfig`: the optional layered document; `apply_to_controller`, `apply_to_scatter` |
| `examples/gs_2d_selftest.tscn` | the suite; all offline, non-zero on failure |

Paths above, except `examples/`, are in `addons/gs_2d/`.

## Running and testing

The family's two steps ([family CLAUDE.md](../../CLAUDE.md#validating-anything)), then `godot --headless --path . res://examples/gs_2d_selftest.tscn`. Run it after any change to the motor, the grid or the scatter: the determinism check is the property every other guarantee here depends on, and it is not visible by reading the code.

## Rules that bite

- `GS2DMotor.simulate()` stays a pure function: no device, clock, node or randomness. [design](docs/design.md#the-motor-is-a-pure-function)
- Commands carry `aim` + `reach`, never a screen position; the server runs `sanitise()` on every one. [design](docs/design.md#the-pointer-is-resolved-in-the-sampler-not-sent)
- `_apply_mass` runs before the move; change mass through `GS2DController.set_mass()`, never `state.mass`. [design](docs/design.md#mass-is-applied-before-the-move-not-after)
- `GS2DScatter._unit` shifts by 39; a hash check needs quadrants, not just magnitude. [design](docs/design.md#the-hash-returns-its-whole-range)
- `reseed()` never resets the index counter; remove the old field from any grid first. A mirroring peer calls `adopt(index)`, never `refill()`. [design](docs/design.md#the-scatter)
- A predicted entity uses `GS2DBodyFlat`, never `GS2DBodyPhysics`. [design](docs/design.md#dot2dbodyflat-is-production-not-a-stub)
- `GS2DGrid.place()` is add and move; eating uses `query_overlapping`, not `query_circle`. [design](docs/design.md#dot2dgrid-is-why-a-crowded-world-is-possible)
- Interest scales with radius, not mass. [design](docs/design.md#interest-scales-with-size)
- Radius is derived on receipt, never replicated; `WORLD_EXTENT` must cover the arena on both peers. [design](docs/design.md#replication-is-2d-deliberately)
- Admin state lives in `GS2DState.admin`, never `flags` (the game's); a game's behaviour declares `var net_admin: int`, and skips its own geometry when `is_noclipped` inside the replayed function. [design](docs/design.md#an-administrators-noclip-freeze-and-speed-are-in-the-state)
- Name no class outside GameSpice Core: GameSpice Net is reached through `Variant` and type names as strings. [design](docs/design.md#coupling-nothing-is-imported)

Deliberately not here: splitting and merging (`GS2DMassRules` holds numbers nothing here reads), ejecting mass, teams, turret rotation, tile maps and navigation, client interpolation, art. [design](docs/design.md#deliberately-not-here)
