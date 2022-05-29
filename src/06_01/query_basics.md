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
