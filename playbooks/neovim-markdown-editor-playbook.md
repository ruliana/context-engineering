# Neovim Markdown Editor Context

## Key Concepts

- **Markdown Preview**: Live HTML preview of markdown files in browser or floating window
- **Tree-sitter**: Modern syntax highlighting and parsing engine for markdown
- **LSP (Language Server Protocol)**: Provides diagnostics, formatting, and completion for markdown
- **Concealing**: Hide or stylize markdown syntax characters for cleaner editing view
- **Table Mode**: Automatic table formatting and alignment in markdown
- **Spell Checking**: Built-in and enhanced spell checking for prose writing
- **Zen Mode**: Distraction-free writing environment
- **TOC (Table of Contents)**: Automatic generation and updating of document outlines
- **Link Management**: Auto-completion and validation of markdown links
- **Export Pipeline**: Converting markdown to PDF, HTML, or other formats

## Common Patterns

### Basic Markdown Setup
```lua
-- In ~/.config/nvim/lua/plugins/markdown.lua
return {
  {
    "nvim-treesitter/nvim-treesitter",
    event = { "BufReadPre", "BufNewFile" },
    build = ":TSUpdate",
    opts = {
      ensure_installed = { "markdown", "markdown_inline" },
      highlight = { enable = true },
      indent = { enable = true },
    }
  },
  {
    "iamcco/markdown-preview.nvim",
    cmd = { "MarkdownPreviewToggle", "MarkdownPreview", "MarkdownPreviewStop" },
    build = "cd app && npm install",
    init = function()
      vim.g.mkdp_filetypes = { "markdown" }
    end,
    ft = { "markdown" },
  }
}
```
**When to use**: Essential setup for any markdown editing workflow
**Expected output**: Syntax highlighting active, `:MarkdownPreview` opens browser preview

### Enhanced Writing Experience
```lua
{
  "folke/zen-mode.nvim",
  cmd = "ZenMode",
  opts = {
    window = {
      backdrop = 0.95,
      width = 80,
      height = 1,
      options = {
        signcolumn = "no",
        number = false,
        relativenumber = false,
        cursorline = false,
        cursorcolumn = false,
        foldcolumn = "0",
        list = false,
      },
    },
    plugins = {
      options = {
        enabled = true,
        ruler = false,
        showcmd = false,
      },
      twilight = { enabled = true },
      gitsigns = { enabled = false },
    },
  },
  keys = {
    { "<leader>zz", "<cmd>ZenMode<cr>", desc = "Zen Mode" },
  },
}
```
**When to use**: Distraction-free writing sessions
**Expected output**: Centered text with minimal UI elements

### Table Management
```lua
{
  "dhruvasagar/vim-table-mode",
  ft = "markdown",
  config = function()
    vim.g.table_mode_corner = "|"
    vim.g.table_mode_corner_corner = "|"
    vim.g.table_mode_header_fillchar = "-"
  end,
  keys = {
    { "<leader>tm", "<cmd>TableModeToggle<cr>", desc = "Toggle Table Mode" },
    { "<leader>tr", "<cmd>TableModeRealign<cr>", desc = "Realign Table" },
  },
}
```
**When to use**: Creating and editing markdown tables
**Expected output**: Automatic table formatting on pipe character input

