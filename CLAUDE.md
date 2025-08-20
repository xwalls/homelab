# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a GitOps-based homelab infrastructure project that automates the deployment and management of a complete Kubernetes cluster and applications. The project uses Infrastructure as Code to provision bare metal servers, install K3s, and deploy applications through ArgoCD.

## Architecture

The project is organized into layered components that build upon each other:

```
./apps     - User-facing applications (Jellyfin, Paperless, Matrix, etc.)
./platform - Service hosting platform (Gitea, Grafana, Woodpecker CI)
./system   - Core cluster components (ArgoCD, Rook Ceph, NGINX, cert-manager)
./metal    - Bare metal provisioning and K3s cluster setup
./external - External services (Cloudflare, Terraform resources)
./docs     - MkDocs documentation
./scripts  - Automation and helper scripts
./test     - Go-based integration tests with Terratest
```

The provisioning flow is: metal → system → platform → apps, with ArgoCD taking over after the system layer is bootstrapped.

## Development Environment

Enter the Nix development shell before working:
```bash
nix develop
```

This provides all required tools including Ansible, OpenTofu, kubectl, Helm, Go, and more defined in `flake.nix`.

## Common Commands

### Main Deployment
```bash
make                    # Full deployment: metal → system → external → smoke test
make configure          # Interactive configuration setup
```

### Component-specific Operations
```bash
make metal             # Provision bare metal and install K3s
make system            # Bootstrap system components via ArgoCD
make external          # Deploy external resources (Terraform)
make test              # Run all integration tests
make smoke-test        # Run smoke tests only
```

### Development and Maintenance
```bash
make docs              # Start MkDocs development server
make git-hooks         # Install pre-commit hooks
make backup            # Setup backups for persistent data
make restore           # Restore from backups
make clean             # Clean up PXE server containers
```

### Testing
```bash
cd test/
make                   # Run all tests with gotestsum
make filter=Smoke      # Run smoke tests only
make filter=<pattern>  # Run tests matching pattern
```

### Environment-specific Commands
```bash
# Metal layer (bare metal provisioning)
cd metal/
make env=prod          # Deploy to production inventory
make env=stag          # Deploy to staging inventory
make boot              # PXE boot and OS installation
make cluster           # K3s cluster setup only

# External resources
cd external/
make plan              # Plan Terraform changes
make apply             # Apply Terraform changes
```

## Key Configuration Files

- `metal/group_vars/all.yml` - Ansible global variables
- `metal/inventories/prod.yml` - Production server inventory with IPs, MACs, disk info
- `external/terraform.tfvars` - External services configuration (Cloudflare, ntfy)
- `apps/*/values.yaml` - Application-specific Helm values
- `platform/*/values.yaml` - Platform service configurations
- `scripts/configure` - Python script for initial setup

## Configuration Process

The `make configure` command runs `scripts/configure` which:
1. Prompts for domain, timezone, IP ranges, and Git repository
2. Performs find-and-replace across the codebase
3. Opens editor for server inventory configuration
4. Handles optional external services setup

## Development Workflow

1. **Setup**: Run `make configure` for initial configuration
2. **Development Shell**: Use `nix develop` for consistent environment
3. **Code Quality**: Pre-commit hooks enforce YAML linting, shellcheck, and Helm validation
4. **Testing**: Integration tests validate cluster functionality using Terratest
5. **GitOps**: Changes are applied through ArgoCD after initial bootstrap

## Scripts Directory

Utility scripts for common operations:
- `configure` - Interactive configuration setup
- `backup`/`restore` - Data backup management with namespace/PVC targeting
- `get-status` - Cluster health checking
- `new-service` - Template generator for adding services
- `onboard-user` - User management automation
- `kanidm-reset-password` - Identity management

## Technology Stack

- **Provisioning**: Ansible for automation, PXE boot for OS installation
- **Kubernetes**: K3s with Cilium CNI
- **GitOps**: ArgoCD for application deployment
- **Storage**: Rook Ceph for distributed storage
- **Networking**: NGINX Ingress, External DNS, Cloudflare Tunnel
- **Monitoring**: Grafana, Prometheus, Loki stack
- **Identity**: Kanidm for SSO
- **CI/CD**: Woodpecker CI
- **Testing**: Go with Terratest for integration testing

## Important Notes

- Project is in ALPHA stage - expect breaking changes
- KUBECONFIG automatically set to `metal/kubeconfig.yaml`
- Server inventory requires IP addresses, MAC addresses, disk names, and network interfaces
- All applications use Helm charts, primarily bjw-s app-template
- Secret management uses External Secrets Operator
- Network policies enforced via Cilium
- Pre-commit hooks ensure code quality and security