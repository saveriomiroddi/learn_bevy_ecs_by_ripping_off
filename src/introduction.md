# Introduction

This guide introduces developers to Bevy's ECS component, by porting the game written for [Hands-on Rust: Effective Learning through 2D Game Development and Play](https://pragprog.com/titles/hwrust/hands-on-rust) to Bevy's ECS.

## How to approach this book

The book, and the [source project](https://github.com/thebracket/HandsOnRust), are structured as a series of incremental steps, leading to a full game.

In the [code repository](https://github.com/64kramsystem/learn_bevy_ecs_by_ripping_off-code), I've made minor cleanups and restructurings to the source project; the locations of the workspaces are:

- [port](https://github.com/64kramsystem/learn_bevy_ecs_by_ripping_off-code/tree/master/port)
- [source](https://github.com/64kramsystem/learn_bevy_ecs_by_ripping_off-code/tree/master/source)

The steps have been numbered (and have matching names in the source/port), in order to facilitate sequential examination and comparison; each step is an independent workspace.

This book presents multiple chapters for each step, explaining the concepts involved. When reading a chapter, one can compare the source step (workspace) with the ported one, and/or with the previous step in the respective project.

The starting chapter is 06.01, where an ECS is introduced in the design. Steps that don't introduce any new ECS concept, or don't significantly extend an introduced one, are skipped.

It's technically possible to read this book standalone, without the source one, but for newcomers to ECS, it's not advised, as in this context I'm not explaining the ECS concepts (only their Bevy API implementation).

## References

The Bevy used version is 0.7.0; all the API links refer to the tag `v0.7.0` in [Bevy's repository](https://github.com/bevyengine/bevy).

## Port style

Since comparison is a foundation of this book, I've maintained high and low-level structures as similar as I could; the most radical divergence is due to Bevy's state management. There are still a few differences, but they're minor.

## Thanks

This project has been kindly sponsored by [Ticketsolve](https://www.ticketsolve.com); naturally, it wouldn't have been possible without the great work of [Herbert Wolverson](https://twitter.com/herberticus), and the [Bevy](https://bevyengine.org) community.