### Advanced Link Management
```lua
{
  "jakewvincent/mkdnflow.nvim",
  ft = "markdown",
  opts = {
    modules = {
      bib = true,
      buffers = true,
      conceal = true,
      cursor = true,
      folds = true,
      links = true,
      lists = true,
      maps = true,
      paths = true,
      tables = true,
      yaml = false
    },
    filetypes = {md = true, rmd = true, markdown = true},
    create_dirs = true,
    perspective = {
      priority = 'first',
      fallback = 'current',
      root_tell = false,
      nvim_wd_heel = false,
      update = false
    },
    wrap = false,
    bib = {
      default_path = nil,
      find_in_root = true
    },
    silent = false,
    links = {
      style = 'markdown',
      name_is_source = false,
      conceal = false,
      context = 0,
      implicit_extension = nil,
      transform_implicit = false,
      transform_explicit = function(text)
        text = text:gsub(" ", "-")
        text = text:lower()
        text = os.date('%Y-%m-%d_')..text
        return(text)
      end
    },
    new_file_template = {
      use_template = false,
      placeholders = {
        before = {
          title = "link_title",
          date = "os_date"
        },
        after = {}
      },
      template = "# {{ title }}"
    },
    to_do = {
      symbols = {' ', '-', 'X'},
      update_parents = true,
      not_started = ' ',
      in_progress = '-',
      complete = 'X'
    },
    tables = {
      trim_whitespace = true,
      format_on_move = true,
      auto_extend_rows = false,
      auto_extend_cols = false
    },
    yaml = {
      bib = { override = false }
    },
    mappings = {
      MkdnEnter = {{'n', 'v'}, '<CR>'},
      MkdnTab = false,
      MkdnSTab = false,
      MkdnNextLink = {'n', '<Tab>'},
      MkdnPrevLink = {'n', '<S-Tab>'},
      MkdnNextHeading = {'n', ']]'},
      MkdnPrevHeading = {'n', '[['},
      MkdnGoBack = {'n', '<BS>'},
      MkdnGoForward = {'n', '<Del>'},
      MkdnCreateLink = false,
      MkdnCreateLinkFromClipboard = {{'n', 'v'}, '<leader>p'},
      MkdnFollowLink = false,
      MkdnDestroyLink = {'n', '<M-CR>'},
      MkdnTagSpan = {'v', '<M-CR>'},
      MkdnMoveSource = {'n', '<F2>'},
      MkdnYankAnchorLink = {'n', 'yaa'},
      MkdnYankFileAnchorLink = {'n', 'yfa'},
      MkdnIncreaseHeading = {'n', '+'},
      MkdnDecreaseHeading = {'n', '-'},
      MkdnToggleToDo = {{'n', 'v'}, '<C-Space>'},
      MkdnNewListItem = false,
      MkdnNewListItemBelowInsert = {'n', 'o'},
      MkdnNewListItemAboveInsert = {'n', 'O'},
      MkdnExtendList = false,
      MkdnUpdateNumbering = {'n', '<leader>nn'},
      MkdnTableNextCell = {'i', '<Tab>'},
      MkdnTablePrevCell = {'i', '<S-Tab>'},
      MkdnTableNextRow = false,
      MkdnTablePrevRow = {'i', '<M-CR>'},
      MkdnTableNewRowBelow = {'n', '<leader>ir'},
      MkdnTableNewRowAbove = {'n', '<leader>iR'},
      MkdnTableNewColAfter = {'n', '<leader>ic'},
      MkdnTableNewColBefore = {'n', '<leader>iC'},
      MkdnFoldSection = {'n', '<leader>f'},
      MkdnUnfoldSection = {'n', '<leader>F'}
    }
  },
}
```
**When to use**: Advanced markdown document management with linking
**Expected output**: Follow links with Enter, create new notes, table navigation

### Grammar and Spell Checking
```lua
{
  "barreiroleo/ltex-extra.nvim",
  ft = { "markdown", "tex" },
  dependencies = { "neovim/nvim-lspconfig" },
  opts = {
    load_langs = { "en-US" },
    init_check = true,
    path = nil,
    log_level = "none",
  },
  config = function(_, opts)
    require("ltex_extra").setup(opts)
  end,
}
```
**When to use**: Professional writing requiring grammar checking
**Expected output**: Grammar errors highlighted, suggestions in LSP diagnostics

## Implementation Details

### Complete Markdown Editor Setup

1. **Create plugin configuration directory**:
```bash
mkdir -p ~/.config/nvim/lua/plugins
touch ~/.config/nvim/lua/plugins/markdown.lua
```

