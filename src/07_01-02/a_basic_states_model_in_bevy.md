# A basic states model in Bevy

**WARNING**: This chapter has a conceptual error - in order to define state conditional, a standard resource should be used (and the related conditional API), not `NextState`. This is due to very poor documentation on this topic, and it's something that I'm going to fix shortly.

## States modeling problems in Bevy v0.7

Prior to any discussion, it must be pointed out that Bevy's state modeling _as a whole_, as of v0.7, it's broken: using the related APIs to their full extent in a project, will cause breakages (e.g. Bevy will hang); a typical example is that one can't use `FixedTimeStep`s and `State`s at the same time, or, in the case of this project, `State`s with `Stage`s.

Fortunately, a 3rd party developer has created a plugin that solves this problem: [`iyes_loopless`](https://github.com/IyesGames/iyes_loopless). In order to proceed with the next steps, it's therefore necessary to add it to the Cargo configuration.

As of June 2022, it's planned for future Bevy versions to integrate the `iyes_loopless` logic in Bevy.

## First concept: `Stage`s

In the previous chapter, we've seen the source project's states model. In order to translate it in Bevy, we'll use three major concepts; the first one is `Stage`s.

On each frame, Bevy's scheduler goes through a predefined set of so-called `Stage`s, which are essentially group of operations performed on a schedule; they are the appropriate abstraction to use (for reasons we'll see in the following chapters) to model states.

By default, Bevy has several stages, however, we're interested only in one: the `Update` stage; the reason is that during the other stages, other operations are performed behind the scenes, and especially in a simple project, one doesn't want to touch them.

In order to accommodate the states we need, we can add new stages - the idea is that we schedule them to be run in the order we want, but always after `Update` and before the stage following it by default (called `PostUpdate`).