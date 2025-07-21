# Neovim Python Development Playbook

## Key Concepts

- **LSP (Language Server Protocol)**: Client-server architecture for IDE features like autocomplete, diagnostics, and code navigation
- **BasedPyright**: Fork of Pyright with enhanced type checking and Pylance features for Python
- **Ruff**: Fast Python linter and formatter written in Rust, replaces multiple tools (flake8, black, isort)
- **Mason.nvim**: Package manager for LSP servers, DAP adapters, linters, and formatters
- **DAP (Debug Adapter Protocol)**: Protocol for debugger integration in editors
- **Neotest**: Extensible testing framework for running and managing tests within Neovim
- **UV**: Modern Python package manager and project management tool, 10-100x faster than pip
- **Virtual Environment Detection**: Automatic detection of venv, UV projects, Poetry, and Conda environments
- **nvim-cmp**: Completion engine with multiple sources (LSP, snippets, buffer, path)

## Common Patterns

### Core Python LSP Setup with BasedPyright + Ruff
```lua
{
  "neovim/nvim-lspconfig",
  event = { "BufReadPre", "BufNewFile" },
  dependencies = {
    "williamboman/mason.nvim",
    "williamboman/mason-lspconfig.nvim",
    "hrsh7th/cmp-nvim-lsp",
  },
  priority = 100,
  config = function()
    local lspconfig = require("lspconfig")
    local capabilities = require("cmp_nvim_lsp").default_capabilities()
    
    -- BasedPyright configuration
    lspconfig.basedpyright.setup({
      capabilities = capabilities,
      settings = {
        basedpyright = {
          analysis = {
            typeCheckingMode = "basic",
            autoSearchPaths = true,
            useLibraryCodeForTypes = true,
            exclude = { "**/node_modules", "**/__pycache__" },
            ignore = { "**/migrations" },
          },
        },
      },
    })

    -- Ruff LSP configuration
    lspconfig.ruff.setup({
      capabilities = capabilities,
      init_options = {
        settings = {
          logLevel = "error",
          args = {
            "--config", vim.fn.expand("~/ruff.toml")
          },
        }
      },
      on_attach = function(client, bufnr)
        -- Disable hover in favor of BasedPyright
        client.server_capabilities.hoverProvider = false
      end,
    })

    -- LSP keybindings
    vim.api.nvim_create_autocmd('LspAttach', {
      callback = function(ev)
        local opts = { buffer = ev.buf, silent = true }
        
        vim.keymap.set('n', 'gd', vim.lsp.buf.definition, opts)
        vim.keymap.set('n', 'K', vim.lsp.buf.hover, opts)
        vim.keymap.set('n', 'gi', vim.lsp.buf.implementation, opts)
        vim.keymap.set('n', '<C-k>', vim.lsp.buf.signature_help, opts)
        vim.keymap.set('n', '<leader>rn', vim.lsp.buf.rename, opts)
        vim.keymap.set({ 'n', 'v' }, '<leader>ca', vim.lsp.buf.code_action, opts)
        vim.keymap.set('n', 'gr', vim.lsp.buf.references, opts)
        vim.keymap.set('n', '<leader>f', function()
          vim.lsp.buf.format { async = true }
        end, opts)
      end,
    })
  end,
}
```
**When to use**: Primary setup for Python language server support
**Expected output**: LSP diagnostics, hover info, goto definition working

### Mason Tool Installation
```lua
{
  "williamboman/mason.nvim",
  cmd = "Mason",
  keys = { { "<leader>cm", "<cmd>Mason<cr>", desc = "Mason" } },
  build = ":MasonUpdate",
  opts = {
    ensure_installed = {
      "basedpyright",
      "ruff",
      "debugpy",
      "mypy",
    },
  },
},
{
  "williamboman/mason-lspconfig.nvim",
  dependencies = { "mason.nvim" },
  -- Note: No automatic setup here to avoid loading conflicts
  -- LSP setup is handled in nvim-lspconfig configuration
}
```
**When to use**: Automatic installation of Python development tools
**Expected output**: Tools installed in `~/.local/share/nvim/mason/bin/`

