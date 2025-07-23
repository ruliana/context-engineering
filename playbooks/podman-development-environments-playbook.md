# Podman Development Environments Playbook

## Key Concepts

- **Podman**: Daemonless, rootless container management tool for creating and maintaining OCI containers
- **Rootless Containers**: Containers running without root privileges, enhancing security for development
- **Bind Mounts**: Direct mapping of host filesystem directories into containers for persistent code access
- **Container Images**: Read-only templates containing application code and dependencies
- **Pods**: Groups of one or more containers sharing network and storage
- **OCI Compliance**: Open Container Initiative standards ensuring container portability
- **Machine**: VM-based container runtime for macOS (similar to Docker Desktop)

## Installation Patterns

### macOS Setup
```bash
# Install via Homebrew
brew install podman

# Initialize and start machine
podman machine init
podman machine start

# Verify installation
podman info
```
**Expected output**: Machine status shows "Running", connection details displayed

### Linux Setup (Debian/Ubuntu)
```bash
# Install Podman
sudo apt-get update && sudo apt-get -y install podman

# Verify rootless setup
podman info --format json | jq '.host.security.rootless'

# Test basic functionality
podman run --rm hello-world
```
**Expected output**: `true` for rootless query, "Hello from Docker!" message

### Linux Setup (Fedora/RHEL)
```bash
# Install Podman
sudo dnf -y install podman

# For development machine features
sudo dnf -y install podman-machine

# Configure user namespace (if needed)
echo "$USER:100000:65536" | sudo tee -a /etc/subuid
echo "$USER:100000:65536" | sudo tee -a /etc/subgid
```
**Expected output**: Successful installation, subuid/subgid entries added

## Common Development Patterns

### Basic Development Container
```bash
# Run Ubuntu container with code persistence
podman run -it --name devenv \
  --mount type=bind,source=$HOME/projects,destination=/workspace \
  --mount type=bind,source=$HOME/.gitconfig,destination=/root/.gitconfig,ro=true \
  ubuntu:22.04 bash
```
**When to use**: Single-project development, quick environment setup
**Expected output**: Interactive shell in container with `/workspace` containing host projects

### Node.js Development Environment
```bash
# Create Node.js dev container with volume persistence
podman run -d --name nodejs-dev \
  --mount type=bind,source=$PWD,destination=/app \
  --mount type=bind,source=$HOME/.npm,destination=/root/.npm \
  -p 3000:3000 -p 3001:3001 \
  -w /app \
  node:18-alpine sleep infinity

# Enter development environment
podman exec -it nodejs-dev sh
```
**When to use**: Node.js projects requiring npm package persistence
**Expected output**: Container running, npm cache persisted, ports accessible

### Python Development Environment
```bash
# Python container with pip cache and virtual environment
podman run -d --name python-dev \
  --mount type=bind,source=$PWD,destination=/workspace \
  --mount type=bind,source=$HOME/.cache/pip,destination=/root/.cache/pip \
  --mount type=tmpfs,destination=/tmp,tmpfs-size=512M \
  -p 8000:8000 -p 5000:5000 \
  -w /workspace \
  python:3.11-slim sleep infinity

# Install development dependencies
podman exec python-dev pip install -r requirements.txt
```
**When to use**: Python projects with dependency isolation
**Expected output**: Container with Python environment, pip cache preserved

### Database Integration Pattern
```bash
# PostgreSQL for development
podman run -d --name postgres-dev \
  -e POSTGRES_PASSWORD=devpass \
  -e POSTGRES_DB=devdb \
  --mount type=volume,source=postgres-data,destination=/var/lib/postgresql/data \
  -p 5432:5432 \
  postgres:15-alpine

# Application container with database connection
podman run -d --name app-dev \
  --mount type=bind,source=$PWD,destination=/app \
  --add-host=postgres:$(podman inspect postgres-dev --format '{{.NetworkSettings.IPAddress}}') \
  -p 8080:8080 \
  -w /app \
  node:18-alpine sleep infinity
```
**When to use**: Applications requiring persistent database during development
**Expected output**: PostgreSQL accessible at localhost:5432, app container can connect via 'postgres' hostname