2. **Basic markdown plugin configuration**:
```lua
-- ~/.config/nvim/lua/plugins/markdown.lua
return {
  -- Syntax highlighting
  {
    "nvim-treesitter/nvim-treesitter",
    event = { "BufReadPre", "BufNewFile" },
    build = ":TSUpdate",
    opts = {
      ensure_installed = { "markdown", "markdown_inline", "html" },
      highlight = { enable = true },
      indent = { enable = true },
    }
  },
  
  -- Live preview
  {
    "iamcco/markdown-preview.nvim",
    cmd = { "MarkdownPreviewToggle", "MarkdownPreview", "MarkdownPreviewStop" },
    build = "cd app && npm install",
    init = function()
      vim.g.mkdp_filetypes = { "markdown" }
      vim.g.mkdp_auto_start = 0
      vim.g.mkdp_auto_close = 1
      vim.g.mkdp_refresh_slow = 0
      vim.g.mkdp_command_for_global = 0
      vim.g.mkdp_open_to_the_world = 0
      vim.g.mkdp_open_ip = ""
      vim.g.mkdp_port = ""
      vim.g.mkdp_theme = "dark"
    end,
    ft = { "markdown" },
  },
  
  -- Table editing
  {
    "dhruvasagar/vim-table-mode",
    ft = "markdown",
    config = function()
      vim.g.table_mode_corner = "|"
      vim.g.table_mode_corner_corner = "|"
      vim.g.table_mode_header_fillchar = "-"
    end,
  },
  
  -- Zen mode for writing
  {
    "folke/zen-mode.nvim",
    cmd = "ZenMode",
    opts = {
      window = {
        backdrop = 0.95,
        width = 80,
        height = 1,
      },
    },
  },
  
  -- Enhanced markdown features
  {
    "jakewvincent/mkdnflow.nvim",
    ft = "markdown",
    opts = {
      modules = {
        bib = true,
        buffers = true,
        conceal = true,
        cursor = true,
        folds = true,  
        links = true,
        lists = true,
        maps = true,
        paths = true,
        tables = true,
      },
    },
  }
}
```

3. **Markdown-specific Neovim settings**:
```lua
-- ~/.config/nvim/lua/config/markdown.lua
vim.api.nvim_create_autocmd("FileType", {
  pattern = "markdown",
  callback = function()
    vim.opt_local.wrap = true
    vim.opt_local.breakindent = true
    vim.opt_local.linebreak = true
    vim.opt_local.spell = true
    vim.opt_local.spelllang = "en_us"
    vim.opt_local.conceallevel = 2
    vim.opt_local.concealcursor = "n"
  end,
})
```

4. **Key mappings for markdown**:
```lua
-- ~/.config/nvim/lua/config/keymaps.lua
local function map(mode, lhs, rhs, opts)
  vim.keymap.set(mode, lhs, rhs, opts or {})
end

-- Markdown specific mappings
map("n", "<leader>mp", "<cmd>MarkdownPreview<cr>", { desc = "Markdown Preview" })
map("n", "<leader>ms", "<cmd>MarkdownPreviewStop<cr>", { desc = "Stop Markdown Preview" })
map("n", "<leader>tm", "<cmd>TableModeToggle<cr>", { desc = "Toggle Table Mode" })
map("n", "<leader>zz", "<cmd>ZenMode<cr>", { desc = "Toggle Zen Mode" })
```

**Validation**: `:Lazy` shows all plugins installed, `:MarkdownPreview` opens browser
**Common errors**: `npm not found` = install Node.js for markdown-preview.nvim

### Language Server Setup for Markdown

1. **Install marksman LSP**:
```bash
# Via Mason
:MasonInstall marksman
```

2. **Configure marksman LSP**:
```lua  
-- ~/.config/nvim/lua/plugins/lsp.lua
{
  "neovim/nvim-lspconfig",
  opts = {
    servers = {
      marksman = {
        filetypes = { "markdown" },
        root_dir = function(fname)
          return require("lspconfig.util").root_pattern(".git", ".marksman.toml")(fname)
            or require("lspconfig.util").path.dirname(fname)
        end,
      },
    },
  },
}
```

**Validation**: `:LspInfo` in markdown file shows marksman attached
**Common errors**: `marksman not found` = ensure Mason installed it correctly

### Export Configuration

