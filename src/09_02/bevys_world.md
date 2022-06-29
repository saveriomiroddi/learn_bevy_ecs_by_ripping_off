# Bevy's World

`World` was briefly mentioned in the [World and systems](../06_01/world_and_systems.md) chapter. In general, there are two ways of accessing the `World` instance: from the `App` instance, and from a system.

We've seen the first case:

```rs
// port: app.rs

let mut ecs = App::new();
// ...
spawn_player(&mut ecs.world, map_builder.player_start);

// port: spawner.rs

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

There's nothing notable in it; we're at the initialization state of the application, so we don't have specific concerns.

In order to get access to `World` from a system, we just specify it as parameter:

```rs
pub fn world_read_system(world: &World) {
    if let Some(player) = world.entity(entity).get::<Player>() {
        // ...
    }
}
```

## Exclusive systems

In the case of mutable `World` access, there are significant constraints.

First, we need to mark the system as "exclusive" when adding it:

```rs
pub fn world_write_system(world: &mut World) {
    if let Some(player) = world.entity(entity).get_mut::<Player>() {
        // ...
    }
}

app.add_system(world_write_system.exclusive_system())
```

Second: mutable access to `World` is incompatible with other queries, for obvious reasons:

```rs
// This is not allowed!
//
pub fn not_working_exclusive_system(
  world: &mut World,
  timer: ResMut<MyTimer>,
) {
  // in theory, here, we we'd have aliasing of the MyTimer resource
}
```

Finally, and very importantly, exclusive systems can't be scheduled concurrently with other systems, potentially causing bottlenecks!

## Clearing the world

In some cases, we want to clear the entities in the world, for example, in order to partially perform a state cleanup (don't forget the resources!); this is easily accomplished:

```rs
world.clear_entities();
```

As of Jun/2022, there is no way to clear resources; however, while there's a [PR open](https://github.com/bevyengine/bevy/pull/3212), it must be very clear that one can't just clear all the resources, since Bevy uses resources for various tasks - fully clearing the world would likely bring Bevy to an unusable state.
