# Managed event passing

This chapter presents an interesting departure from the source project! We're going to implement event passing; this is interesting because it's possible to observe the improvements offered compared to manual event handling (which is the implementation chosen in the source project).

## A look at a manual implementation

Let's see what the source project implementation. Since the messages are regular entities, first, they need to be declared:

```rs
// Source: component.rs

#[derive(Clone, Copy, Debug, PartialEq)]
pub struct WantsToMove {
    pub entity: Entity,
    pub destination: Point,
}
```

Now, they're simply added to the world:

```rs
// Source: player_input.rs (edited)

pub fn player_input(
    commands: &mut CommandBuffer,
) {
    // ...

    // Legion’s push function doesn’t work with single-component insertions

    commands.push((
        (),
        WantsToMove {
            entity: *player_entity,
            destination: player_destination,
        },
    ));

    // ...
}
```

Here we observe a small limitation of Legion, in particular, when used for events - it doesn't accept insertion of individual components, so we need to add a phony one (the empty tuple `()`).

Reading is performed via a standard query:

```rs
// Source: movement.rs (extract)

pub fn movement(
    entity: &Entity,
    want_move: &WantsToMove,
    // ...
    commands: &mut CommandBuffer,
) {
    if map.can_enter_tile(want_move.destination) {
        // ... operate on `want_move`
    }

    commands.remove(*entity);
}
```

Note how we have to manually remove the pseudo-message entity.

That's all. This implementation is very simple, which works well in the design of this game.

## Bevy's Event passing

Bevy's managed event passing is equally simple. We define the event type, and we register it:

```rs
// Port: events.rs

pub struct WantsToMove {
    pub entity: Entity,
    pub destination: Point,
}

// Port: main.rs

ecs.add_event::<WantsToMove>();
```

Note that differently from entities, message types don't need to be derived as components; in the port, this gives a minor simplification - we don't need to use the wrapper component `PointC` - we use just `Point`.

Now, let's send the events:

```rs
// Port: player_input.rs (edit)

pub fn player_input(
    mut move_events: EventWriter<WantsToMove>,
) {
    // ... compute the player destination ...

    move_events.send(WantsToMove {
        entity: *player_entity,
        destination: player_destination,
    });

    // ...
}
```

and read them:

```rs
// Port: movement.rs (edit)

pub fn movement(
    mut move_events: EventReader<WantsToMove>,
    // ...
) {
    for &WantsToMove {
        entity,
        destination,
    } in move_events.iter()
    {
        if map.can_enter_tile(destination) {
            // ... operate on entity/destination
        }
    }
}
```

Very straightforward. There is a very important difference of the port: we don't need to remove the message; this is taken care of by Bevy.

## Systems ordering

There are two crucial concepts to be aware of, when using Bevy's events:

1. events persist for at most two frames - the frame where the event is sent, and the next;
2. the developer must take care of the systems ordering, in order to read the events as soon as possible.

In some games, lag is not a problem, however, in some others, if the systems ordering is improperly designed, event reading may be lag, and the lag can propagate to other systems/frames.

It's therefore useful to review how system ordering is designed in the port, at this stage:

```rs
// Port: mod.rs (extract)

// Here we write the event; this is the default (Update) stage.
//
app.add_system(
    player_input::player_input.run_in_state(AwaitingInput)
);

// Here we read the events; the stage is the next, which guarantees the ordering.
//
app.add_system_set_to_stage(
    MovePlayer,
    ConditionSet::new()
        .run_in_state(PlayerTurn)
        .with_system(movement::movement) // reads the event
        .into(),
);
```

We don't need to introduce any syncing (ie. extra stages) - the project's states and actions are well-defined, so the sync points (implemented via stages) fit cleanly and, in this case, for free.

## Considerations

Whether to use or not managed event passing, is a valid question; after all, one can just write entities, send them around, then remove them. In the context of this project, the advantages that managed events passing gives are:

- it avoids manual removal of the messages;
- the implementation is semantically more consistent with its intent;
- as a consequence of the previous point, workarounds are not necessary.

These don't constitute a very significant improvement, however, given the simplicity of Bevy's API, in my opinion, using it is a no-brainer.

One larger projects, when there are multiple readers, the advantage of not having to manually clear the events, is definitely more significant.
