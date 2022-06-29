# Exercise for the reader

I think that fully practicing the concepts studied on a book, brings understanding to another level. This book presents a fantastic opportunity!

## Exercise

The source project has an uneven design. While the game itself is designed very neatly, its initialization is inconsistent with the rest, as it has a procedural design:

- it access the `World` directly, and doesn't use systems;
- it uses (local) variables, and reinstantiates them, instead of using resources.

The exercise is to redesign the game setup, so that it fully applies the ECS design, by using systems and resources. If possible, don't reinitialize resources - reuse them!

In my opinion, performing this exercise will give the reader a more solid understanding of how to work with ECSs and design around them!
