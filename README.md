# Personal Dotfiles

Personal system configuration files managed with [chezmoi](https://www.chezmoi.io/).

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
├── README.md                 # This file
├── CLAUDE.md                 # Claude Code assistant instructions
├── docs/                     # Documentation
│   ├── SETUP.md             # Installation and setup guide
│   ├── TEMPLATES.md         # Template system usage
│   ├── SECURITY.md          # Encryption and security
│   └── TROUBLESHOOTING.md   # Common issues and solutions
├── .chezmoi.toml.tmpl       # Configuration template
├── dot_gitconfig.tmpl       # Git configuration template
├── dot_bashrc               # Bash configuration
├── private_dot_ssh/         # SSH keys (encrypted)
└── dot_config/              # Application configs
    ├── nvim/                # Neovim configuration
    └── ...
```

## Key Features

- **Multi-device support**: Different configurations for work/personal machines
- **Template system**: Dynamic configuration based on hostname, OS, etc.
- **Encryption**: Secure storage of SSH keys and sensitive data using age
- **Automated setup**: Scripts for installing packages and initial setup

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

The main configuration is in `.chezmoi.toml.tmpl` which provides:

- Host-specific variables
- OS-specific settings
- User data (name, email, etc.)
- Encrypted data handling

## Security

Sensitive files are encrypted using [age](https://github.com/FiloSottile/age):

- SSH private keys
- API tokens
- Personal certificates

See [Security Documentation](docs/SECURITY.md) for setup details.

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