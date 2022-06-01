# Introduction to System sets

One fundamental different between the architectures of the source design (Legion) and the port (Bevy) is how systems are grouped and scheduled.

In Legion, this is very simple; all the systems that belong to a (conceptual) state, are grouped together into a a "Schedule", which is sent to Legion, to be run on each update:

```rs
// 06_EntitiesComponentsAndSystems_01_playerecs: mod.rs

Schedule::builder()
    .add_system(player_input::player_input_system())
    .add_system(map_render::map_render_system())
    .add_system(entity_render::entity_render_system())
    .build()

// 06_EntitiesComponentsAndSystems_01_playerecs: main.rs

// `self.systems` is the Schedule instance built above.
//
self.systems.execute(&mut self.ecs, &mut self.resources);
```

Bevy uses similar semantics as well, however, they are more conceptually more complex; the closest concept in Bevy is the system of `Stage`s and `SystemSet`s. In this chapter, we'll explore the `SystemSet`s.

## Warning

As of v0.7, Bevy's states are practically unusable, except for trivial programs; this is due to the internal APIs used to implement them (see [here](https://bevy-cheatbook.github.io/programming/states.html#combining-with-other-run-criteria)). Trying, for example, to use the standard `SystemSet`/`Stage` Bevy APIs for this project, will cause the program to hang.

Fortunately, there is a third party crate, [iyes_loopless](https://github.com/IyesGames/iyes_loopless), that solves this problem, while also maintaining the semantics identical (or very close) to Bevy's, with a clean interface.

The rest of this book assumes that the `iyes_loopless` crate is used:

```toml
# Cargo.toml

[dependencies]
bevy = "0.7.0"
iyes_loopless = "0.5.1"
```

## `SystemSet`s

A `SystemSet` in Bevy, is the abstraction that allows grouping and scheduling the systems, based on user-provided properties.

At this step, only the grouping functionality is used:

```rs
// 06_EntitiesComponentsAndSystems_01_playerecs: mod.rs

SystemSet::new()
    .with_system(player_input::player_input)
    .with_system(map_render::map_render)
    .with_system(entity_render::entity_render)
```

The above is the simplest form that a `SystemSet` can have: an anonymous group of systems that will run in parallel.

Don't forget the *parallel* - by default, Bevy runs the systems in parallel, so consider this when designing the system.

Another extremely important concept is that changes applied by a system to entities, are **not** seen by systems in the same system set. This doesn't matter now (the renderings are independent of the player entity properties), so we'll get back on this later.