1. **Pandoc integration**:
```lua
{
  "aspeddro/pandoc.nvim",
  ft = "markdown",
  opts = {
    default = {
      args = {
        {"-f", "markdown"},
        {"-t", "html"},
        {"--standalone"}
      }
    },
    commands = {
      {
        name = "pdf",
        args = {
          {"-f", "markdown"},
          {"-t", "pdf"},
          {"--pdf-engine", "xelatex"}
        }
      }
    }
  }
}
```

**Validation**: `:Pandoc pdf` creates PDF file in same directory
**Common errors**: `xelatex not found` = install LaTeX distribution

## Validation Methods

### Feature Testing Checklist
- **Syntax highlighting**: Open `.md` file, verify colors and formatting
- **Live preview**: `:MarkdownPreview` opens browser with rendered content
- **Table mode**: Type `|` characters, tables auto-format
- **Link following**: Press Enter on `[link](target)` navigates correctly
- **Spell checking**: Misspelled words show red underlines
- **TOC generation**: `:GenTocMarked` creates table of contents
- **Zen mode**: `:ZenMode` centers text and hides UI elements
- **Export**: `:Pandoc pdf` generates PDF from markdown

### Performance Validation
- **Startup time**: `nvim --startuptime startup.log file.md` under 200ms
- **Large files**: 10k+ line markdown files scroll smoothly
- **Preview responsiveness**: Changes appear in preview within 1 second
- **Memory usage**: `:lua print(collectgarbage("count"))` shows reasonable usage

### Common Error Patterns
```
Error executing vim.schedule lua callback: ...markdown-preview.nvim/app/server.js not found
```
**Cause**: markdown-preview.nvim build failed during installation
**Fix**: `:Lazy build markdown-preview.nvim` or manually run `cd ~/.local/share/nvim/lazy/markdown-preview.nvim/app && npm install`

```
E5108: Error executing lua ...ltex_extra.lua:123: attempt to index field 'clients' (a nil value)
```
**Cause**: ltex-extra plugin incompatibility with Neovim version
**Fix**: Update plugin or use alternative grammar checker

```
spell: Cannot find word list "en.utf-8.spl"
```
**Cause**: Missing spell files for spell checking
**Fix**: `:set spell` downloads spell files automatically, or manually download

## Troubleshooting

### Markdown Preview Not Working
**Symptoms**: `:MarkdownPreview` command not found or browser doesn't open
**Cause**: Node.js not installed or build process failed
**Fix**: Install Node.js, then `:Lazy build markdown-preview.nvim`

### Tables Not Auto-Formatting
**Symptoms**: Typing `|` doesn't trigger table alignment
**Cause**: Table mode not enabled or wrong filetype detection
**Fix**: `:TableModeToggle` manually, verify `set ft=markdown`

### Poor Performance with Large Files
**Symptoms**: Lag when editing large markdown documents
**Cause**: Tree-sitter parsing overhead or too many active plugins
**Fix**: Disable syntax highlighting for large files: `:set syntax=off`

### Links Not Following Correctly
**Symptoms**: Enter key doesn't navigate to linked files
**Cause**: mkdnflow plugin not configured or wrong file paths
**Fix**: Check link paths are relative to current file location

### Spell Check False Positives
**Symptoms**: Correct words flagged as misspelled
**Cause**: Missing words in dictionary or wrong language setting
**Fix**: Add words with `zg` or change `:set spelllang=en_us`

### Export Failures
**Symptoms**: Pandoc commands fail or produce empty files
**Cause**: Pandoc not installed or missing dependencies
**Fix**: Install pandoc: `brew install pandoc` (macOS) or `apt install pandoc` (Linux)

## Authoritative References

- [Neovim Documentation](https://neovim.io/doc/)
- [Tree-sitter Documentation](https://tree-sitter.github.io/tree-sitter/)
- [Markdown Specification](https://spec.commonmark.org/)
- [Pandoc Manual](https://pandoc.org/MANUAL.html)
- [LSP Specification](https://microsoft.github.io/language-server-protocol/)
- [Lazy.nvim Documentation](https://lazy.folke.io/)