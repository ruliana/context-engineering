# Neovim Lazy Package Manager Context

## Key Concepts

- **init.lua**: Main Neovim configuration file located at `~/.config/nvim/init.lua`
- **Lazy.nvim**: Modern plugin manager with automatic lazy-loading and async execution
- **Plugin Spec**: Lua table defining plugin configuration, dependencies, and loading behavior
- **Lazy Loading**: Deferred plugin loading based on events, commands, or file types
- **Bootstrap**: Automatic installation code that downloads Lazy.nvim if not present
- **Leader Key**: Primary key prefix for custom mappings (typically space or backslash)
- **Lockfile**: `lazy-lock.json` file tracking exact plugin versions for reproducibility

## Common Patterns

### Basic Installation Pattern
```lua
-- Bootstrap lazy.nvim
local lazypath = vim.fn.stdpath("data") .. "/lazy/lazy.nvim"
if not (vim.uv or vim.loop).fs_stat(lazypath) then
  local lazyrepo = "https://github.com/folke/lazy.nvim.git"
  local out = vim.fn.system({
    "git", "clone", 
    "--filter=blob:none", 
    "--branch=stable", 
    lazyrepo, 
    lazypath 
  })
  if vim.v.shell_error ~= 0 then
    vim.api.nvim_echo({
      { "Failed to clone lazy.nvim\n", "ErrorMsg" },
      { out, "WarningMsg" }
    }, true, {})
    os.exit(1)
  end
end
vim.opt.rtp:prepend(lazypath)

-- Set leader keys before lazy setup
vim.g.mapleader = " "
vim.g.maplocalleader = "\\"

-- Initialize lazy.nvim
require("lazy").setup({
  spec = {
    { import = "plugins" },
  },
  install = { colorscheme = { "habamax" } },
  checker = { enabled = true },
})
```
**When to use**: Initial Neovim setup with Lazy.nvim
**Expected output**: Lazy.nvim UI opens with `:Lazy` command

### Simple Plugin Specification
```lua
{
  "folke/todo-comments.nvim",
  opts = {}
}
```
**When to use**: Basic plugin with default configuration
**Expected output**: Plugin loads automatically on startup

### Event-Based Lazy Loading
```lua
{
  "nvim-treesitter/nvim-treesitter",
  event = { "BufReadPre", "BufNewFile" },
  build = ":TSUpdate",
  opts = {
    highlight = { enable = true },
    indent = { enable = true },
  }
}
```
**When to use**: Heavy plugins that should load only when editing files
**Expected output**: Plugin loads when opening/creating files

### Command-Based Lazy Loading
```lua
{
  "nvim-telescope/telescope.nvim",
  cmd = "Telescope",
  dependencies = { "nvim-lua/plenary.nvim" },
  opts = {}
}
```
**When to use**: Plugins accessed via specific commands
**Expected output**: Plugin loads when running `:Telescope` command

### Filetype-Based Lazy Loading
```lua
{
  "fatih/vim-go",
  ft = "go",
  config = function()
    vim.g.go_def_mode = 'gopls'
  end
}
```
**When to use**: Language-specific plugins
**Expected output**: Plugin loads only for Go files

### Key Mapping Lazy Loading
```lua
{
  "folke/which-key.nvim",
  keys = { "<leader>", "<c-r>", "<c-w>", '"', "'", "`", "c", "v", "g" },
  config = function()
    require("which-key").setup()
  end
}
```
**When to use**: Plugins triggered by specific key combinations
**Expected output**: Plugin loads when pressing defined keys

## Implementation Details

### Directory Structure Setup
1. Create configuration directory:
```bash
mkdir -p ~/.config/nvim/lua/plugins
mkdir -p ~/.config/nvim/lua/config
```

2. Create main init.lua file:
```bash
touch ~/.config/nvim/init.lua
```

3. Create plugin directory:
```bash
touch ~/.config/nvim/lua/plugins/init.lua
```

**Validation**: `ls -la ~/.config/nvim/` shows proper structure
**Common errors**: `E5113: Error while calling lua chunk` = syntax error in Lua files

### Plugin Specification Patterns

#### Local Plugin Development
```lua
{
  dir = "~/projects/my-plugin",
  name = "my-plugin",
  config = function()
    require("my-plugin").setup()
  end
}
```

#### Version Pinning
```lua
{
  "folke/tokyonight.nvim",
  tag = "v2.9.0",
  priority = 1000,
  config = function()
    vim.cmd.colorscheme("tokyonight")
  end
}
```

#### Conditional Loading
```lua
{
  "nvim-neo-tree/neo-tree.nvim",
  enabled = function()
    return vim.fn.executable("git") == 1
  end,
  dependencies = {
    "nvim-lua/plenary.nvim",
    "nvim-tree/nvim-web-devicons",
    "MunifTanjim/nui.nvim"
  }
}
```

### Plugin Import Structure
```lua
-- In ~/.config/nvim/lua/plugins/editor.lua
return {
  {
    "folke/flash.nvim",
    event = "VeryLazy",
    opts = {},
    keys = {
      { "s", mode = { "n", "x", "o" }, function() require("flash").jump() end, desc = "Flash" },
    },
  }
}
```

## Validation Methods

### Installation Verification
- **Check Lazy status**: `:Lazy` opens plugin manager UI
- **Health check**: `:checkhealth lazy` shows system status
- **Plugin list**: `:Lazy show` displays installed plugins
- **Update check**: `:Lazy update` fetches latest versions

### Configuration Testing
- **Reload config**: `:so %` (source current file)
- **Lua execution**: `:lua print("test")` shows "test" output
- **Error checking**: `:messages` displays recent error messages
- **Plugin loading**: `:Lazy profile` shows startup performance

### Common Error Patterns
```
E5113: Error while calling lua chunk
```
**Cause**: Lua syntax error in configuration files
**Fix**: Check for missing commas, quotes, or brackets

```
Failed to clone lazy.nvim
```
**Cause**: Network connectivity or git configuration issues
**Fix**: Verify internet connection and git installation

```
module 'lazy' not found
```
**Cause**: Bootstrap code not executed or failed
**Fix**: Ensure bootstrap code runs before require("lazy")

## Troubleshooting

### Plugin Not Loading
**Symptoms**: Plugin functionality unavailable, no error messages
**Cause**: Incorrect lazy loading conditions or missing dependencies
**Fix**: Check `event`, `cmd`, `ft`, or `keys` specifications match usage

### Slow Startup Time
**Symptoms**: Neovim takes >1 second to start
**Cause**: Too many plugins loading on startup
**Fix**: Add lazy loading with `event = "VeryLazy"` or specific triggers

### Configuration Not Applied
**Symptoms**: Plugin settings not taking effect
**Cause**: Configuration function not called or incorrect opts
**Fix**: Use `config = function()` instead of `opts` for complex setups

### Dependency Conflicts
**Symptoms**: Error messages about missing modules
**Cause**: Plugin dependencies not properly specified
**Fix**: Add missing dependencies to `dependencies` table

### Update Failures
**Symptoms**: `:Lazy update` shows errors for specific plugins
**Cause**: Git conflicts or plugin repository issues
**Fix**: Use `:Lazy clean` then `:Lazy install` to reset plugins

## Authoritative References

- [Lazy.nvim Official Documentation](https://lazy.folke.io/)
- [Lazy.nvim GitHub Repository](https://github.com/folke/lazy.nvim)
- [Neovim Lua Guide](https://neovim.io/doc/user/lua-guide.html)
- [Neovim API Documentation](https://neovim.io/doc/user/api.html)