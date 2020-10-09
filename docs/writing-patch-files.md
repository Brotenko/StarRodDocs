# Writing Patch Files

## Introduction

Since the memory of the N64 is limited, sections of the ROM containing assets, scripts, and code are loaded into memory as they are needed. For example, data is loaded whenever the player enters a new map, starts a battle, uses a move, and so on. These chunks of data are referred to as **_data sections_**. Each map has its own map data section while battles are grouped together by area (ex: battles from Dry Dry Desert or battles from Koopa Bros Fortress). Each boss usually has their own battle data section. Each player move has its own data section, as do different actions in the overworld.

Star Rod rips these data sections from the ROM and recursively scans through them to detect scripts and data structures. Most data structures have been identified and reverse engineered. Star Rod's automated system will find the vast majority of them and write them to text files: <code>mscr</code> for maps and <code>bscr</code> for battles. Much of the game logic is implemented through a custom event and scripting engine, which is available to us as **_script bytecode_**. This bytecode is sufficiently high-level for us to create new enemies, setup complex map logic, and make new cutscenes without having to write any (or perhaps just a little) assembly.

The scripts are dumped along with all the other data structures in each data section. The dumped data structures can be **_patched_** by editing their text in patch files: <code>mpat</code> for map-related data and <code>bpat</code> for battle-related data. These patches are converted from their various text formats to binary and written over the original structures. If the patched structure is larger than the original, it will be moved to a free location in memory and all pointers to it are automatically updated. You can also declare new data structures in your patch files. Since the text formats for structures in source and patch files are identical, you can copy structures from one source file into a patch file. For example, this can be used to copy enemies from one battle section to another or copy NPCs from one map to another.

The distinction between map and battle data serves two important purposes. First, the set of “library” functions and scripts that are available to call from scripts is different in these two environments. This is because “library” or “common” data is loaded into and out of memory whenever a battle begins or ends. The list of available library functions and scripts can be found in the database folder. Second, different data structures are used in battles and maps. For example, NPCs are only relevant to map scripts and Actors are only relevant to battle scripts. Keeping track of which functions and data structures are available to which scripts is something Star Rod does automatically.

To really understand how these files are structured, you should inspect the dumped sources and study how the game logic is structured. The best way to implement a new feature is often to find a map or battle that already does something similar to what you want and see how the developers did it. For example, the overworld AI for Boo enemies in Pro Mode was created by duplicating and modifying the Ember AI.

## Basic Patch File Syntax

Patches are written like this:

```text
@ $DataTable_80241C00 {
    002A0000 002B0010 002C0015 002D0002
}
```

This tells Star Rod to look for a data structure called <code>DataTable_80241C00</code> and overwrite it with the 16 bytes that appear on the following line. Patches are contained within curly brackets. Let's try something more advanced. We can choose to only overwrite the third value by adding an offset to the start of our patchs:

```text
@ $DataTable_80241C00 { 
    [8] 002C0015 
}
```

