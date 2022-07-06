# Introduction to steps 07.01-02

**WARNING**: This chapter has a conceptual error - in order to define state conditional, a standard resource should be used (and the related conditional API), not `NextState`. This is due to very poor documentation on this topic, and it's something that I'm going to fix shortly.

This chapter covers both steps 07.01 and 07.02; we're going to explore `State`s, `SystemSet`s and `Stage`s (and the `iyes_loopless` crate).

The Bevy concepts described here are the foundation for structuring games, and this step is the one that departs the most from the source project.

The directory names are `07_TurnBasedGames_01_wandering` and `07_TurnBasedGames_02_turnbased`.