## IDE Integration Patterns

### VSCode Integration
```bash
# Create development container with VSCode server
podman run -d --name vscode-dev \
  --mount type=bind,source=$HOME/projects,destination=/workspace \
  --mount type=bind,source=$HOME/.ssh,destination=/root/.ssh,ro=true \
  --mount type=bind,source=$HOME/.gitconfig,destination=/root/.gitconfig,ro=true \
  -p 8080:8080 \
  -e PASSWORD=devpassword \
  codercom/code-server:latest

# Access via browser: http://localhost:8080
```
**When to use**: Browser-based development, remote development scenarios
**Expected output**: VSCode interface accessible in browser, full project access

### VSCode Dev Containers Setup
```json
// .devcontainer/devcontainer.json
{
  "name": "Podman Development",
  "dockerComposeFile": "../docker-compose.yml",
  "service": "dev",
  "workspaceFolder": "/workspace",
  "customizations": {
    "vscode": {
      "extensions": [
        "ms-vscode.vscode-typescript-next"
      ]
    }
  }
}
```
**Expected output**: VSCode automatically uses Podman container for development

### Neovim Integration
```bash
# Create Neovim development environment
podman run -it --name nvim-dev \
  --mount type=bind,source=$HOME/projects,destination=/workspace \
  --mount type=bind,source=$HOME/.config/nvim,destination=/root/.config/nvim,ro=true \
  --mount type=bind,source=$HOME/.gitconfig,destination=/root/.gitconfig,ro=true \
  -w /workspace \
  alpine/neovim:latest-stable sh

# Install language servers in container
apk add --no-cache nodejs npm python3 py3-pip
npm install -g typescript-language-server
pip install python-lsp-server
```
**When to use**: Terminal-based development with Neovim
**Expected output**: Neovim with full configuration, language servers available

## Advanced Configuration Patterns

### Rootless Pod Development
```bash
# Create development pod
podman pod create --name dev-pod -p 3000:3000 -p 5432:5432 -p 8080:8080

# Add application container to pod
podman run -d --name app --pod dev-pod \
  --mount type=bind,source=$PWD,destination=/app \
  -w /app node:18-alpine sleep infinity

# Add database container to pod
podman run -d --name db --pod dev-pod \
  -e POSTGRES_PASSWORD=devpass \
  postgres:15-alpine

# Containers share network - app connects to db via localhost
```
**When to use**: Multi-service applications requiring network isolation
**Expected output**: Containers communicate via localhost, external access via pod ports

### Development Environment as Systemd Service
```bash
# Generate systemd unit file
podman generate systemd --new --name nodejs-dev > ~/.config/systemd/user/nodejs-dev.service

# Enable and start service
systemctl --user daemon-reload
systemctl --user enable nodejs-dev.service
systemctl --user start nodejs-dev.service

# Check status
systemctl --user status nodejs-dev.service
```
**When to use**: Persistent development environments across reboots
**Expected output**: Container automatically starts with user session

### Custom Development Image
```dockerfile
# Dockerfile.dev
FROM node:18-alpine
RUN apk add --no-cache git openssh-client
RUN npm install -g nodemon typescript
WORKDIR /app
CMD ["sleep", "infinity"]
```

```bash
# Build custom development image
podman build -f Dockerfile.dev -t my-node-dev .

# Use custom image
podman run -d --name custom-dev \
  --mount type=bind,source=$PWD,destination=/app \
  -p 3000:3000 my-node-dev
```
**When to use**: Standardized development environments across projects
**Expected output**: Consistent toolchain across different projects

### Volume Management
```bash
# Create named volume for dependencies
podman volume create node-modules

# Use named volume for dependencies
podman run -d --name app-dev \
  --mount type=bind,source=$PWD,destination=/app \
  --mount type=volume,source=node-modules,destination=/app/node_modules \
  -p 3000:3000 -w /app node:18-alpine sleep infinity

# List volumes
podman volume ls

# Inspect volume
podman volume inspect node-modules
```
**When to use**: Preserving build artifacts and dependencies across container restarts
**Expected output**: Fast container startup, preserved node_modules

