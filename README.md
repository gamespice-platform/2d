Deterministic 2D movement and world code for Godot 4, for games seen from above. It is the 2D counterpart of [dot-player-controller](https://github.com/modcommunity/dot-player-controller) and part of the [dot-*](https://github.com/modcommunity) family. It needs [dot-core](https://github.com/modcommunity/dot-core) and nothing else.

It gives you three movement feels (top-down, thrust, and mass-based blob), a spatial hash for fast neighbour queries, seeded pellet fields, and `Vector2` replication specs. [game-hungario](https://github.com/zees-games/game-hungario) is built on it.

## Install

1. Copy `addons/dot_2d/` and `addons/dot_core/` into your project.
2. Enable both in *Project > Project Settings > Plugins*.

## Use

```gdscript
var world := Dot2DArena.new()
world.bounds = Rect2(Vector2(-2000, -2000), Vector2(4000, 4000))
add_child(world)

var player := Dot2DController.new()
player.tunables = Dot2DTunables.blob()
add_child(player)
player.attach(world, session_id)

# Once per simulated tick, on client and server alike.
player.simulate_tick(tick, 1.0 / 60.0)
world.sync_grid()

# Who can this blob eat?
for id in world.overlapping(state.position, state.radius, my_id):
    ...
```

## How it works

The simulation is a pure function of a `Dot2DCommand`: no device, no clock, no node, no randomness. A client predicting a move, a server re-running it and a reconciliation replay all reach the same position.

The sampler turns the pointer into a direction and a world-space distance before the command is sent. A screen position means nothing on a server that has no window or camera.

## Classes

| Class | What it does |
| --- | --- |
| `Dot2DCommand` | What a player asked for, sanitised against hostile input. |
| `Dot2DState` | Position, velocity, facing, radius and mass. Can be snapshotted and replayed. |
| `Dot2DTunables` | How the entity moves: top-down, thrust or blob. |
| `Dot2DMassRules` | How mass becomes size, speed and the right to eat. |
| `Dot2DMotor` | The simulation itself. Pure and node-free. |
| `Dot2DAdminModifiers` | Admin noclip, freeze and speed step, carried in the state so the owning client predicts them. |
| `Dot2DBody` | Collision. `Flat` is for a bounded rectangle, `Physics` for everything else. |
| `Dot2DGrid` | A uniform spatial hash for local queries. |
| `Dot2DScatter` | Pellet fields laid out from a seed and refilled on a per-tick budget. |
| `Dot2DArena` | The world: bounds, grid, interest rectangles and spawn positions. |
| `Dot2DController` | Drives one entity, local, commanded or remote. |
| `Dot2DSampler` | Turns devices into commands. The only place input is read. |
| `Dot2DCameraRig` | Follows the entity, zooms with size and stays inside the world. |
| `Dot2DNetSync` | What to replicate, without naming a dot-net type. |

## Mass rules

`Dot2DMassRules` keeps the three blob relationships in one place so they agree. These are the defaults and can be changed in the rules resource.

| Rule | Default |
| --- | --- |
| Radius | Grows with mass^0.5, so two small blobs equal one big one by area. |
| Speed | Falls with mass^-0.44, down to a floor so the biggest blob never stops. |
| Eating | Needs both a mass ratio and an overlap. `can_eat` checks both in one call. |

## Collision backends

`Dot2DBodyFlat` is the one to use in production. It is analytic, and a client and a server get bit-identical answers. A physics query depends on solver state from earlier steps, so it cannot promise that. `Dot2DBodyPhysics` is for unpredicted entities and single-player games.

## Limits

| Setting | Default | Meaning |
| --- | --- | --- |
| `Dot2DMassRules.max_pieces` | 16 | Most pieces one player may have; 0 = unlimited, negative is refused by `validate()`. |

## Test

```bash
godot --headless --path . --import
godot --headless --path . res://examples/dot_2d_selftest.tscn
```

The self-test runs offline and exits non-zero on any failure.

## Status

Built and maintained with Claude Code. Every addon has a headless test suite and the suites pass, but little of this has been in front of real players yet. Please report rough edges by opening an issue.

## Licence

MIT. See [LICENSE](LICENSE). Only the TMC back-end web infrastructure is not open source; if you use your own back end instead of TMC's, you build that part yourself.