### UV Integration Plugin
```lua
{
  "benomahony/uv.nvim",
  ft = "python",
  opts = {
    auto_activate = true,
    auto_commands = true,
  },
  keys = {
    { "<leader>xr", "<cmd>UVRun<cr>", desc = "UV Run" },
    { "<leader>xa", "<cmd>UVAdd<cr>", desc = "UV Add Package" },
    { "<leader>xd", "<cmd>UVRemove<cr>", desc = "UV Remove Package" },
    { "<leader>xi", "<cmd>UVInit<cr>", desc = "UV Init Project" },
    { "<leader>xs", "<cmd>UVSync<cr>", desc = "UV Sync" },
  },
}
```
**When to use**: Seamless UV package management within Neovim
**Expected output**: UV commands available, automatic venv activation

### Python Debugging with DAP
```lua
{
  "mfussenegger/nvim-dap",
  dependencies = {
    "mfussenegger/nvim-dap-python",
    "rcarriga/nvim-dap-ui",
    "theHamsta/nvim-dap-virtual-text",
    "nvim-neotest/nvim-nio",
  },
  config = function()
    local dap = require("dap")
    local dapui = require("dapui")
    
    -- Setup DAP UI
    dapui.setup()
    
    -- Setup Python debugging
    require("dap-python").setup("~/.local/share/nvim/mason/packages/debugpy/venv/bin/python")
    
    -- Auto-open/close DAP UI
    dap.listeners.after.event_initialized["dapui_config"] = function()
      dapui.open()
    end
    dap.listeners.before.event_terminated["dapui_config"] = function()
      dapui.close()
    end
  end,
  keys = {
    { "<F5>", function() require("dap").continue() end, desc = "Debug Continue" },
    { "<F10>", function() require("dap").step_over() end, desc = "Debug Step Over" },
    { "<F11>", function() require("dap").step_into() end, desc = "Debug Step Into" },
    { "<F12>", function() require("dap").step_out() end, desc = "Debug Step Out" },
    { "<leader>db", function() require("dap").toggle_breakpoint() end, desc = "Debug Breakpoint" },
    { "<leader>dr", function() require("dap").repl.open() end, desc = "Debug REPL" },
  },
}
```
**When to use**: Full debugging support for Python applications
**Expected output**: Breakpoints, step debugging, variable inspection working

### Python Testing with Neotest
```lua
{
  "nvim-neotest/neotest",
  dependencies = {
    "nvim-neotest/nvim-nio",
    "nvim-lua/plenary.nvim",
    "antoinemadec/FixCursorHold.nvim",
    "nvim-treesitter/nvim-treesitter",
    "nvim-neotest/neotest-python",
  },
  config = function()
    require("neotest").setup({
      adapters = {
        require("neotest-python")({
          runner = "pytest",
          python = function()
            -- Use UV project Python if available
            local uv_python = vim.fn.system("uv run which python"):gsub("\n", "")
            if vim.v.shell_error == 0 then
              return uv_python
            end
            return vim.fn.exepath("python3") or vim.fn.exepath("python")
          end,
          args = { "--log-level", "DEBUG" },
          dap = { justMyCode = false },
        }),
      },
    })
  end,
  keys = {
    { "<leader>tt", function() require("neotest").run.run() end, desc = "Test Nearest" },
    { "<leader>tf", function() require("neotest").run.run(vim.fn.expand("%")) end, desc = "Test File" },
    { "<leader>ta", function() require("neotest").run.run(vim.fn.getcwd()) end, desc = "Test All" },
    { "<leader>ts", function() require("neotest").summary.toggle() end, desc = "Test Summary" },
    { "<leader>to", function() require("neotest").output.open({ enter = true }) end, desc = "Test Output" },
  },
}
```
**When to use**: Running and managing Python tests (pytest/unittest)
**Expected output**: Test results, coverage, and debugging integration

