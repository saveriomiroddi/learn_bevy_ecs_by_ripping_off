# Components and Resources

## Components

In Bevy, components must derive `Component`:

```rs
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

Resources, differently from components, currently (this may change in the future!) don't need to be derived; anything inserted in Bevy's ECS is a resource:

```rs
// WRITEME
```
