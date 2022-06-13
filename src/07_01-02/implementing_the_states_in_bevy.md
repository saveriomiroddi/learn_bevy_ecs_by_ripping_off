# Implementing the states in Bevy

In this chapter, we'll use the `iyes_loopless` Bevy crate!

## Adding the stages

First, let's define the stages:

```rs
// Port: game_stage.rs

#[derive(Debug, Clone, Eq, PartialEq, Hash, StageLabel)]
pub enum GameStage {
    MovePlayer,
    PlayerCollisions,
    MoveMonsters,
    MonsterCollisions,
}
```

Is there one missing? No! For simplicity, we're going to use Bevy's standard one, `Update`, for the rendering. It's perfectly possible to leave `Update` untouched, and add an extra state for the rendering; it's up to the developer's taste.

Now, we'll need to register them; this is performed via some `App` APIs, of which, for simplicity, we'll use just one, `add_stage_after`:

```rs
// Port: main.rs

ecs.add_stage_after(CoreStage::Update, MovePlayer, SystemStage::parallel())
    .add_stage_after(MovePlayer, PlayerCollisions, SystemStage::parallel())
    .add_stage_after(PlayerCollisions, MoveMonsters, SystemStage::parallel())
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

This is again, an API of `App`.

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
