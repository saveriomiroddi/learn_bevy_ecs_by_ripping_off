# Base structure

The very first step in the port is plugging Bevy's ECS. The source project has a mixed, game engine-wise, design:

- the game loop is managed by the game engine ([bracket-lib](https://github.com/amethyst/bracket-lib));
- for each cycle (frame), an update is invoked on the ECS ([Legion](https://github.com/amethyst/legion)).

This is bare-bones structure of the source project:

```rs
struct State {
    ecs: World,
    resources: Resources,
    systems: Schedule,
}

impl State {
    fn new() -> Self {
        // [...]
        Self {
            ecs: World::default(),
            resources: Resources::default(),
            systems: build_scheduler(),
        }
    }
}

impl GameState for State {
    fn tick(&mut self, ctx: &mut BTerm) {
        // [...]
        self.systems.execute(&mut self.ecs, &mut self.resources);
        render_draw_buffer(ctx).expect("Render error");
    }
}

fn main() -> BError {
    // [...]
    main_loop(context, State::new())
}
```

The translation is straightforward:

- `World` and `Resources` map to [`App`](https://github.com/bevyengine/bevy/blob/83c6ffb73c4a91182cda10141f824987ef3fba2f/crates/bevy_app/src/app.rs#L46) (which also owns the resources)
- Legion's systems update for each tick maps to the [`App#update()` API](https://github.com/bevyengine/bevy/blob/83c6ffb73c4a91182cda10141f824987ef3fba2f/crates/bevy_app/src/app.rs#L111).

leading to:

```rs
struct State {
    ecs: App,
}

impl State {
    fn new() -> Self {
        // [...]
        Self { ecs: App::new() }
    }
}

impl GameState for State {
    fn tick(&mut self, ctx: &mut BTerm) {
        // [...]
        self.ecs.update();
        render_draw_buffer(ctx).expect("Render error");
    }
}
```
