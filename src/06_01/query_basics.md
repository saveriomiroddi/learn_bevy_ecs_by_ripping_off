# Query basics

Queries are a shiny example of Bevy's capabilities. Putting aside performance, any project will certainly benefit from their ergonomy.

Their distinctive trait is that they're entirely, and automatically, defined by the systems function signature, while allowing at the same time a great amount of flxeibility.

## Basic queries definition and iteration

This is a basic (edited) example:

```rs
pub fn entity_render(query: Query<(&PointC, Option<&Render>)>) {
    for (pos: &PointC, render: Option<&Render>) in query.iter() {
        if let Some(render) = render {
          // ...
        }
    }
}
```

Here, we query all the entities that have a `PointC`, and optionally a `Render` component, and retrieve both of them in the query.

Iteration can be mutable or immutable; in order to make the above mutable:

- make `query` mutable: `mut Query<(&PointC, Option<&Render>)>`
- use the mutable iterator API: `query.iter_mut()`

## Resources and commands

In addition to Queries, systems are also typically provided access to commands and resources:

```rs
pub fn player_input(
    mut commands: Commands,
    mut player_query: Query<&mut PointC, With<Player>>, //(1) (2)
    (map, key, mut camera): (Res<Map>, Option<Res<VirtualKeyCode>>, ResMut<Camera>),
)
```

Commands are used to perform write operations, typically on entities and resources; while the details will be described later, in this context, an example invocation is:

```rs
commands.remove_resource::<VirtualKeyCode>();
```

which removes a resource from the storage.

We can observe resources access on the third set of parameters; its definition is intuitive, and the main information we need to know is:

- if a resource may not exist, it must be accessed via `Option<Res<T>>`;
- immutable/mutable access to a resource are performed via, respectively, the types `Res`/`ResMut`.

Queries resources implement `Deref`/`DerefMut`, so we can access them without any syntactic requirement, for example:

```rs
camera.on_player_move(destination);
```

which is invoking the method:

```rs
impl Camera {
    pub fn on_player_move(&mut self, player_position: Point) { /* ... */ }
}
```

since access to `ResMut` automatically dereferences to `&mut Camera`.

## Query conditionals

By using components directly as query parameters, we've made an implicit assumption: that we want to search entities that include all the components specified, and that we want to retrieve all the components.

A companion concept one may want to express is exclusion: query entities that do _not_ include a component; we use the `With` trait for this:

```rs
pub fn entity_render(query: Query<&PointC, Without<Invisibility>>) {
    for pos: &PointC in query.iter() {
      // ...
    }
}
```

in this case we want all the entities that have a `PointC` component, but not an `Invisibility` one.

There's also something worth mentioning: when querying a single component, we don't use a tuple as `Query` type parameter `Query<(&Player), ...>`, but the type alone; this has an effect on the iteration, which is `pos: &PointC` instead of `(pos: &PointC)`.

The port does not make use of the `Without` conditionals, however, there is a link: the inclusion conditional - `With`.

What's the use, if `Query` implicitly perform inclusion? Convenience, and potentially performance. In some cases, we query for a component, but we don't need to actively use it; by specifying it as `With` parameter, it's not retrieved; an (edited) example is:

```rs
pub fn player_input(mut player_query: Query<&mut PointC, With<Player>>) {
    if let Ok(pos) = player_query.get_single() {
        camera.on_player_move(pos);
    }
}
```

In this case we want to know the position of the player, in order to process it, but since we don't process the Player component, we don't retrieve it.

There's a new API used here: `get_single()`; it is part of a family of APIs that allow access to a set of components in a query, without iteration:

| API                | Checked? (Ret. type) |  Access   |
| ------------------ | :------------------: | :-------: |
| `single()`         |       N (`T`)        | immutable |
| `single_mut()`     |       N (`T`)        |  mutable  |
| `get_single()`     |   Y (`Result<T>`)    | immutable |
| `get_single_mut()` |   Y (`Result<T>`)    |  mutable  |

(note that the return type is simplified)

The APIs are easy to remember:

- the `get_` prefix specifies checked access (returns `None` if an instance is not found, instead of panicking), similarly to `HashMap`;
- the `_mut` suffix specifies mutable access.