### Autocompletion with nvim-cmp
```lua
{
  "hrsh7th/nvim-cmp",
  event = "InsertEnter",
  dependencies = {
    "hrsh7th/cmp-nvim-lsp",
    "hrsh7th/cmp-buffer",
    "hrsh7th/cmp-path",
    "L3MON4D3/LuaSnip",
    "saadparwaiz1/cmp_luasnip",
    "rafamadriz/friendly-snippets",
  },
  config = function()
    local cmp = require("cmp")
    local luasnip = require("luasnip")

    require("luasnip.loaders.from_vscode").lazy_load()

    cmp.setup({
      snippet = {
        expand = function(args)
          luasnip.lsp_expand(args.body)
        end,
      },
      mapping = cmp.mapping.preset.insert({
        ["<C-Space>"] = cmp.mapping.complete(),
        ["<CR>"] = cmp.mapping.confirm({ select = true }),
        ["<Tab>"] = cmp.mapping(function(fallback)
          if cmp.visible() then
            cmp.select_next_item()
          elseif luasnip.expand_or_jumpable() then
            luasnip.expand_or_jump()
          else
            fallback()
          end
        end, { "i", "s" }),
      }),
      sources = cmp.config.sources({
        { name = "nvim_lsp" },
        { name = "luasnip" },
        { name = "buffer" },
        { name = "path" },
      }),
    })
  end,
}
```
**When to use**: Intelligent autocompletion for Python development
**Expected output**: LSP-powered completions, snippet expansion, buffer completions

## Implementation Details

### Ruff Configuration File Setup
1. Create Ruff configuration file:
```bash
touch ~/ruff.toml
```

2. Add basic Ruff configuration:
```toml
[tool.ruff]
# Ruff configuration file
line-length = 88
target-version = "py38"

[tool.ruff.lint]
select = ["E", "W", "F"]
ignore = []

[tool.ruff.format]
quote-style = "double"
indent-style = "space"
```

3. For project-specific configuration:
```bash
# In your project root
touch pyproject.toml
```

```toml
[tool.ruff]
line-length = 100
target-version = "py311"

[tool.ruff.lint]
select = ["E", "W", "F", "I"]
ignore = ["E501"]
```

**Validation**: Ruff LSP should stop showing workspace settings errors
**Common errors**: Missing configuration file = "LSP[ruff] Error while resolving settings"

### UV Project Setup Workflow
1. Initialize new Python project:
```bash
uv init my-python-project
cd my-python-project
```

2. Add dependencies:
```bash
uv add fastapi uvicorn
uv add --dev pytest black ruff mypy
```

3. Create virtual environment (if not auto-created):
```bash
uv venv
```

4. Activate and run:
```bash
uv run python main.py
uv run pytest
```

**Validation**: `uv sync` shows all dependencies installed correctly
**Common errors**: `command not found: uv` = UV not installed or not in PATH

### LSP Server Configuration Validation
1. Check LSP status:
```vim
:LspInfo
```

2. Check Mason installation:
```vim
:Mason
```

3. Verify active LSP clients:
```vim
:lua for _, client in ipairs(vim.lsp.get_active_clients()) do print('LSP Client:', client.name, 'ID:', client.id) end
```

4. Check capabilities:
```vim
:lua =vim.lsp.get_clients()[1].server_capabilities
```

5. Test with headless mode:
```bash
# Check if both servers attach to Python files
nvim --headless test.py -c "sleep 3" -c "lua for _, client in ipairs(vim.lsp.get_active_clients()) do print('- ' .. client.name) end" -c "qa"

# Verify tools are installed
nvim --headless -c "lua local mason = require('mason-registry'); print('BasedPyright:', mason.is_installed('basedpyright')); print('Ruff:', mason.is_installed('ruff'))" -c "qa"
```

**Validation**: BasedPyright and Ruff show as "attached" in LspInfo
**Common errors**: `No client with id` = LSP server not started

