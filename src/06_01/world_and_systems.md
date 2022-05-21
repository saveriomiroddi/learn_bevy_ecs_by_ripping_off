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

In Bevy, the semantics are similar, although the idea is to use systems without interacting with the underlying data structures; this is because the engine manages them, and accessing them directly limits the (parallel) scheduling.

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

In Legion, systems require attributes, and queries are manually invoked:

```rs
// 06_EntitiesComponentsAndSystems_01_playerecs: entity_render.rs

#[system]
#[read_component(Point)]
#[read_component(Render)]
pub fn entity_render(ecs: &SubWorld, #[resource] camera: &Camera) {
    // ...

    <(&Point, &Render)>::query()
        .iter(ecs)
        .for_each(|(pos, render)| {
            draw_batch.set(*pos - offset, render.color, render.glyph);
        });

    // ...
}
```

In Bevy, systems are simpler; they don't require attributes, and the queries are declared as function parameters:

```rs
// 06_EntitiesComponentsAndSystems_01_playerecs: entity_render.rs

pub fn entity_render(query: Query<(&PointC, &Render)>, camera: Res<Camera>) {
    // ...

    for (pos, render) in query.iter() {
        draw_batch.set(pos.0 - offset, render.color, render.glyph);
    }

    // ...
}
```

In this example, we perform two accesses:

- we query entities, via `Query` type; in this case, entities with `PointC` and `Render` components
- we retrieve a `Camera` resource, via `Res` (readonly access) type

Another fundamental API component is `Commands`, which is used primarly to add/remove entities and resources.

The following example shows other functionalities:

```rs
// 06_EntitiesComponentsAndSystems_01_playerecs: player_input.rs

pub fn player_input(
    mut commands: Commands,
    mut player_query: Query<&mut PointC, With<Player>>,
    (map, key, mut camera): (Res<Map>, Option<Res<VirtualKeyCode>>, ResMut<Camera>),
) {
  // ...
}
```

There are a few new concepts here (besides `Commands`):

1. `With`: query entities with a certain component, without retrieving it
1. `ResMut`: mutably access a resource
1. `Option<Res>`: access a resource that may not be existing

Note how `Option<Res<T>>` is semantically different from `Res<Option<T>>`:

1. `Option<Res<T>>` is a resource of type `T` that may not be stored
2. `Res<Option<T>>` is a resource of type `<Option<T>>` that is assumed to be stored

if a resource is not stored, the system will panic in the second case, but not the first.

Querying will be explored more in detail in the next chapters.
