# Icons

There are two icon varieties used in different situations. **_Item icons_** share their ID with entries in the item table and are responsible for the appearance of items in the game world. Menu icons govern the appearance of items in the pause screen and elsewhere in the UI. Their IDs are assigned to various items in the item table. Both types of icons reference the same pool of images.

## Editing existing icons

Icon textures can be altered by simply editing the images in <code>$MOD/image/icon/image</code>. Be careful not to change their size. Remember that icons in pairs need both images to be updated, with the second being a desaturated version of the first (same image, different palette).

More sophisticated edits are possible through editing their animation files. You will almost never want to do this, so I leave it as an exercise to the reader.

## Adding new icons

Strictly speaking, you cannot add new menu or item icons.