## Overview

The existing `tinymist_query::jump_from_cursor` function already maps a UTF-8 byte offset in a Typst source to all matching `PagedPosition` values in a compiled document. The new command exposes that operation through the normal compiler-query path so it uses Tinymist's selected project, latest successful document, in-memory source contents, and negotiated LSP position encoding.

## Command Contract

Clients call `workspace/executeCommand` with command `tinymist.getDocumentPosition` and one `TextDocumentPositionParams` argument:

```json
{
  "textDocument": { "uri": "file:///project/main.typ" },
  "position": { "line": 10, "character": 7 }
}
```

The result is either `null` or an array of objects with `page_no`, `x`, and `y`. Pages are one-based. Coordinates are measured in PDF points from the top-left of the page.

- `null` means the source, source position, or latest successful compiled document is unavailable.
- `[]` means the request was resolved against a compiled document but Typst found no rendered location.
- Multiple positions are retained because source content can render more than once.

## Query Integration

Add a `DocumentPositionRequest` semantic query with the source path and LSP position. It resolves the source, converts the LSP position through the negotiated encoding, obtains the successful document, invokes `jump_from_cursor`, and converts each result to the existing serializable `tinymist_world::debug_loc::DocumentPosition`.

Add matching variants to `CompilerQueryRequest` and `CompilerQueryResponse`. Route the query as `PinnedFirst` and associate it with the requested source path, consistent with other document-specific semantic queries.

## LSP Integration

The command handler parses a standard `TextDocumentPositionParams`, converts the URI through Tinymist's existing path helper, and schedules the semantic query. Register it in the core command table rather than the preview-feature command table so external PDF integrations do not require the built-in preview server.

## Inverse Command

Clients call `workspace/executeCommand` with command
`tinymist.getSourcePosition` and one argument containing the master document
identifier and a rendered document position:

```json
{
  "textDocument": { "uri": "file:///project/main.typ" },
  "position": { "page_no": 1, "x": 70.866, "y": 78.104 }
}
```

The master document selects the compilation context. The result is either
`null` or a standard `TextDocumentPositionParams` whose URI may identify any
source file used by that compilation:

```json
{
  "textDocument": { "uri": "file:///project/included.typ" },
  "position": { "line": 10, "character": 7 }
}
```

Pages and coordinates use the same one-based, top-left PDF-point convention as
`tinymist.getDocumentPosition`. Result lines are zero-based and characters use
the LSP session's negotiated position encoding. `null` covers unavailable
compilations, invalid pages, unmapped clicks, links, and source locations that
cannot be resolved to a URI.

Add a `SourcePositionRequest` semantic query with the master path and
`DocumentPosition`. It obtains the successful paged document, selects the
requested page, invokes `jump_from_click`, resolves the returned source span and
offset through the active world, and converts the byte offset to the negotiated
LSP position. Offsets at the end of a span are retained.

## Compatibility and Consistency

The commands are additive. Their positions describe Tinymist's latest successful compilation. Clients are responsible for ensuring any displayed PDF was produced from the same document revision and compiler settings.
