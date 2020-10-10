# Writing Scripts

## Script Bytecode

Paper Mario uses a custom scripting language to implement most of its high-level game logic. These scripts are compiled into bytecode that is read by an interpreter at runtime.  They often call functions to execute particular tasks like _PlaySound()_ or _RemoveActor()_. This is quite fortunate for the modding community, as it allows for easy higher-level modifications. It is easy to add, for example, new enemy attacks.

Internally, each script has a **_script context_**, a data structure that manages the state of the script, stores its variable values, etc. A new script context is created every time a new script is run and the context is deleted when the script is finished executing.

## Inline Script Expressions

The value of script variables can be assigned with mathematical expressions like this:

```text
Set   *Var[0] = *Var[2] + *Var[3] - 10`
```

These are automatically compiled into basic bytecode commands and optimized for space. You should use these wherever possible to maintain reliability. Use Set and SetF to distinguish between values which should be handled as floats or integers.