By default, each token in a patch is treated as a hexadecimal 32-bit integer. So the value “1” writes “00000001”. This can be changed by adding a suffix identifying the token as a decimal number (`), and as either a 16-bit short (s) or a 8-bit byte (b). You can use the minus sign for negative numbers in any format. Float values will be treated as IEEE floats, but leading decimal places are not allowed. Here are some examples of numbers:

```text
128  = 00000128
100` = 00000064
12b  = 12
12`b = 0C
1FFs = 01FF
-10s = FFF0
3.2  = 404CCCCD
-2.5 = C0200000
.5   = error
```

More sophisticated patches are available for certain data structures. Function patches expect MIPS assembly code, script patches expect lines of Paper Mario's scripting language, strings expect Star Rod's string markup language, and so on. <br/>
You can also use constant values defined locally or globally, enumerated values from the database directory, or special expressions like <code>~Vec3d:LocationMarker</code> that produce specially formatted data or read data from editable map files.

### Patch Syntax Reference

| % comment                                           | Single line comment                                                                                                                                                                                                                            |
| --------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| /%<br>...<br>%/ <img width=200 visibility: hidden/> | Multi-line comment.<br>Can begin or end at any point in a line. Everything between these will be replaced with a single space and will not begin a new line.                                                                                   |
| @ $Pointer {<br>.. }                                | Starts a new patch overwriting the data structure at <code>$Pointer</code>. The <code>$</code> indiciates this is the address of some named data structure.                                                                                    |
| @ $Pointer {<br>[40] ... }                          | Add an offset to the patch location.                                                                                                                                                                                                           |
| @ $Pointer {<br>[40] ...<br> [60] ... }             | Alternate way to create patches with offsets. More than one can be used, but each must start on a new line.                                                                                                                                    |
| @ $Pointer [40]{<br>[40] ...<br> [60] ... }         | Set the _base offset_ for a patch. Other patch offsets for this struct will be relative to this value. Hence, the <code>[10]</code> will begin at <code>$Pointer[50]</code> and the <code>[20]</code> will begin at <code>$Pointer[60]</code>. |
| .Constant                                           | Local/global constants, enum values from in /database/types/, or struct-specific constant values.                                                                                                                                              |
| *ScriptVariable                                     | Special pointers used for both local variables in scripts and global game data. See “Script Variables” for more information.                                                                                                                   |
| ~Expression                                         | Converts the enclosed statement into some complicated binary expression. Used by some scripts to reference data from maps and other source files.                                                                                              |
| #directive                                          | Used for imports, creating new structs, renaming them, and more.                                                                                                                                                                               |

<br>

### Directives

**#new:StructType $StructName** <br>
Creates a new struct and immediately begins reading a patch for it. You can add extra patches to the struct just like any other.

**#import OtherFile.mpat** <br>
Imports all patches from another patch file in the relevant <code>/import/</code> directory. For battle scripts, the <code>/enemy/</code> directory may also be used. You can add patches to imported structs if you need to modify their properties.

**#import MyFile.bpat FileNamespace** <br>
The second example shows an import with a qualifying namespace. Structures imported in this way will have names like: <code>$FileNamespace:PointerName</code>. If the imported file has imports of its own, the resulting import will have a nested namespace, ie: <code>$Outer:Inner:OriginalName</code>.
 
**#define .Name Value** <br>
Creates a new constant. The constant may be referenced anywhere in the current script. Global constants must be defined in either a global patch file or an enum file.
 
**#alias $ComplicatedStructName $SimpleName** <br>
Adds another name to reference an existing struct.
 
**#delete $StructName** <br>
Clears the memory associated with an existing struct. Useful if you need to create more room in an existing map or battle section.
 
**#reserve 80231000 80238000** <br>
Reserves an area of RAM. No new structures will be placed there. You can use it for whatever you like. General Guy uses this area to load each wave of minions. Only one region of RAM may be reserved per patch file.

### Expressions

**~String:*StringName*** <br>
Replaced with the string index assigned to a named custom string (see “Editing Strings”).

**~SizeOf:*Type*** <br>
Replaced with an int token equal to the size of a struct type, e.g. <code>~Sizeof:Actor</code>. Only works for structs that have a fixed size, i.e. not Function, Script, etc.
 
**~Index:*VariableName*** <br>
Replaced by the index of a variable.
Example: <code>~Index:*DojoRank</code> resolves to 1C since *DojoRank = *GameByte[01C].
 
**~Func:*FunctionName*** <br>
Used in function structs to identify calls to known engine functions.
Function names are read from <code>/database/*.lib</code>
 
**~FX:*EffectName*** <br>
Used in script calls to the PlayEffect function, assigning meaningful names to a pair of function arguments. Values are read from <code>/database/types/effects.txt</code>

**~Flags:*FlagType*:*FlagValues*** <br>
**~Flags:*FlagType*:*FlagValues*:*Constant*** <br>
**~Flags:*FlagType*::*Constant*** <br>
Flag type is a name from a _flags_ file in <code>$database/types/</code>. Flag values is a series of flag names separated by |. This value can be blank, but must always be present. Constant is just a constant value to OR with the combination of flag values (intended for undocumented flags). The constant field may be omitted. <br>
Example: <code>~Flags:DamageType:NoContact|Fire</code> for a fire-type move that doesn't count as direct contact.
 
**~Short:*ConstName*** <br>
Resolves a constant and writes it to the patch as a 16-bit short.
 
**~Byte:*ConstName*** <br>
Resolves a constant and writes it to the patch as a 16-bit short.

**~Sprite:*SpriteName*:*AnimationName*** <br>
**~Sprite:*SpriteName*:*PaletteName*:*AnimationName*** <br>
Expands to an NPC sprite animation ID. Names of animations and palettes can be changed using the Sprite Editor.

**~RasterFile:*Format*:*Filename*** <br>
**~PaletteFile:*Format*:*Filename*** <br>
Copies the contents of an image or palette file from the <code>$mod/res</code> directory according to a specified image format, e.g. CI-4, IA-8, etc. The filename is relative to <code>$mod/res</code>.
Supported formats are: I-4, I-8, IA-4, IA-8, IA-16, CI-4, CI-8, RGBA-16, RGBA-32

**~BinaryFile:*Filename*** <br>
Copies the contents of a binary file from the <code>$mod/res</code> directory as a series of byte tokens. The file will not be padded. The filename is relative to <code>$mod/res</code>.

### Map Expressions

Expressions can be used to reference map object data, like ID values or positions from names given in the map editor. When used in a map patch file (.mpat), these expressions may omit the ***MapName*** field when referencing the current map.

**~Model:*MapName*:*ModelName*** <br>
**~ModelShort:*MapName*:*ModelName*** <br>
Replaced by the ID (32 or 16 bit) of the first model in the model tree with a given name based on a breadth first traversal. Avoid issues by using unique model names.
 
**~Collider:*MapName*:*ColliderName*** <br>
**~ColliderShort:*MapName*:*ColliderName*** <br>
**~Zone:*MapName*:*ZoneName*** <br>
**~ZoneShort:*MapName*:*ZoneName*** <br>
Identical to Model and ModelShort.

**~XXX:*MapName*:*MarkerName*** <br>
Marker location expression with valid values for the location XXX are:
```text
Vec2d,  Vec2f     2D planar position
VecXZd, VecXZf    Alternative name for 2D planar position
Vec3d,  Vec3f     3D position
PosXd,  PosXf     X coordinate value
PosYd,  PosYf     Y coordinate value
PosZd,  PosZf     Z coordinate value
Angle,  Angled    Yaw angle
Vec4d,  Vec4f     3D position followed by yaw angle
```
In each case, the expression will be replaced by either floating point (**f**) or 32-bit int (**d**) values for the given location.

**~Path3d:*MapName*:*MarkerName*** <br>
**~Path3f:*MapName*:*MarkerName*** <br>
Replaced by an array of 3D points in the path assigned to the marker.
 
**~PushGrid:*MapName*:*MarkerName*** <br>
Used by CreatePushBlockGrid to specify a grid for blue pushable blocks.

### Constant Offsets 

You can get a relative address within a struct using an offset, which may either be a number or a constant value:

```text
$MyPointer[offset]
$MyPointer[.Constant]
```

Functions allow using one of their internal labels as offsets:

```text
$MyFunction[.o150]
```

Offsets may also be applied to constants, in which case the value of the constants are added together. You may nest offsets in this way:

```text
#define .ConstA 10
#define ConstB 3
.ConstA[.ConstB] % == 13
$MyPointer[.ConstA[.ConstB[2]]] % address of MyPointer + 13
```

### Script Variables 

There are a set of special values which are used in scripts to denote variables. If one of these variables is passed to a function, it will usually read the value from that variable instead of the literal value of the argument. Each set of script variables can be understood as an array with a certain scope and for a certain purpose.

| Name          | Description                             | Maximum | First Value      |
| ------------- | --------------------------------------- | :-----: | ---------------- |
| *GameByte[i]  | Global saved byte.                      |  0x200  | F5DE0180 (-170m) |
| *GameFlag[i]  | Global saved flag.                      |  0x800  | F8405B80 (-130m) |
| *ModByte[i]   | Extra saved byte for mods.              | 0x1000  | ---              |
| *ModFlag[i]   | Extra saved flag for mods.              | 0x8000  | ---              |
| *AreaByte[i]  | Cleared on area change.                 |  0x10   | F70F2E80 (-150m) |
| *AreaFlag[i]  | Cleared on area change.                 |  0x100  | F9718880 (-110m) |
| *MapVar[i]    | Cleared on map change.                  |  0x10   | FD050F80 (-50m)  |
| *MapFlag[i]   | Cleared on map change.                  |  0x60   | FAA2B580 (-90m)  |
| *Fixed[0.3]   | Fixed point real.<br>Precision = 1/1024 | ±19531  | F24A7A80 (-230m) |
| *Array[i]     | From allocated script array.            | varies  | F4ACD480 (-190m) |
| *FlagArray[i] | From allocated script array.            | varies  | F37BA780 (-210m) |
| *Var[i]       | Script variables.                       |  0x10   | FE363C80 (-30m)  |
| *Flag[i]      | Script flags.                           |  0x60   | FBD3E280 (-70m)  |

*Note:* Flags are packed into 32-bit words from LSB to MSB, <br>
1F 1E 1D ... 02 01 00        3F 3E 3D ... 22 21 20        etc.

Just like with other offsets, the array offsets into variables can either be numeric, e.g. <code>*Var[4]</code>, <code>\*GameByte[80]</code>, etc; or constant <code>*GameByte[.SomeConstant]</code>.

##### Map Variables

const pointer 802DA480 holds address of MapFlags (usually 802DBC70, size = 0xC bytes) <br>
const pointer 802DA484 holds address of MapVars (usually 802DBCA8, size = 0x40 bytes) <br>
During map transitions, these are cleared by [802C3278]

##### Area Variables 

800DBF70-800DBF90 <br>
800DBF90-800DBFA0 <br>
The area byte/flag variables are cleared whenever the player leaves the area. They have a limited capacity, so don't overuse them. This is for persistent, but temporary data. For example, an NPC that says different things when you talk to them repeatedly. <br>
During area transitions, these are cleared by [80145390]

##### Utility Functions

Functions read from and write to these variables with a set of helper functions, which you must familiarize yourself with. Each takes an argument pointing to the “script context” -- the data structure holding the state of the script which called the function. For more on that, read section [3.1. Script Bytecode](). These functions can be called with a NULL (0) script context, in which case <code>*Var[x]</code> and <code>*Flag[x]</code> will not be resolvable. Whether variables are treated as integers or floats depends on which set of functions you use.

**get_variable &emsp;&emsp;&emsp;&emsp; (802C7ABC)** <br>
A0     Script context pointer. <br>
A1     “Variable” to resolve. Could be constant or a script variable value like FE363C80. <br>
V0    (return) Integer value of variable.
 
**set_variable &emsp;&emsp;&emsp;&emsp; (802C8098)** <br>
A0     Script context pointer. <br>
A1     “Variable” to resolve. Could be constant or a script variable value like FE363C80. <br>
A2     New value to set. <br>
V0    (return) Previous integer value of variable.
 
**get_float_variable &emsp; (802C842C)** <br>
A0     Script context pointer. <br>
A1     “Variable” to resolve. Could be constant or a script variable value like FE363C80. <br>
F0    (return) Floating-point value of variable.
 
**set_float_variable &emsp; (802C8640)** <br>
A0     Script context pointer. <br>
A1     “Variable” to resolve. Could be constant or a script variable value like FE363C80. <br>
A2     New value to set. <br>
F0    (return) Previous floating-point value of variable.

**get_float &emsp;&emsp;&emsp;&emsp;&emsp;&emsp; (802C4920)** <br>
A0    *Fixed script variable. If out of range, assumes value is a “reasonably-sized” float. <br>
F0    Floating point value.

**get_fixed &emsp;&emsp;&emsp;&emsp;&emsp;&emsp; (802C496C)** <br>
F12    *Fixed script variable <br>
V0    *Fixed representation. Not range-checked! Ensure arguments no larger than ±19531.

**set_global_flag &emsp;&emsp;&emsp; (80145450)** <br>
sets GlobalFlag[i], accepts either indices or encoded values. returns the old value.

**get_global_flag &emsp;&emsp;&emsp; (801454BC)** <br>
returns GlobalFlag[i], accepts either indices or encoded values.

**set_global_byte &emsp;&emsp;&emsp; (80145520)** <br>
sets GlobalByte[i] at 800DBD70, accepts ONLY indices, returns the old value.

**get_global_byte &emsp;&emsp;&emsp; (80145538)** <br>
returns GlobalByte[i] from 800DBD70, accepts ONLY indices.

**set_area_byte &emsp;&emsp;&emsp;&emsp; (80145638)** <br>
sets AreaByte[i] at 800DBF90, accepts ONLY indices, returns the old value.

**get_area_byte &emsp;&emsp;&emsp;&emsp; (80145650)** <br>
returns AreaByte[i] from 800DBF90, accepts ONLY indices.

**set_area_flag &emsp;&emsp;&emsp;&emsp;&emsp; (801455A0)** <br>
sets AreaFlag[i] at 800DBF70, accepts ONLY indices, returns the old value.

**get_area_flag &emsp;&emsp;&emsp;&emsp;&emsp; (801455F0)** <br>
returns AreaFlag[i] from 800DBF70, accepts ONLY indices.

## Global Patches

Patches in <code>$mod/global/patches/</code> are applied directly to the ROM, allowing for global modifications.

### Modifying Existing Data

@Data XXXX <br>
@Function XXXX <br>
@Hook XXXX <br>
@Script:Global XXXX <br>
@Script:Map XXXX <br>
@Script:Battle XXXX <br>

@Fill XXXX YYYY <br>
Fills a region of the ROM between XXX and YYY with whatever pattern is supplied as the patch. The pattern will repeat to fill the region and be truncated if it goes beyond the end.

### Creating New Global Structs

**#new:Data $Name** <br>
Use this to patch arbitrary data to the ROM.

**#new:Function $Name** <br>
Use this to patch a function written in MIPS assembly to the ROM.

**#new:Script:Global $Name** <br>
**#new:Script:Map $Name** <br>
**#new:Script:Battle $Name** <br>
Use these to create new scripts using particular function APIs.

**#reserve XXXX $Name** <br>
Reserves XXXX bytes in RAM and stores the address in the <code>$Name</code> pointer, which can be referenced elsewhere in global patches.

**#export:Function $GlobalFunction** &ensp; Create a struct as global. <br>
**#export .NewConstName XXXX** &emsp;&emsp; Create a constant as global. <br>
**#export $GlobalPointer** &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp; Make an existing struct global. <br>
**#export .ExistingConstName** &emsp;&emsp;&emsp;&emsp; Make an existing constant global. <br>

Export makes things global. It works with both pointers and constants. You can also use <code>#export:</code> instead of <code>#new:</code> as a shorthand for creating global structs.

## Hint Files

The automatic recursive dumping system is sometimes unable to find all data structures in a data section, and generally unable to give descriptive names to the structures it does find. Fortunately, a hint system allows mod authors to supply hints to the dumping process. Hint files are located in <code>/database/hints/</code> with names that match the source files they apply to. You’ll find a large variety of default hints there. Within each file, the following hints are supported:

| Command | Description                                                                                                         |
| ------- | ------------------------------------------------------------------------------------------------------------------- |
| add     | Identifies a new data structure of a given type. <br> ex: <code>add      80240210   Function</code>                 |
| name    | Assign a unique name to a data structure. <br> ex: <code>name     80240210   Function_FadeScreenToBlack</code>      |
| size    | Forces a certain size for a data structure. Rarely needed. <br> ex: <code>size     80240210   800</code>            |
| newline | Sets how many 32-bit words should be printed on each line. Default = 8. <br> ex: <code>newline  80240210   6</code> |
|         |

<br>