# Using Strings

## String Theory

Messages in Paper Mario use a special string format which Star Rod translates into a custom markup language. Non-printing characters are represented by tags enclosed by square brackets with colon-separated parameters. Tags are used for special characters, such as symbols and button icons like <code>[C-LEFT];</code> pauses or delays in printing like <code>[PAUSE];</code> and more complex formatting functions which modify text size/color/etc, e.g. <code>[Size:0A:0A]</code>, or show graphics like Icons and Images. Each string has a unique identifier which includes a 16-bit section ID and a 16-bit message ID. For example, string 001C0002 is message 2 in section 1C. Check the /strings/ folder of your ROM dump for numerous examples. When you compile your mod, Star Rod converts these back into the native Paper Mario string format.

## Star Rod String Markup Guide

### Special Characters

Button Icons    <code>[A] [B] [L] [R] [Z] [C-UP] [C-DOWN] [C-LEFT] [C-RIGHT] [START]</code> <br>
Solid Arrows  <code>[UP] [DOWN] [LEFT] [RIGHT]</code> <br>
Misc Shapes    <code>[NOTE] [HEART] [STAR] [CIRCLE] [CROSS]</code> <br>
Characters for %, [], and {} can be written with an escape character \%, \[, \], \{, and \}.

### Formatting Tags

| Tag          | Description                                                                |
| ------------ | -------------------------------------------------------------------------- |
| [END]        | Terminates the string. All valid strings must end with this tag.           |
| [...]        | Causes the parser to combine the current line with the next one.           |
| [WAIT]       | Waits for the player to press A.                                           |
| [NEXT]       | Scrolls the message box down to the next set of text.                      |
| [STYLE:type] | Sets the style of the message box. Found at the beginning of most strings. |

### Message Box Styles

| Style    | Description                                                                                                                                      |
| -------- | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| RIGHT    | Standard NPC speech bubble connected to the speaker from the right side.                                                                         |
| LEFT     | Standard NPC speech bubble connected to the speaker from the left side.                                                                          |
| CENTER   | Standard NPC speech bubble connected to the speaker from the center.                                                                             |
| TATTLE   | Adaptive-size NPC speech bubble designed for Goombario's map tattles.                                                                            |
| CHOICE   | Expects<br>args: 4 (width, pos x, height, pos y)                                                                                                 |
|          |
| INSPECT  | Grey with a scrolling background pattern. Used when inspecting things.                                                                           |
| SIGN     | Text box looks like a wooden sign.                                                                                                               |
| LAMPPOST | Metallic sign used for the lamp post in Toad Town.<br>args: height of text box                                                                   |
|          |
| POPUP    | Creates a two-line text box in the middle of the screen with an automatic width. Used in scripts only for the “You got Kooper's shell!” message. |
| POSTCARD | Similar to INSPECT. Displays a postcard image on the bottom half of the screen.<br>args: postcard index                                          |
|          |
| UPGRADE  | Used for upgrade blocks. Silent. Grey transparent scrolling BG, rounded frame.<br>args: 4 (width, pos x, height, pos y)                          |
|          |
| NARRATE  | "you got X!", "Y joined your party!"                                                                                                             |
| EPILOGUE | Used on the End of Chapter screens.<br>Silent and centered with no background.                                                                   |
|          |

### Functions

_**TODO**_

### Effect Types

| Effect      | Description                                                                                        |
| ----------- | -------------------------------------------------------------------------------------------------- |
| Rainbow     | Text colors change in a rainbow pattern.                                                           |
| RainbowB    | Same (?) as Rainbow.                                                                               |
| DropShadow  | Adds a drop shadow effect to text.                                                                 |
| Faded       | Text is partially transparent.<br>args: opacity (00 = invisible, FF = opaque)                      |
|             |
| Jitter      | Causes the characters to move around randomly. It looks like the speaker is cold or frightened.    |
| FadedJitter | Partially transparent version of Jitter.<br>args: opacity (00 = invisible, FF = opaque)            |
| Noise       | A noise pattern is overlaid over the text.                                                         |
| FadedNoise  | Partially transparent Noise overlaid on text.<br>args: noise opacity (00 = invisible, FF = opaque) |
|             |
| Wavy        | Characters move in little circles, creating an overall sine wave effect.                           |
| WavyB       | Same (?) as Wavy.                                                                                  |
| SizeJitter  | Character sizes fluctuate randomly.                                                                |
| SizeWave    | Characters move in and out, creating a wave effect of changing size.                               |
| Shrinking   | Large characters are printed that quickly shrink to normal size.                                   |
| Growing     | Small characters are printed that quickly grow to normal size.                                     |

## Modifying/Adding Messages

