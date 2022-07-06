# Designing the states

**WARNING**: This chapter has a conceptual error - in order to define state conditional, a standard resource should be used (and the related conditional API), not `NextState`. This is due to very poor documentation on this topic, and it's something that I'm going to fix shortly.

Now that we've examined all the concepts involved, let's design the Bevy's states.

We'll use the following code as reference, which is the source project as of this step:

```rs
// source: mod.rs

pub fn build_input_scheduler() -> Schedule {
    Schedule::builder()
        .add_system(player_input::player_input_system())
        .flush()
        .add_system(map_render::map_render_system())
        .add_system(entity_render::entity_render_system())
        .build()
}

pub fn build_player_scheduler() -> Schedule {
    Schedule::builder()
        .add_system(collisions::collisions_system())
        .flush()
        .add_system(map_render::map_render_system())
        .add_system(entity_render::entity_render_system())
        .add_system(end_turn::end_turn_system())
        .build()
}

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

## A first, direct translation

In a direct transation, we create one stage for each transaction:

| stage description         | source scheduler number | stage count |
| ------------------------- | :---------------------: | :---------: |
| get player input          |            1            |      1      |
| render player input       |            1            |      2      |
| handle player collisions  |            2            |      3      |
| render player collisions  |            2            |      4      |
| move monsters             |            3            |      5      |
| handle monster collisions |            3            |      6      |
| render move monsters      |            3            |      7      |

All good! This can work, and it's a very straight model. Something important to keep in mind is that, in the game design, if the player doesn't press any key, the subsequent stages won't be executed!

## Simplifying the source project states

Now, let's look at the rendering systems: `map_render::map_render_system()` and `entity_render::entity_render_system()`; we notice two things:

1. they are common to all the schedulers;
2. they need to be run in their separate transaction, otherwise, they may render a partial state (this would be the equivalent of the "read uncommitted" transaction level in database systems).

However, if we consider that this all happens over a single frame, we now see an opportunity for simplification: we just need to render only once! Let's have a look at the new table:

| stage description         | source scheduler number | stage count |
| ------------------------- | :---------------------: | :---------: |
| get player input          |            1            |      1      |
| handle player collisions  |            2            |      2      |
| move monsters             |            3            |      3      |
| handle monster collisions |            3            |      4      |
| render                    |            4            |      5      |

We need a small modification here 😉 If the player doesn't send any input, the rendering won't be executed. Fortunately, we can shift the render:

| stage description         | source scheduler number | stage count |
| ------------------------- | :---------------------: | :---------: |
| render                    |            4            |      1      |
| get player input          |            1            |      2      |
| handle player collisions  |            2            |      3      |
| move monsters             |            3            |      4      |
| handle monster collisions |            3            |      5      |

Excellent. From the user perspective, if we render at the beginning of a frame, or at the end of the previous one, it makes no difference - except that, with the former approach, we've solved the rendering problem.

We can go further! we don't strictly need to sequentially render and get the player input in a sequence; this is a slow game, and rendering a tiny fraction of second after won't make any difference; the new plan is therefore:

| stage description         | source scheduler number | stage count |
| ------------------------- | :---------------------: | :---------: |
| render+get player input   |           4+1           |      1      |
| handle player collisions  |            2            |      2      |
| move monsters             |            3            |      3      |
| handle monster collisions |            3            |      4      |

Notice that when we talk about a tiny fraction of second, it not _one frame_ of delay; the input/rendering systems are rendered in parallel (as opposed to in a sequence), so there is virtually zero delay.

## Summary of the design

With the design above, we now have 4 stages:

- render+get player input
- handle player collisions
- move monsters
- handle monster collisions

We'll also keep the 3 game states of the source project:

- awaiting input
- player turn
- monster turn

Remember: states are used to represent high-level game states, while stages are subdivisions of the states, for transactional purposes.