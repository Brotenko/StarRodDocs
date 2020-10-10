# Editing Sprites

## Adding a new NPC sprite

To add a new NPC sprite, duplicate one of the folders in $MOD/sprite/npc/src/, give it a new name, and add the new sprite to $MOD/sprite/SpriteTable.xml. The new sprite will now appear in the sprite editor and will be built when you compile your mod.

## Palettes

Sprites only used in the world may have whatever palettes you like, but sprites which are used in battle are expected to have a certain number of palettes in a particular order to properly display status ailments. The expected order is:

**Partner Actors:**

|   x   | Status        | Description |
| :---: | ------------- | ----------- |
|   0   | Normal        | Unchanged   |
|   1   | Poisoned      | Green tint  |
|   2   | Turn Finished | Darker tint |
|   3   | Shocked       | Yellow tint |
|   4   | Burned        | Blackened   |

<br>

**Enemy Actor:**

|   x   | Status        | Description |
| :---: | ------------- | ----------- |
|   0   | Normal        | Unchanged   |
|   1   | Poisoned      | Green tint  |
|   2   | Turn Finished | Darker tint |
|   3   | Shocked       | Yellow tint |
|   4   | Burned        | Blackened   |

<br>

Enemies with different color-variants (palette-swaps) have a separate set of palettes for each color variant (see sprite 31). In the sprite editor, the number of color variants is controlled by the “Groups” field of the “Spritesheet” tab. The palettes are expected in the following order:

```text
Normal 1, Normal 2, …, Normal N
Poisoned 1, Poisoned 2, …, Poisoned N
Dizzy 1, Dizzy 2, …, Dizzy N
Shocked 1, Shocked 2, … Shocked N
Burned (only one)
```

Additional palettes for certain “accessories” appear after these (see sprite 68).
