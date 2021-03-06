# Implementing the states in Bevy

In this chapter, we'll use the `iyes_loopless` Bevy crate!

## Adding the stages

First, let's define the stages:

```rs
// Port: game_stage.rs

#[derive(Debug, Clone, Eq, PartialEq, Hash, StageLabel)]
pub enum GameStage {
    MovePlayer,
    MoveMonsters,
    MonsterCollisions,
}
```

Is there one missing? No! For simplicity, we're going to use Bevy's standard one, `Update`, for the rendering and player input. It's perfectly possible to leave `Update` untouched, and add an extra state for those systems; it's up to the developer's taste.

Now, we'll need to register them; this is performed via some `App` APIs, of which, for simplicity, we'll use just one, `add_stage_after`:

```rs
// Port: main.rs

ecs.add_stage_after(CoreStage::Update, MovePlayer, SystemStage::parallel())
    .add_stage_after(MovePlayer, MoveMonsters, SystemStage::parallel())
    .add_stage_after(MoveMonsters, MonsterCollisions, SystemStage::parallel());
```

Note that we're specifying that we want to run the systems of each stage in parallel (the alternative is `SystemStage::single_threaded()`), taking advantage of Bevy's ECS.

## Adding the states

Now, let's define the states, which are encoded using a standard Bevy resource:

```rs
// Port: turn_state.rs

#[derive(Debug, Clone, Eq, PartialEq, Hash)]
pub enum TurnState {
    AwaitingInput,
    PlayerTurn,
    MonsterTurn,
}
```

We set the initial state by adding the resource to the ECS:

```rs
// Port: main.rs

ecs.insert_resource(TurnState::AwaitingInput);
```

In the next section, we'll encode the conditions.

## Setting the `SystemSet`s

Now we can add the `SystemSet`s.

```rs
// Port: mod.rs (edited)

pub fn build_system_sets(app: &mut App) {
    app.add_system_set(
        SystemSet::new()
            .with_system(map_render::map_render)
            .with_system(entity_render::entity_render),
    );

    app.add_system(
        player_input::player_input
            .run_if_resource_equals(TurnState::AwaitingInput)
    );

    app.add_system_set_to_stage(
        GameStage::MovePlayer,
        ConditionSet::new()
            .run_if_resource_equals(TurnState::PlayerTurn)
            .with_system(collisions::collisions)
            .with_system(end_turn::end_turn)
            .into(),
    );

    app.add_system_set_to_stage(
        GameStage::MoveMonsters,
        ConditionSet::new()
            .run_if_resource_equals(TurnState::MonsterTurn)
            .with_system(random_move::random_move)
            .into(),
    );

    app.add_system_set_to_stage(
        GameStage::MonsterCollisions,
        ConditionSet::new()
            .run_if_resource_equals(TurnState::MonsterTurn)
            .with_system(collisions::collisions)
            .with_system(end_turn::end_turn)
            .into(),
    );
}
```

The state handling is done; let's review it.

First, rendering:

```rs
    app.add_system_set(
        SystemSet::new()
            .with_system(map_render::map_render)
            .with_system(entity_render::entity_render),
    );
```

By not specifying the stage, we're adding those systems to the `Update` stage. We're also not specifying a state, as we don't strictly need it; we could bind it to the `AwaitingInput` state, but conceptually speaking, there isn't such connection.

Note how there is no temporal dependency between the two rendering systems (therefore, they will run in parallel); since they can execute independently (they draw to two different planes), we take the chance to parallelize them.

Now, the player input:

```rs
    app.add_system(
        player_input::player_input
            .run_if_resource_equals(TurnState::AwaitingInput)
    );
```

This also goes in `Update`, however, we slot it in the associated game state (`AwaitingInput`). Note how we encode the conditional using the `run_if_resource_equals()` `iyes_loopless` API; it allows a system/set to run if the ECS includes an enum resource whose variant is the one specified.

It's extremely important *not* to use the `iyes_loopless` state management API (`run_in_state()`) for this purpose! While both are backed by resources, the difference is that the decision whether to run each system/set, is taken:

- with the `run_in_state()` condition, once per frame, at the beginning of it;
- with the `run_if_resource_equals()` condition, at the beginning of each stage.

The consequence is that if we set a new state (by updating the resource) in a given stage, systems in subsequent stages with conditions encoded via `run_in_state()`, will not run, even if they match the new state, because the decision not to run them was taken at the beginning of the frame!

Now, the player move:

```rs
    app.add_system_set_to_stage(
        GameStage::MovePlayer,
        ConditionSet::new()
            .run_if_resource_equals(TurnState::PlayerTurn)
            .with_system(collisions::collisions)
            .with_system(end_turn::end_turn)
            .into(),
    );
```

There are a few notable things here. First, we need to invoke `ConditionSet#into()` - this is because such type belongs to the `iyes_loopless` crate, and it needs to be converted to make it compatible with Bevy's `add_system_set_to_stage` APIs.

Then, we're now adding the system set to a stage (`MovePlayer`).

Like for rendering, we run the systems in parallel. Since we are sure that Bevy will wait for both systems to complete before moving to the next stage (`MoveMonsters`), there's no risk of a race condition.

The rest of the systems follow the same structure.

A small detail not to forget is turn state change is performed by the state machine system `end_turn`, but not in the `AwaitingInput` state - in this case, it's performed by the `player_input` system (since it depends on the user input):

```rs
// Port: player_input.rs (extract)

commands.insert_resource(TurnState::PlayerTurn);
```

as you can see, changing state is just a matter of updating the state resource.
