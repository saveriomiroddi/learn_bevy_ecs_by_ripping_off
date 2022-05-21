# Components and Resources

## Components

In Bevy, components must derive `Component`:

```rs
// 06_EntitiesComponentsAndSystems_01_playerecs: component.rs

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

## Resources

Resources, differently from components, currently (this may change in the future!) don't need to be derived; anything inserted in Bevy's ECS is a resource.

```rs
// 06_EntitiesComponentsAndSystems_01_playerecs: main.rs

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

Note how int the example above, we remove a resource through `World`; while this is acceptable, we should strive to use systems where possible. The technical reason for this usage is that Bevy provides resource insertion via the `App` instance, but not the removal. This makes sense architecturally, since in the initialization of a project, one insert resources rather than removing them.

In the next chapters, we'll see how to access resources in systems.
