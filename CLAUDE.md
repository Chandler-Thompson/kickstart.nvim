# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Formatting

```sh
stylua --check .   # check formatting
stylua .           # fix formatting
```

Config in `.stylua.toml`: 160-column width, 2-space indent, single-quote preference, no call parentheses.

## Architecture

All Neovim configuration currently lives in `init.lua`. The intent is to evolve this into a modular, system-agnostic distribution — `init.lua` will be broken up over time, with modules landing in `lua/custom/plugins/` and `lua/kickstart/plugins/`. Assume bash and tmux are available; no other system-specific tooling should be required.

**Plugin manager**: lazy.nvim (auto-bootstrapped on first launch). Plugin versions are pinned in `lazy-lock.json`.

**LSP setup flow**: `nvim-lspconfig` depends on `mason.nvim` → `mason-lspconfig.nvim` → `mason-tool-installer.nvim`. Tools listed in `ensure_installed` are auto-installed by Mason. Adding a new LSP means adding it to the `servers` table in the `nvim-lspconfig` config block; Mason handles installation automatically.

**Completion**: `blink.cmp` with `LuaSnip` for snippets. Capabilities from blink are passed to all LSP servers.

**Autoformat**: `conform.nvim` formats on save (`BufWritePre`). Lua uses `stylua`; format-on-save is disabled for C/C++. `<leader>f` formats the current buffer manually.

## Custom plugins and overrides

- `lua/custom/plugins/init.lua` — currently sets a few vim options and returns `{}`. To add plugins, return them from this file and uncomment `{ import = 'custom.plugins' }` in `init.lua`.
- `lua/kickstart/plugins/` — optional built-in extras (debug, autopairs, neo-tree, lint, indent_line, gitsigns). Only `debug` is currently enabled (via `require 'kickstart.plugins.debug'` in `init.lua`).

## Nerd Font

`vim.g.have_nerd_font = false` at the top of `init.lua` controls icon rendering throughout (which-key, mini.statusline, diagnostic signs, lazy.nvim UI). Toggle to `true` when a Nerd Font is active in the terminal.

## Game engine and language support (in progress)

The goal is full LSP coverage for Unity, Java, and Godot development using explicitly specified SDKs and runtimes (not relying on system-installed versions).

**C# / Unity** — `seblyng/roslyn.nvim` provides the Roslyn LSP. The Roslyn server binary must be installed separately (not via Mason); configure the path when adding it. `filewatching = "auto"`, `lock_target = false`. Note: the plugin table in `init.lua` is missing a closing `}`, so all plugins after it are inadvertently nested inside it — exercise care when editing that section.

**Java** — `nvim-java/nvim-java` is a dependency of `nvim-lspconfig`. The `mason-lspconfig` handler for `jdtls` calls `require('java').setup()` before `lspconfig.jdtls.setup`.

**GDScript / Godot** — `gdscript` LSP is configured directly via `lspconfig` (not Mason-managed). `gdtoolkit` is installed via Mason for formatting. Treesitter parser for `gdscript` is included. A `godothost` socket is unconditionally started on launch (the `if projectfile` check is always truthy — the intent was to check file existence but `vim.fn.getcwd() .. '/project.godot'` is always a non-nil string).

## Key bindings reference

Leader key is `<Space>`. Notable custom mappings:
- `<leader>s*` — Telescope search group (`sf` files, `sg` live grep, `sw` word, `sd` diagnostics, `sn` nvim config files, etc.)
- `<leader>f` — format buffer
- `<leader>th` — toggle inlay hints
- `<leader>q` — open diagnostic location list
- `grn` — LSP rename
- `gra` — code action (normal + visual)
- `grr` — references (Telescope)
- `grd` — definition (Telescope)
- `grD` — declaration
- `gri` — implementation (Telescope)
- `grt` — type definition (Telescope)
- `gO` — document symbols (Telescope)
- `gW` — workspace symbols (Telescope)
