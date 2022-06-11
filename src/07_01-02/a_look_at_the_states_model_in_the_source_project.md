# A look at the states model in the source project

Before getting technical, we need to have a look at how the source project models the states:

```rs
// source: mod.rs (edited)

pub fn build_input_scheduler() -> Schedule {
    Schedule::builder()
        .add_system(player_input::player_input_system())
        .add_system(map_render::map_render_system())
        .add_system(entity_render::entity_render_system())
}

pub fn build_player_scheduler() -> Schedule {
    Schedule::builder()
        .add_system(collisions::collisions_system())
        .add_system(map_render::map_render_system())
        .add_system(entity_render::entity_render_system())
        .add_system(end_turn::end_turn_system())
}

pub fn build_monster_scheduler() -> Schedule {
    Schedule::builder()
        .add_system(random_move::random_move_system())
        .add_system(collisions::collisions_system())
        .add_system(map_render::map_render_system())
        .add_system(entity_render::entity_render_system())
        .add_system(end_turn::end_turn_system())
}
```

We observe that there are three states:

1. player input
1. collision handling (following player move)
1. monster moving, and collision handling

Every state performs rendering; state change is performed in two different ways:

- on the player input state, performed by the `player_input` system, when the user presses a key;
- on the other states, performed by an adhoc system.

As previously mentioned, Bevy does have a concept semantically similar to Legion's `Schedule`, however, it's more complicated, and it's better/closer modeled by a set of other types.