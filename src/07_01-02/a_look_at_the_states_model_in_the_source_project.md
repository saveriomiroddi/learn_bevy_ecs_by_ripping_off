# A look at the states model in the source project

## Source structure, and Bevy's `Stage`s

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

## Commands flushing and `Stage`s modeling

In the source schedulers, there's something interesting - the `flush()` commands:

```rs
// source: mod.rs

pub fn build_monster_scheduler() -> Schedule {
    Schedule::builder()
        .add_system(random_move::random_move_system())
        .flush()
        .add_system(collisions::collisions_system())
        .flush()
        .add_system(map_render::map_render_system())
        .add_system(entity_render::entity_render_system())
        .add_system(end_turn::end_turn_system())
        .build()
}
```

This is a crucial concept to understand!

When ECSs like Legion or Bevy modify entities/components, they don't commit the changes immediately; if one accidentally forgets flushing, it will lead to a very confusing situation, because systems won't find the expected data.

Compared to (current) Bevy, Legion has an advantage - as seen above, Legion has an API for flushing at will, while Bevy doesn't (as of Jun/2022, there's an open issue).

Fortunately, we can easily model this behavior using `Stage`s - when a `Stage` is completed, the changes are committed.

Bevy's model has a disadvantage, though: we'll need to model more states than we do in Legion. In the source project, there are three schedulers, and that's all; in Bevy, we'll need:

- three states, one for each Legion scheduler
- seven stages, one for each commands transaction

This is not dramatic, but there is certainly an increase of the complexity that we'll need to handle; fortunately, in our case, we can reduce the number of stages.
