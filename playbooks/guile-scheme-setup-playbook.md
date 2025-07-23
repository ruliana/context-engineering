# Guile Scheme Setup Playbook

## Key Concepts

- **Guile**: GNU's implementation of the Scheme programming language (R5RS, R6RS, R7RS compliant)
- **Guild**: Guile's compiler that produces bytecode (.go files) from Scheme source
- **GUILE_LOAD_PATH**: Environment variable controlling where Guile searches for source modules
- **GUILE_LOAD_COMPILED_PATH**: Environment variable controlling where Guile searches for compiled modules (.go files)
- **REPL**: Read-Eval-Print Loop, Guile's interactive development environment
- **Site directory**: System-wide location for installed Guile modules (`/usr/share/guile/site/X.Y`)
- **User directory**: Per-user location for Guile modules (`~/.local/share/guile/site/X.Y`)
- **pre-inst-env**: Development script pattern for testing uninstalled modules

## Common Patterns

### System Installation Pattern

```bash
# Ubuntu/Debian
sudo apt update && sudo apt install guile-3.0 guile-3.0-dev

# macOS with Homebrew
brew install guile
```

**When to use**: Fresh system setup or global Guile installation  
**Expected output**: `guile --version` shows `Guile 3.0.x`

### Project Isolation Pattern

```bash
# Create project directory structure
mkdir -p myproject/{src,lib,scripts}
cd myproject

# Create project environment script
cat > env.sh << 'EOF'
#!/bin/bash
export GUILE_LOAD_PATH="$(pwd)/src:$(pwd)/lib:${GUILE_LOAD_PATH}"
export GUILE_LOAD_COMPILED_PATH="$(pwd)/.go:${GUILE_LOAD_COMPILED_PATH}"
export PATH="$(pwd)/scripts:${PATH}"
EOF

# Activate project environment
source env.sh
guile -c "(display %load-path) (newline)"
```

**When to use**: Individual project development requiring module isolation  
**Expected output**: Load path shows project directories first

### Module Development Pattern

```bash
# Create pre-inst-env for development
cat > pre-inst-env << 'EOF'
#!/bin/sh
GUILE_LOAD_PATH="$(pwd):${GUILE_LOAD_PATH}"
GUILE_LOAD_COMPILED_PATH="$(pwd)/.go:${GUILE_LOAD_COMPILED_PATH}"
export GUILE_LOAD_PATH GUILE_LOAD_COMPILED_PATH
exec "$@"
EOF
chmod +x pre-inst-env

# Test uninstalled modules
./pre-inst-env guile -c "(use-modules (my-module))"
```

**When to use**: Developing Guile modules before installation  
**Expected output**: Module loads without installation errors

## Implementation Details

### Ubuntu Installation

```bash
# Install Guile and development tools
sudo apt update
sudo apt install guile-3.0 guile-3.0-dev build-essential

# Verify installation
guile --version
guild --version

# Check module paths
guile -c "(display %load-path) (newline)"
```

**Validation**: `guile --version` returns version 3.0.x or higher  
**Common errors**: `E: Package 'guile-3.0' has no installation candidate` = enable universe repository

### macOS Installation

```bash
# Install via Homebrew
brew install guile

# Optional: Add Guile Homebrew tap for additional libraries
brew tap aconchillo/guile
brew install guile-json guile-sqlite3

# Verify installation
which guile
guile --version
```

**Validation**: `/usr/local/bin/guile` exists and is executable  
**Common errors**: `command not found: guile` = check PATH includes `/usr/local/bin`

### Project Environment Setup