### Debug Configuration Verification
1. Set breakpoint and run:
```vim
:lua require('dap').toggle_breakpoint()
:lua require('dap').continue()
```

2. Check DAP configurations:
```vim
:lua vim.print(require('dap').configurations.python)
```

**Validation**: Debugger stops at breakpoints, variables visible in DAP UI
**Common errors**: `No configuration found` = debugpy not installed or configured wrong

### Virtual Environment Detection Testing
1. Check detected Python path:
```vim
:lua print(vim.fn.exepath('python'))
```

2. Verify LSP uses correct Python:
```vim
:LspInfo
```

3. Test UV integration:
```vim
:UVStatus
```

**Validation**: LSP shows Python path matching project venv
**Common errors**: LSP using global Python = venv not activated properly

## Troubleshooting

### Mason-LSPConfig Plugin Loading Error
**Symptoms**: Error message "attempt to call field 'enable' (a nil value)" during startup
**Cause**: mason-lspconfig trying to auto-enable LSP servers before nvim-lspconfig is loaded
**Fix**:
1. Remove automatic setup from mason-lspconfig configuration
2. Handle LSP setup in nvim-lspconfig config function instead
3. Ensure proper dependency order: mason.nvim → mason-lspconfig.nvim → nvim-lspconfig

**Correct mason-lspconfig configuration**:
```lua
{
  "williamboman/mason-lspconfig.nvim",
  dependencies = { "mason.nvim" },
  -- No automatic setup to avoid conflicts
}
```

### Ruff LSP Workspace Settings Error
**Symptoms**: "LSP[ruff] Error while resolving settings from workspace" messages
**Cause**: Missing Ruff configuration file or incorrect workspace settings
**Fix**:
1. Create `~/ruff.toml` configuration file (see Ruff Configuration File Setup section)
2. Add proper logLevel setting in Ruff LSP configuration:
```lua
lspconfig.ruff.setup({
  init_options = {
    settings = {
      logLevel = "error",
      args = { "--config", vim.fn.expand("~/ruff.toml") },
    }
  },
})
```
3. Disable Ruff hover to prevent conflicts with BasedPyright

### LSP Not Starting
**Symptoms**: No diagnostics, hover info, or completions in Python files
**Cause**: LSP server not installed or configured incorrectly
**Fix**: 
1. Run `:Mason` and install `basedpyright` and `ruff`
2. Check `:LspInfo` for error messages
3. Restart LSP with `:LspRestart`

### Debugger Not Working
**Symptoms**: Breakpoints not hit, DAP UI not opening
**Cause**: debugpy not installed or incorrect Python path
**Fix**:
1. Install debugpy: `uv add --dev debugpy`
2. Update DAP Python path: `require("dap-python").setup("path/to/python")`
3. Verify configuration: `:lua vim.print(require('dap').configurations.python)`

### Tests Not Discovered
**Symptoms**: Neotest shows no tests found
**Cause**: Incorrect test file naming or pytest not available
**Fix**:
1. Ensure test files follow `test_*.py` or `*_test.py` pattern
2. Install pytest: `uv add --dev pytest`
3. Check Neotest status: `:lua require('neotest').status.summary()`

### Virtual Environment Not Detected
**Symptoms**: LSP uses global Python, imports not found
**Cause**: Venv not activated or UV project not recognized
**Fix**:
1. Run `uv sync` to ensure project is properly set up
2. Check UV status: `:UVStatus`
3. Manually activate: `source .venv/bin/activate` before starting Neovim

### Slow Autocompletion
**Symptoms**: Long delays when typing, LSP timeouts
**Cause**: Heavy LSP analysis or too many completion sources
**Fix**:
1. Reduce BasedPyright analysis scope in settings
2. Use `event = "VeryLazy"` for non-essential plugins
3. Disable unused completion sources in nvim-cmp

