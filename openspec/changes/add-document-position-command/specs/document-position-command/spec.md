## ADDED Requirements

### Requirement: Clients can resolve source positions to rendered document positions

Tinymist SHALL expose a public `tinymist.getDocumentPosition` command that resolves a text document position against the latest successful compilation and returns every corresponding rendered document position.

#### Scenario: Source position maps to rendered output

- **WHEN** a client executes `tinymist.getDocumentPosition` with a valid text document URI and LSP position
- **AND** Tinymist has a successful compiled document containing rendered content for that source position
- **THEN** Tinymist returns every matching rendered position
- **AND** each position contains a one-based page number and x/y coordinates in PDF points from the page's top-left

#### Scenario: LSP position encoding is respected

- **WHEN** a source position follows the position encoding negotiated for the LSP session
- **THEN** Tinymist converts that position to Typst's UTF-8 byte offset before resolving rendered positions

#### Scenario: Source position has no rendered output

- **WHEN** the source and successful compiled document are available
- **AND** the source position does not map to rendered content
- **THEN** Tinymist returns an empty array

#### Scenario: Compilation or source is unavailable

- **WHEN** Tinymist cannot resolve the source, source position, or latest successful compiled document
- **THEN** Tinymist returns null

### Requirement: Document position lookup does not require built-in preview

Tinymist SHALL advertise and serve `tinymist.getDocumentPosition` independently of the built-in preview feature.

#### Scenario: External PDF client uses the command

- **WHEN** Tinymist is built without the preview feature
- **THEN** the server advertises `tinymist.getDocumentPosition`
- **AND** a client can resolve positions from Tinymist's latest successful compiled document

### Requirement: Clients can resolve rendered document positions to source positions

Tinymist SHALL expose a public `tinymist.getSourcePosition` command that
resolves a rendered position in the latest successful compilation to a source
document and LSP position.

#### Scenario: Rendered position maps to source

- **WHEN** a client executes `tinymist.getSourcePosition` with a master document URI and valid rendered position
- **AND** the rendered item has a resolvable source span
- **THEN** Tinymist returns standard `TextDocumentPositionParams`
- **AND** the result URI identifies the source containing that span

#### Scenario: Result position encoding is respected

- **WHEN** a rendered position resolves to a source offset
- **THEN** Tinymist converts that offset using the LSP session's negotiated position encoding

#### Scenario: Rendered position has no source mapping

- **WHEN** the compilation is unavailable, the page is invalid, or the rendered position does not resolve to a source location
- **THEN** Tinymist returns null

### Requirement: Source position lookup does not require built-in preview

Tinymist SHALL advertise and serve `tinymist.getSourcePosition` independently
of the built-in preview feature.

#### Scenario: External PDF client performs inverse search

- **WHEN** Tinymist is built without the preview feature
- **THEN** the server advertises `tinymist.getSourcePosition`
- **AND** a client can resolve a rendered position from Tinymist's latest successful compiled document