```bash
# Create project structure
mkdir -p myproject/{modules,compiled,bin,tests}
cd myproject

# Initialize project configuration
cat > .guile-project << 'EOF'
(use-modules (ice-9 ftw))

(define project-root (getcwd))
(define modules-dir (string-append project-root "/modules"))
(define compiled-dir (string-append project-root "/compiled"))

(set! %load-path (cons modules-dir %load-path))
(set! %load-compiled-path (cons compiled-dir %load-compiled-path))
EOF

# Create activation script
cat > activate << 'EOF'
#!/bin/bash
export GUILE_LOAD_PATH="$(pwd)/modules:${GUILE_LOAD_PATH}"
export GUILE_LOAD_COMPILED_PATH="$(pwd)/compiled:${GUILE_LOAD_COMPILED_PATH}" 
export GUILE_INIT_FILE="$(pwd)/.guile-project"
export PS1="(guile-project) $PS1"
EOF
chmod +x activate

# Activate environment
source activate
```

**Validation**: `echo $GUILE_LOAD_PATH` shows project directories first  
**Common errors**: Permission denied = ensure activate script is executable

### Module Compilation

```bash
# Compile single module
guild compile -o compiled/mymodule.go modules/mymodule.scm

# Compile all modules
find modules -name "*.scm" -exec guild compile -o compiled/{}.go {} \;

# Compile with optimization
guild compile -O2 -o compiled/mymodule.go modules/mymodule.scm
```

**Validation**: `.go` files created in compiled directory  
**Common errors**: `guild: command not found` = install guile-dev package

### Development Workflow

```bash
# Create module template
mkdir -p modules/myproject
cat > modules/myproject/core.scm << 'EOF'
(define-module (myproject core)
  #:export (hello-world))

(define (hello-world name)
  (format #t "Hello, ~a!~%" name))
EOF

# Test module interactively
guile -c "
(add-to-load-path \"$(pwd)/modules\")
(use-modules (myproject core))
(hello-world \"Guile\")"

# Create executable script
cat > bin/hello << 'EOF'
#!/usr/bin/env guile
!#
(add-to-load-path (string-append (dirname (car (command-line))) "/../modules"))
(use-modules (myproject core))
(hello-world "World")
EOF
chmod +x bin/hello

./bin/hello
```

**Validation**: Script outputs "Hello, World!"  
**Common errors**: Module not found = check GUILE_LOAD_PATH configuration

## Troubleshooting

### Module Not Found Error

**Symptoms**: `ERROR: no code for module (my-module)`  
**Cause**: Module not in load path or incorrect module definition  
**Fix**: Check `%load-path` and verify module file structure matches module name

### Guild Compilation Fails

**Symptoms**: `guild: compile: warning: compilation of X failed`  
**Cause**: Syntax errors or missing dependencies  
**Fix**: Check syntax with `guile -c "(load \"file.scm\")"` first

### Permission Denied on Scripts

**Symptoms**: `bash: ./script: Permission denied`  
**Cause**: Script not executable  
**Fix**: `chmod +x script` or `bash script`

### Environment Variables Not Persistent

**Symptoms**: Variables reset after closing terminal  
**Cause**: Variables not saved to shell profile  
**Fix**: Add exports to `~/.bashrc` or `~/.zshrc`

### Homebrew Installation Issues (macOS)

**Symptoms**: `Error: guile: no bottle available!`  
**Cause**: Architecture mismatch or outdated Homebrew  
**Fix**: `brew update && brew upgrade` then retry installation

### Ubuntu Package Not Available

**Symptoms**: `Package 'guile-3.0' has no installation candidate`  
**Cause**: Universe repository not enabled  
**Fix**: `sudo add-apt-repository universe && sudo apt update`

## Authoritative References

- [GNU Guile Manual](https://www.gnu.org/software/guile/manual/guile.html)
- [Guile Reference Manual - Environment Variables](https://www.gnu.org/software/guile/manual/html_node/Environment-Variables.html)
- [Guile Reference Manual - Load Paths](https://www.gnu.org/software/guile/manual/html_node/Load-Paths.html)
- [GNU Guile Download Page](https://www.gnu.org/software/guile/download/)
- [Guile Installation Guide](https://www.gnu.org/software/guile/manual/html_node/Obtaining-and-Installing-Guile.html)