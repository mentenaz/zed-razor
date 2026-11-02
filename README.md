# zed-razor

Razor (`.razor`, `.cshtml`) support for the [Zed editor](https://zed.dev) — syntax highlighting, IntelliSense, diagnostics, go-to-definition, and more for ASP.NET Blazor/Razor Pages.

## About this fork

This is a fork of [dsbraz/zed-razor](https://github.com/dsbraz/zed-razor), maintained by
[Mentenaz](https://github.com/mentenaz) for [Mentenaz Forge Super](https://forge.mentenaz-server.com)
and published to its extension registry. It differs from upstream in these ways:

- The language server is started with the arguments current Roslyn builds accept
  (`--autoLoadProjects`, and no `--razorSourceGenerator` or `--razorDesignTimePath`).
- Small highlighting and C# injection fixes in `languages/razor/`, and JavaScript injection for
  inline event handlers (`onclick="showTab('one')"`) and, combined into one program, for
  `<script>` blocks that contain Razor values.
- The grammar comes from [mentenaz/tree-sitter-razor](https://github.com/mentenaz/tree-sitter-razor),
  a fork that reads braces as text inside HTML elements (so `<style>`, `<script>` and JSON parse),
  allows the other kind of quote inside a quoted attribute value (so `onclick="show('x')"`
  parses), and supports `@@` and `@:`. It is pinned to an exact commit instead of `main`, so builds are
  reproducible.

Original work by Daniel Braz, under the MIT licence (see `LICENSE`).

## Future improvements

Ideas that are not built yet, roughly in the order they would be done:

1. **`<script>` and `<style>` as raw-text elements** (in the grammar, `mentenaz/tree-sitter-razor`).
   Today a `<` inside a script (`for (var i = 0; i < 10; i++)`) breaks the parse, and markup-looking
   text inside a string (`"<div></div>"`) is read as real elements. Inside these two elements only the
   closing tag should end the text, while `@` expressions are still parsed.
2. **Hover and completion for JavaScript and CSS inside `<script>` and `<style>`.** Injections only
   colour the code; the editor picks language servers per file, so no JavaScript server sees these
   regions. A small second language server for Razor files could answer only when the cursor is inside
   a script or style block: it would blank out the rest of the file with spaces (so every position stays
   the same and nothing needs translating), hand that virtual document to a TypeScript or CSS server, and
   return its answers. Without type definitions, `$` would be unknown, so errors from those regions would
   be turned off and only hover and completion offered.
3. Parse expressions inside an `@:` line instead of keeping the whole line as text.
4. Read `user@example.com` as text, as Razor does, instead of as an expression.
5. C# injection for Blazor directive attributes such as `@onclick="Handler"`.

## Features

- **Syntax Highlighting** — Razor directives (`@page`, `@model`, `@inject`, `@code`, etc.), control structures (`@if`, `@foreach`, `@switch`), explicit/implicit expressions, HTML tags and Blazor components (PascalCase)
- **C# Language Injection** — Full C# highlighting inside `@code`, `@functions`, `@(...)` expressions and control structures
- **Bracket Matching** — `{}`, `()`, `[]`, `<>`, `<!-- -->` and `@* *@`
- **Auto-Indentation** — Smart indentation for HTML elements and C# blocks
- **Code Outline** — Navigate directives, `@code` blocks, sections and components
- **Language Server (LSP)** — Completions, diagnostics, hover, go-to-definition, find references, rename, formatting, inlay hints and code actions via Roslyn + Razor co-hosting (downloaded automatically)

## Repository Structure

```
zed-razor/
├── src/                # Rust LSP bridge (download + launch Roslyn)
├── languages/razor/    # Language queries (.scm) and config
├── lsp-razor/          # Language server documentation and configuration reference
└── extension.toml      # Extension manifest
```

The Tree-sitter grammar lives in a separate repository: [dsbraz/tree-sitter-razor](https://github.com/dsbraz/tree-sitter-razor).

## Installation

### From Zed Extension Registry (recommended)

Search for **Razor** in Zed's extension browser (`Cmd+Shift+X`).

### As a Dev Extension

1. Clone the repository:
   ```bash
   git clone https://github.com/dsbraz/zed-razor.git
   ```

2. Open the command palette (`Cmd+Shift+P` / `Ctrl+Shift+P`)

3. Run `zed: install dev extension`

4. Select the `zed-razor/` directory

Zed will compile the Tree-sitter grammar to WASM and start downloading the Roslyn language server (~150 MB) in the background on first use.

## Language Server

The extension automatically downloads the [Roslyn Language Server](https://github.com/dotnet/roslyn) with Razor co-hosting support from [Crashdummyy/roslynLanguageServer](https://github.com/Crashdummyy/roslynLanguageServer). No manual installation required.

Supported platforms: macOS (ARM64, x64), Linux (x64, ARM64), Windows (x64, ARM64).

For manual configuration options, see [`lsp-razor/README.md`](lsp-razor/README.md).

## Configuration

Add to your Zed `settings.json` to customize language behavior:

```json
{
  "languages": {
    "Razor": {
      "tab_size": 4,
      "hard_tabs": false
    }
  },
  "lsp": {
    "roslyn-razor": {
      "binary": {
        "path": "/custom/path/to/Microsoft.CodeAnalysis.LanguageServer"
      }
    }
  }
}
```

## Contributing

PRs are welcome. Contributions can target:

- **LSP bridge** — `src/razor.rs` (download and launch logic)
- **Language queries** — `languages/razor/*.scm` (highlighting, injections, indentation)
- **Tree-sitter grammar** — [dsbraz/tree-sitter-razor](https://github.com/dsbraz/tree-sitter-razor)
- **LSP docs** — `lsp-razor/README.md`

## License

MIT
