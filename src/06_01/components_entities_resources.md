# Components, entities and Resources

## Components

In Bevy, components must derive `Component`:

```rs
// port: component.rs

#[derive(Component)]
pub struct Render {
    pub color: ColorPair,
    pub glyph: FontCharType,
}

#[derive(Component)]
pub struct Player;

#[derive(Component)]
pub struct PointC(pub Point);
```

There's something interesting here: `PointC`, which is a tuple struct, unlike the other (regular) structs.

Why so? Because the type `Point` is not defined in our crate, and therefore, can't derive `Component`; in order to work this problem around, we wrap it in our type (`PointC`), and derive the wrapping type.

## Entities

Adding entities to the ECS is trivial both in Legion and Bevy.

The simplest possible form that we can adopt to create an entity (in Bevy lexicon, a "Bundle"), is to insert a tuple:

```rs
// port: spawner.rs

// Note that we generally use systems to manage entities, but in this case, we're strictly following
// the source project's design.
//
pub fn spawn_player(world: &mut World, pos: Point) {
    world.spawn().insert_bundle((
        Player,
        PointC(pos),
        Render {
            color: ColorPair::new(WHITE, BLACK),
            glyph: to_cp437('@'),
        },
    ));
}
```

The entity inserted above has three components (`Player`, `PointC` and `Render`), which are bundled together as tuple.

We can use a more declarative approach, and use an annotated struct:

```rs
#[derive(Bundle)]
struct PlayerBundle {
  player: Player,
  pos: PointC,
  render: Render,
}
```

It's crucial, in this case, to annotate the struct with `#[derive(Bundle)]`; if one forgets this, and accidentally uses a bundle as component, no error will be raised!

Bundles can be nested (and again, we need to annotate the children, this time with `#[bundle]`), although in this project, this feature is not used:

```rs
#[derive(Bundle)]
struct PlayerBundle {
  player: Player,
  pos: PointC,

  #[bundle]
  sprite: SpriteSheetBundle,
}
```

## Resources

Resources, differently from components, currently (this may change in the future!) don't need to be derived; anything inserted in Bevy's ECS is a resource.

```rs
// port: main.rs

// key is a bracket-lib `VirtualKeyCode` type
// ecs is the Bevy `App` instance
//
if let Some(key) = ctx.key {
    self.ecs.insert_resource(key);
} else {
    self.ecs.world.remove_resource::<VirtualKeyCode>();
}
```

Since there can be only one resource for a given type, when a resource is inserted, any existing resource with the same type is going to be overwritten.

Note how in the example above, we remove a resource through `World`; while this is acceptable, we should strive to use systems where possible. The technical reason for this usage is that Bevy provides resource insertion via the `App` instance, but not the removal. This makes sense architecturally, since in the initialization of a project, one insert resources rather than removing them.

Later, we'll see how to access resources in systems.
