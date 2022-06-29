# Conclusion

In this mini-book, we've explored how Bevy's ECS applies to a full-fledged, nontrivial, game.

Before reading Hands-on Rust, ECSs were a bit of mystery to me. After the book, basing a game on an ECS felt natural and advantageous. Nowadays, I think that a game does not need to be large in order to - overall - profit from an ECS.

I'm impressed by Bevy's ECS interface, as the querying system is very coincise, and makes for readable code. On the other hand, Bevy has a fundamental, unaddressed, problem in the state managament, however, the `iyes_loopless` solves it cleanly, and one can produce solid work with this engine.

Happy experimentation 😄
