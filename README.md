# Personal Dotfiles

Personal system configuration files managed with [chezmoi](https://www.chezmoi.io/) with multi-environment support and encryption.

## Quick Start

```bash
# Install dotfiles on new machine
chezmoi init --apply https://github.com/username/dotfiles.git

# Daily usage
chezmoi diff      # Check changes
chezmoi apply     # Apply changes
chezmoi update    # Update from remote and apply
```

## What's Managed

- Shell configuration (bash, zsh)
- Git configuration
- SSH keys (encrypted)
- Application configs
- Development tools setup

## Repository Structure

```
~/.local/share/chezmoi/
├── README.md                 # This file (ignored by apply)
├── CLAUDE.md                 # Claude Code assistant instructions
├── .chezmoiignore           # Files to exclude from home directory
├── .chezmoi.toml.tmpl       # Main configuration template
├── docs/                     # Documentation (ignored by apply)
│   ├── SETUP.md             # Installation and setup guide
│   ├── TEMPLATES.md         # Template system usage
│   ├── SECURITY.md          # Encryption and security
│   └── TROUBLESHOOTING.md   # Common issues and solutions
├── dot_gitconfig.tmpl       # Git configuration template
├── dot_bashrc               # Bash configuration
├── private_dot_ssh/         # SSH keys (encrypted)
│   ├── private_id_rsa.age  # Encrypted SSH key
│   └── config.tmpl         # SSH config template
└── dot_config/              # Application configs
    ├── nvim/               # Neovim configuration
    └── ...
```

## Key Features

- **Multi-environment**: Auto-detection of work/personal machines
- **Templates**: Dynamic configs with OS detection (macOS/Linux/Windows)
- **Encryption**: Age encryption for SSH keys and sensitive data
- **Documentation**: Setup, templates, security, and troubleshooting guides

## Documentation

- **[Setup Guide](docs/SETUP.md)**: Detailed installation and initial setup
- **[Templates](docs/TEMPLATES.md)**: Using the template system for dynamic configs
- **[Security](docs/SECURITY.md)**: Encryption setup and best practices
- **[Troubleshooting](docs/TROUBLESHOOTING.md)**: Common issues and solutions

## Common Commands

```bash
# Add new file to management
chezmoi add ~/.vimrc

# Edit managed file
chezmoi edit ~/.bashrc

# Apply changes
chezmoi apply

# View source directory
chezmoi cd

# Check status
chezmoi status

# Update from remote
chezmoi update
```

## Configuration

`.chezmoi.toml.tmpl` provides:
- Environment detection (work/personal)
- OS-specific settings and package managers
- User data with environment overrides
- Package lists and proxy settings

## Security

Sensitive files encrypted with [age](https://github.com/FiloSottile/age):
- SSH private keys (`.age` extension)
- API tokens and certificates
- Multi-device key management

See [docs/SECURITY.md](docs/SECURITY.md) for setup details.

## Prerequisites

- [chezmoi](https://www.chezmoi.io/install/)
- [age](https://github.com/FiloSottile/age) (for encryption)
- Git

## Installation

```bash
# macOS
brew install chezmoi age

# Linux
sh -c "$(curl -fsLS get.chezmoi.io)"
```

See [Setup Guide](docs/SETUP.md) for detailed instructions.

## File Management

**Ignored files** (`.chezmoiignore`):
- `docs/`, `README.md`, `CLAUDE.md`, `.git/`

**Template examples**:
- Git config with work/personal switching
- SSH config with environment-specific settings
- Shell config with OS-specific aliases

## Quick Reference

```bash
chezmoi status          # Check changes
chezmoi cat ~/.gitconfig # View template output
chezmoi data            # Debug template data
```