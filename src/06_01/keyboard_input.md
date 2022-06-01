# Keyboard input

In Bevy, keyboard input handling is performed via resources:

```rs
fn keyboard_input(keys: Res<Input<KeyCode>>) {
    if keys.just_pressed(KeyCode::W) { /* ... */ }

    // ...
}
```

In this context, we rely on bracket-lib for input handling, so the API above is not used.

The only thing that needs to be kept in mind is that, in the context of the port, we need to remove the key resource:

```rs
// port: main.rs

// key is a bracket-lib `VirtualKeyCode` type
//
if let Some(key) = ctx.key {
    self.ecs.insert_resource(key);
} else {
    self.ecs.world.remove_resource::<VirtualKeyCode>();
}
```

If we forget the `remove_resource` invocation, if the player doesn't press any key, the current frame will still hold the keypress resource from the previous frame!

This is not needed when using Bevy input handling (with an exception: see [here](https://bevy-cheatbook.github.io/programming/states.html#with-input)); it's only necessary in the context of the port.
