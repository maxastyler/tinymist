## Why

Editor integrations can ask Tinymist's built-in web preview to follow the source cursor, but they cannot retrieve the corresponding rendered page coordinates. Clients such as Emacs therefore cannot implement forward search into an external PDF viewer while reusing Tinymist's compiled document and Typst's source-to-output mapping.

## What Changes

- Add a public `tinymist.getDocumentPosition` LSP command.
- Add a public `tinymist.getSourcePosition` LSP command as its inverse.
- Accept a standard text document URI and negotiated LSP position.
- Return every corresponding rendered page position from the latest successful compilation.
- Accept a master document URI and rendered page position, and return the
  corresponding source document URI and negotiated LSP position.
- Document the command's parameters, coordinate system, and unavailable/unmapped results.

## Capabilities

### New Capabilities

- `document-position-command`: Let editor clients resolve positions in both
  directions between Typst sources and rendered page coordinates through
  `workspace/executeCommand`.

### Modified Capabilities

None.

## Impact

- `tinymist-query` gains a semantic request around the existing source-to-document jump implementation.
- `tinymist-query` exposes the existing document-to-source jump implementation
  through the semantic-query path.
- `tinymist` gains and advertises a public execute command independent of the built-in preview feature.
- The command-system documentation gains the public wire contract.
