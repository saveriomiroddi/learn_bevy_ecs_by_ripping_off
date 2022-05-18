# World and systems

## World

In Legion, we work directly on `World` and `Resources`:

```rs
// 06_EntitiesComponentsAndSystems_01_playerecs: main.rs

let mut ecs = World::default();
let mut resources = Resources::default();

// [...]

self.systems.execute(&mut self.ecs, &mut self.resources);
```

In Bevy, the semantics are different, and the idea is to use systems without interacting with the underlying data structures; this is because the engine manages them, and accessing them directly limits the (parallel) scheduling.

We still can manipulate `World`, just as long as we're aware that accessing it mutably, prevents systems from running. In the ported project, in order to maintain the structure similar to the source, I've intentionally done it:

```rs
// 06_EntitiesComponentsAndSystems_01_playerecs: spawner.rs

// The world instance is accessible via App#world.
//
world.spawn().insert_bundle((
    Player,
    PointC(pos),
    Render {
        color: ColorPair::new(WHITE, BLACK),
        glyph: to_cp437('@'),
    },
));
```

Generally speaking, while one can use this approach, in particular in stages where performance is not a concern, using systems should be preferred.

## Systems

WRITEME
