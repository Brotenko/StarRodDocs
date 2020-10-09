# Getting Started

## Introduction

Star Rod is a collection of modding tools for Paper Mario 64 (US version 1.0). These tools allow modders to make sweeping changes to the game, from tweaking and rebalancing existing content to adding multiple chapters worth of new content. Paired with a basic text editor, you’ll be able to add or edit maps, battles, enemies, badges, items, strings, and more. The tools contain a map editor, sprite/animation editor, image editor, and assembler.

## Recommended Software

Star Rod effectively dumps many game assets to text files, which you may edit and compile back to the ROM. For this reason, a large part of modding Paper Mario will involve editing text files. Context-sensitive color coding and highlighting help to read these files and prevent basic syntax errors. Use either _Notepad++_ or _VS Code_. _Notepad++_ user-defined language files are provided with Star Rod for script files and string files. A nice plugin for _VS Code_ is available [here](https://marketplace.visualstudio.com/items?itemName=nanaian.vscode-star-rod) (plugin courtesy of [nanalan](https://imalex.xyz/)).

Sprites and textures can be edited in your favorite image editor and converted to the proper format using Star Rod’s image editor. Bear in mind that all sprites and most textures use a color-indexed image format with strict limits for the number of colors per image (16 for sprites, 16 or 256 for textures). Many artists prefer using [Aseprite](https://www.aseprite.org/) for creating pixel art which can be exported as a color-indexed PNG or imported directly into Star Rod’s sprite editor.

Star Rod is written in Java and requires at least Java 1.8 (JRE 8 or newer).

## Creating a Mod

The first time you launch Star Rod, you will be prompted to select a valid Paper Mario US v1.0 rom and a directory for your mod. There are a variety of tools you can use to dump a backup from your own cartridge. Please ensure your cartridge matches the required version. Star Rod will perform validation before proceeding. Your mod directory will contain all the source files you’ll be creating to make your Paper Mario mod.

Once the new project is set up, you must first dump the rom. A folder will be created in the same directory as the ROM with all the modifiable files extracted. This process will take several minutes the first time as assets are repackaged and converted to human-readable formats.

After dumping is complete, use “Copy Assets to Mod” to populate your mod directory with files from the dump directory. The next section will explore the directories in more detail.

Once your edits are ready, use “Compile Mod” to produce a modded Paper Mario rom you can test out in an emulator. To ensure highest compatibility, you should use [Project 64](https://www.pj64-emu.com/) on interpreter mode with the [GLideN64 video plugin](https://github.com/gonetz/GLideN64/releases). The modded ROM will be found in <code>$mod/out</code>. When your mod is done and you’re ready to share it with the world, use “Package Mod” to create a binary diff file which users can combine with their own Paper Mario US v1.0 ROMs. Do not distribute patched ROMs.

## Directories

### Database

### Dump Directory

### Mod Directory

## Notation Used in this Guide
