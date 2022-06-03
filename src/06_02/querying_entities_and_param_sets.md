# Querying: Entities and Param sets

## Querying entities

In the previous chapter about queries, we've queried, and acted upon, only components. How about we need to act entities?

Both Legion and Bevy use a very similar interface - simply, by adding `Entity` to the query interface.

This is a query example:

```rs
// port: collisions.rs (edited)

pub fn collisions(
    mut commands: Commands,
    enemies_query: Query<(Entity, &PointC), With<Enemy>>,
) {
    for (entity, pos) in enemies_query.iter() {
        // ... act on the entity ...
    }
}
```

The type `Entity` is a classic example of handle; it's a lightweight type that supports common operations (copy, comparison, hash...), and therefore, can be conveniently passed around.

Expanding the example above, we can observe how we actually use it:

```rs
// port: collisions.rs

pub fn collisions(
    mut commands: Commands,
    player_query: Query<&PointC, With<Player>>,
    enemies_query: Query<(Entity, &PointC), With<Enemy>>,
) {
    let player_pos = player_query.single().0;

    for (entity, pos) in enemies_query.iter() {
        if pos.0 == player_pos {
            commands.entity(entity).despawn()
        }
    }
}
```

specifically, we pass it to the entity despawning API, when certain component conditions are met (enemy `pos` matching the player `pos`).

Watch out! As we'll see later, despawning doesn't have an immediate effect.

## Param sets

Param sets are a functionality that is not needed in the port, but it's important to know nonetheless.

Let's say, for the sake of example, that we have the following system:

```rs
// port: collisions.rs (edited)

pub fn collisions(
    player_query: Query<&PointC, With<Player>>,
    mut enemies_query: Query<&mut PointC, With<Enemy>>,
) {
    let player_pos = player_query.single().0;

    for mut pos in enemies_query.iter_mut() {
        if pos.0 == player_pos {
            pos.0.x = 0;
            pos.0.y = 0;
        }
    }
}
```

which model a logic where instead of despawning the enemy entities, we move them away (by changing pos).

The project compiles! But, surprise surprise... once when running, we get a crash:

```
thread 'main' panicked at 'error[B0001]: Query<(Entity, &mut PointC), With<Enemy>>
in system collisions accesses component(s) PointC in a way that conflicts with a
previous system parameter. Consider using `Without<T>` to create disjoint Queries
or merging conflicting Queries into a `ParamSet`.', /home/user/.cargo/registry/src/github.com-1ecc6299db9ec823/bevy_ecs-0.7.0/src/system/system_param.rs:173:5
stack backtrace:
   0: rust_begin_unwind
             at /rustc/4cbaac699c14b7ac7cc80e54823b2ef6afeb64af/library/std/src/panicking.rs:584:5
   1: core::panicking::panic_fmt
             at /rustc/4cbaac699c14b7ac7cc80e54823b2ef6afeb64af/library/core/src/panicking.rs:142:14
   2: bevy_ecs::system::system_param::assert_component_access_compatibility
             at /home/user/.cargo/registry/src/github.com-1ecc6299db9ec823/bevy_ecs-0.7.0/src/system/system_param.rs:173:5
   3: <bevy_ecs::query::state::QueryState<Q,F> as bevy_ecs::system::system_param::SystemParamState>::init
             at /home/user/.cargo/registry/src/github.com-1ecc6299db9ec823/bevy_ecs-0.7.0/src/system/system_param.rs:113:9
   4: <(P0,P1,P2) as bevy_ecs::system::system_param::SystemParamState>::init
             at /home/user/.cargo/registry/src/github.com-1ecc6299db9ec823/bevy_ecs-0.7.0/src/system/system_param.rs:1247:21
```

This is interesting; we're prevented from query `PointC` in two queries, with incompatible access mode. Bevy doesn't enforce incompatible access compile-time, but it does enforce it runtime.

In order to solve this problem, one of the two solutions referenced is to use `ParamSet`:

```rs
pub fn collisions(
    mut queries: ParamSet<(
        Query<&PointC, With<Player>>,
        Query<&mut PointC, With<Enemy>>,
    )>,
) {
    let player_pos = queries.p0().single().0;

    for mut pos in queries.p1().iter_mut() {
        if pos.0 == player_pos {
            pos.0.x = 0;
            pos.0.y = 0;
        }
    }
}
```

There's no significant difference; we just group the queries inside a `ParamSet`, and access them via `pN()` functions. Considering the constraint, this is impressively ergonomic!