### UV Commands Not Working
**Symptoms**: `:UVAdd` or other UV commands not found
**Cause**: uv.nvim plugin not loaded or UV not installed
**Fix**:
1. Install UV: `curl -LsSf https://astral.sh/uv/install.sh | sh`
2. Ensure uv.nvim loads on Python filetypes
3. Check plugin status: `:Lazy load uv.nvim`

## Configuration Validation

### Automated Setup Verification
```bash
#!/bin/bash
# Validate Python development setup

echo "Testing Neovim Python configuration..."

# Test configuration loads without errors
nvim --headless -c "lua print('Configuration loaded successfully')" -c "qa"

# Check LSP servers attach to Python files
echo "print('test')" > test_validation.py
nvim --headless test_validation.py -c "sleep 3" -c "lua for _, client in ipairs(vim.lsp.get_active_clients()) do print('LSP:', client.name) end" -c "qa"
rm test_validation.py

# Verify Mason tools
nvim --headless -c "lua local mason = require('mason-registry'); print('Tools installed:'); for _, tool in ipairs({'basedpyright', 'ruff', 'debugpy'}) do print('-', tool, ':', mason.is_installed(tool)) end" -c "qa"

echo "Validation complete!"
```

### Error Detection Testing
```bash
# Create test file with intentional errors
cat > error_test.py << 'EOF'
def test_func(x: int) -> int:
    return x + "string"  # Type error

undefined_var  # Name error
EOF

# Check if LSP detects errors
nvim --headless error_test.py -c "sleep 5" -c "lua local diagnostics = vim.diagnostic.get(0); print('Errors found:', #diagnostics); for _, d in ipairs(diagnostics) do print('Line', d.lnum + 1, ':', d.message) end" -c "qa"

rm error_test.py
```

## Configuration Best Practices

### Plugin Loading Order
**Critical**: Plugins must load in correct order to avoid conflicts

1. **Mason Core** (mason.nvim) - Must load first
2. **Mason LSP Bridge** (mason-lspconfig.nvim) - Depends on Mason, no automatic setup
3. **LSP Configuration** (nvim-lspconfig) - Handles actual LSP server setup
4. **Completion** (nvim-cmp, cmp-nvim-lsp) - Integrates with LSP
5. **Language-specific** (DAP, UV integration) - Load after core LSP

**Correct dependency chain**:
```lua
-- 1. Mason first
{ "williamboman/mason.nvim" }

-- 2. Mason-LSPConfig (no automatic setup)
{ "williamboman/mason-lspconfig.nvim", dependencies = { "mason.nvim" } }

-- 3. LSP Config with priority
{ 
  "neovim/nvim-lspconfig", 
  priority = 100,
  dependencies = { "mason.nvim", "mason-lspconfig.nvim", "cmp-nvim-lsp" }
}
```

### Multi-Project Workspace Setup
```lua
-- Per-project configuration
vim.api.nvim_create_autocmd("DirChanged", {
  callback = function()
    -- Reload LSP servers when changing projects
    vim.cmd("LspRestart")
    
    -- Update UV project context
    if vim.fn.filereadable("pyproject.toml") == 1 then
      vim.g.current_uv_project = vim.fn.getcwd()
    end
  end,
})
```

## Authoritative References

- [Neovim LSP Documentation](https://neovim.io/doc/user/lsp.html)
- [nvim-lspconfig Server Configurations](https://github.com/neovim/nvim-lspconfig/blob/master/doc/server_configurations.md)
- [Mason.nvim Package Registry](https://mason-registry.dev/registry/list)
- [BasedPyright Documentation](https://docs.basedpyright.com/)
- [Ruff Configuration Guide](https://docs.astral.sh/ruff/configuration/)
- [UV Official Documentation](https://docs.astral.sh/uv/)
- [nvim-dap Configuration](https://github.com/mfussenegger/nvim-dap/wiki)
- [Neotest Adapters](https://github.com/nvim-neotest/neotest#supported-runners)
- [nvim-cmp Sources](https://github.com/hrsh7th/nvim-cmp/wiki/List-of-sources)