## Why

Editor integrations can ask Tinymist's built-in web preview to follow the source cursor, but they cannot retrieve the corresponding rendered page coordinates. Clients such as Emacs therefore cannot implement forward search into an external PDF viewer while reusing Tinymist's compiled document and Typst's source-to-output mapping.

## What Changes

- Add a public `tinymist.getDocumentPosition` LSP command.
- Accept a standard text document URI and negotiated LSP position.
- Return every corresponding rendered page position from the latest successful compilation.
- Document the command's parameters, coordinate system, and unavailable/unmapped results.

## Capabilities

### New Capabilities

- `document-position-command`: Let editor clients resolve a Typst source position to one or more rendered page coordinates through `workspace/executeCommand`.

### Modified Capabilities

None.

## Impact

- `tinymist-query` gains a semantic request around the existing source-to-document jump implementation.
- `tinymist` gains and advertises a public execute command independent of the built-in preview feature.
- The command-system documentation gains the public wire contract.