Any existing string can be modified or overwritten using text files in <code>$mod/strings/</code>. New ones can be added to any section, including a new section (2F) for custom strings. To illustrate, let's modify an existing string: 001C0002 (the tattle for Paragoomba).

Create a text file <code>$mod/strings/example.str</code> (any filename will do) and insert the following:

```text
#string:1C:02 {
[STYLE:RIGHT][...]
Paragoomba tattle!
[WAIT][END] }
```
 
The first line is the string ID with section and message IDs separated by colons. Then the string begins. All valid strings must be terminated by an [END] tag.
 
You can add new strings with whatever ID you like, so long as the section ID is valid. If you don't care what ID the string gets, or prefer not to use hardcoded string IDs, you can give them a name instead. This name can be used in script files and will automatically be replaced by the proper string ID when you compile your mod. The following example is referenced in script files with <code>{String:MyStringName}</code>:

```text
#string:1C:(MyStringName) {
[STYLE:RIGHT][...]
Paragoomba tattle!
[WAIT][END] }
```
 
Another way to use custom strings is to simply embed them in script files. This is great for strings that you don't plan to reuse, or which are not displayed in menus. Here's an example from a script file:

```text
#string $MyExampleString {
[STYLE:RIGHT][...]
This is an example string.
[WAIT][END] }
```

## Examples

### String with Variables

Strings may contain variables to create messages like _“You won X coins!”_ or _“Do you want to cook with Y?”_. Up to four variables can be set, denoted by [Var:0] through [Var:3], which are substituted when the message is displayed. Use these functions to set them in scripts before displaying the message:

```text
Integer values:     Call    SetMessageValue  ( value, variable index )
String values:      Call    SetMessageString ( string, variable index )
```

String variables are usually set with pointers to the string. For example, you can use 8014C294 and 8014C290 to conditionally add “s” to the end of words if they are plural.

### Presenting a Choice

A proper choice dialog has three strings: the question, the options, and the response. Questions should be asked with <code>SayMessage0()</code> and end with {Func_04}, for example:

```text
[STYLE:RIGHT]
Are you ready for a trial?[BR]
[Func_04][END]
```
 
Options must use the following format. The <code>SetCancel</code> call is optional. Choices can contain up to five options. If you need more, use a nested dialog tree.

```text
[STYLE:CHOICE:60:70:80:3E][DelayOff][...]
[Cursor:00][Option:00]Yes
[Cursor:01][Option:01]No
[Cursor:02][Option:02]Tell me more[...]
[Option:FF][DelayOn][SetCancel:01}[...]
[EndChoice:03][END]
```
 
Responses should be shown with <code>SayMessage2()</code> and begin with [NEXT] rather than [STYLE].

```text
[NEXT]All right, maybe next time.[WAIT][END]
```
 
**TIP:** When setting up choice dialog boxes, you may have trouble deciding how big to make them and where to place them on the screen. When displayed, their size/position is stored at the following memory addresses, where you can edit them for a live preview:

```text
801555E2   hpos (0 = left)
801555E4   vpos (0 = top)
8015569C   hsize
8015569E   vsize
```

To center the box, set hpos = 160` - hsize/2.

### Displaying an Image

Strings can display arbitrary images using the Image1 and Image7 functions. Both operate in the same manner, the latter just requires you to specify all the properties (position, etc) of the image in one function, rather than using several functions in conjunction to format it. The images may either be embedded in the map (see mgm_01) or loaded from the asset table (see osr_00 or any map where you get a new partner). In either case, you must tell the message system which image you want to use with <code>set_message_images</code> before displaying the message. The argument to this function is a pointer to an “image table”, which you must set up.

Here is an example:

```text
#new:Function $SetImageList {
    PUSH      RA
    LIA       A0, $ImageList
    JAL       80125B2C
    NOP
    POP       RA
    JR        RA
    ADDIU     V0, R0, 2
} 
   
#new:IntTable $ImageList {
$Image0_Raster $Image0_Palette 00200020 00000002 00000000
$Image1_Raster $Image1_Palette 00200020 00000002 00000000
}

% ~RasterFile etc refer to files in `$mod/res/`
#new:IntTable $Image0_Raster { ~RasterFile:CI-4:example0.png }
#new:IntTable $Image0_Palette { ~PaletteFile:CI-4:example0.png }

#new:IntTable $Image1_Raster { ~RasterFile:CI-4:example1.png }
#new:IntTable $Image1_Palette { ~PaletteFile:CI-4:example1.png }

#new:Script $Conversation {
    ...
    Call     $SetImageList ( )
    Call     SayMessage0 ( ... $ShowImageString )
    ...
}

#string $ShowImageString {
    [Image7:00:00:55:61:01:FF:0F][Func_04][END]
}
```