## Implementation Details

### Starting Development Environment
1. `podman run -d --name <env-name> --mount type=bind,source=$PWD,destination=/workspace -p <ports> <image> sleep infinity`
2. `podman exec -it <env-name> bash`
3. Navigate to `/workspace` and begin development

**Validation**: `pwd` shows `/workspace`, `ls` shows your project files
**Common errors**: 
- `Error: container <name> already exists` = `podman rm <name>` first
- `Permission denied` = Check SELinux context with `ls -Z`

### Code Persistence Setup
1. Create project directory: `mkdir -p ~/projects/myapp`
2. Initialize project: `cd ~/projects/myapp && git init`
3. Mount project: `--mount type=bind,source=$HOME/projects/myapp,destination=/app`
4. Work in container: `podman exec -it <container> -w /app bash`

**Validation**: Changes made in container appear in `~/projects/myapp`
**Common errors**: 
- `No such file or directory` = Verify source path exists
- `Read-only file system` = Remove `,ro` from mount options

### Multi-Service Development
1. `podman pod create --name devstack -p 3000:3000 -p 5432:5432`
2. `podman run -d --name db --pod devstack postgres:15`
3. `podman run -d --name app --pod devstack --mount type=bind,source=$PWD,destination=/app node:18`
4. Connect services via `localhost:<port>`

**Validation**: `podman pod ps` shows running pod, services accessible
**Common errors**: 
- `Port already in use` = Use different ports or stop conflicting services
- `Connection refused` = Check if service is ready with `podman logs <container>`

## Troubleshooting

### Container Won't Start
**Symptoms**: `podman run` command fails or container exits immediately
**Cause**: Image not found, port conflicts, or invalid mount paths
**Fix**: 
- Check image: `podman images | grep <image-name>`
- Verify ports: `podman ps` and `netstat -tlnp | grep <port>`
- Validate paths: `ls -la <source-path>`

### Permission Denied in Container
**Symptoms**: Cannot write to mounted volumes, "Permission denied" errors
**Cause**: SELinux or user namespace mapping issues
**Fix**:
```bash
# For SELinux
podman run --security-opt label=disable <options>
# Or add :Z to mount
--mount type=bind,source=$PWD,destination=/app:Z

# For user mapping
podman unshare chown -R 0:0 /path/to/directory
```

### Slow File System Performance
**Symptoms**: File operations in container are significantly slower than host
**Cause**: Volume mount overhead, particularly on macOS
**Fix**:
- Use named volumes for frequently accessed data: `podman volume create fast-vol`
- Consider delegated mounts: `--mount type=bind,source=$PWD,destination=/app,bind-propagation=cached`
- Use container-local storage for temporary files

### Port Forwarding Not Working
**Symptoms**: Cannot access container ports from host
**Cause**: Port not exposed, firewall blocking, or service not binding to 0.0.0.0
**Fix**:
```bash
# Check container ports
podman port <container-name>

# Verify service binding
podman exec <container> netstat -tlnp

# Use host networking (less secure)
podman run --network host <options>
```

### Container Not Persisting Data
**Symptoms**: Data lost when container restarts
**Cause**: Data written to container filesystem instead of volumes
**Fix**:
- Mount persistent directories: `--mount type=bind,source=/host/path,destination=/container/path`
- Use named volumes: `--mount type=volume,source=data-vol,destination=/data`
- Check application configuration to write to mounted paths

## Authoritative References

- [Official Podman Documentation](https://docs.podman.io/en/latest/)
- [Podman Installation Guide](https://podman.io/docs/installation)
- [Podman Run Reference](https://docs.podman.io/en/latest/markdown/podman-run.1.html)
- [Container Storage Interface](https://docs.podman.io/en/latest/markdown/podman-volume.1.html)
- [Podman Machine Documentation](https://docs.podman.io/en/latest/markdown/podman-machine.1.html)
- [Red Hat Podman Guide](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/building_running_and_managing_containers/index)
- [Rootless Containers Documentation](https://github.com/containers/podman/blob/main/docs/tutorials/rootless_tutorial.md)