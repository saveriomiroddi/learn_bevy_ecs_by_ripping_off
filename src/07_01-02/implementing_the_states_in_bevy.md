# Implementing the states in Bevy

**WARNING**: This chapter has a conceptual error - in order to define state conditionals, a standard resource should be used (and the related conditional API), not `NextState`. This is due to very poor documentation on this topic, and it's something that I'm going to fix shortly.

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

Now, let's define the states:

```rs
// Port: turn_state.rs

#[derive(Debug, Clone, Eq, PartialEq, Hash)]
pub enum TurnState {
    AwaitingInput,
    PlayerTurn,
    MonsterTurn,
}
```

We don't need to register the states, but we need to tell the scheduler which is the starting one:

```rs
// Port: main.rs

ecs.add_loopless_state(TurnState::AwaitingInput);
```

This is again, an API of `App` (but added by `iyes_loopless`).

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
            .run_in_state(AwaitingInput)
    );

    app.add_system_set_to_stage(
        GameStage::MovePlayer,
        ConditionSet::new()
            .run_in_state(TurnState::PlayerTurn)
            .with_system(collisions::collisions)
            .with_system(end_turn::end_turn)
            .into(),
    );

    app.add_system_set_to_stage(
        GameStage::MoveMonsters,
        ConditionSet::new()
            .run_in_state(TurnState::MonsterTurn)
            .with_system(random_move::random_move)
            .into(),
    );

    app.add_system_set_to_stage(
        GameStage::MonsterCollisions,
        ConditionSet::new()
            .run_in_state(TurnState::MonsterTurn)
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
            .run_in_state(AwaitingInput)
    );
```

This also goes in `Update`, however, we slot it in the associated game state (`AwaitingInput`).

Now, the player move:

```rs
    app.add_system_set_to_stage(
        GameStage::MovePlayer,
        ConditionSet::new()
            .run_in_state(TurnState::PlayerTurn)
            .with_system(collisions::collisions)
            .with_system(end_turn::end_turn)
            .into(),
    );
```

There are a few notable things here. First, we need to invoke `ConditionSet#into()` - this is because such type belongs to the `iyes_loopless` crate, and it needs to be converted to make it compatible with Bevy's `add_system_set_to_stage` APIs.

Then, we're now adding the system set to a stage (`MovePlayer`).

Like for rendering, we run the systems in parallel. Since we are sure that Bevy will wait for both systems to complete before moving to the next stage (`MoveMonsters`), there's no risk of a race condition.

The rest of the systems follow the same structure.

A small detail not to forget is turn state change is performed by the state machine system `end_turn`, but not in the `AwaitingInput` state - in this case, it's performed by the `player_input` system (since it depends on the user input).
