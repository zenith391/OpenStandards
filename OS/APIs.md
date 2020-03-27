## Standard OS APIs for interoperability

This document outlines APIs that OpenComputers operating systems should attempt to implement.

### `filesystem`

As detailed on the [OpenComputers Wiki](https://ocdoc.cil.li/api:filesystem).

### `io`

Operating systems should attempt to implement as much functionality from the Lua-standard `io` as possible (see the [Lua manual entry](https://lua.org/manual/5.3/manual.html#6.8) for details).

### `package`

OSes should implement at least the following subset of the `package` library:

- `package.path: string`
  A string containing the path(s) that `require` will search for modules.

- `package.loaded: table`
  A table of all currently loaded APIs.

- `package.searchpath(name: string, path: string[, sep: string[, rep: string]]): string or boolean, error`
  Searches the provided path for `name`, replacing all instances of `sep` with `rep`. `sep` defaults to `.`, and `rep` to `/`.

- `require(modname: string): table`
  Searches the package path for a module. Throws an error if the module is not found, or returns the API if it is.

### `term`

An OS' `term` API should provide at a minimum the following:

- `term.setCursor(x: number, y: number)`
  Sets the term cursor position to `(x, y)`. Returns `nil` and an error message on failure.

- `term.getCursor(): number, number`
  Returns the current term cursor position.

- `term.setCursorBlink(blink: boolean)`
  Controls whether the cursor should blink or, in some cases, whether it should be shown.

- `term.getCursorBlink(): boolean`
  Returns whether the cursor is currently blinking or, in some cases, whether it is shown.

- `term.write(text: string[, wrapMode: string]): number`
  Writes `text` to the screen. `wrapMode` can be `"precise"` or `"word"`, and specifies how words should be wrapped.

- `term.read([history: table[, replace: string[, default: string]]])`
  Reads text input from the user.
