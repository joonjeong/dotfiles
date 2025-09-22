# Setup Guide

Complete setup guide for chezmoi dotfiles management.

## Prerequisites

### Install chezmoi

**macOS:**
```bash
brew install chezmoi
```

**Linux:**
```bash
# Using snap
sudo snap install chezmoi --classic

# Or direct download
sh -c "$(curl -fsLS get.chezmoi.io)"
```

### Install age (for encryption)

**macOS:**
```bash
brew install age
```

**Linux:**
```bash
# Ubuntu/Debian
sudo apt update && sudo apt install age

# Or download from GitHub releases
wget https://github.com/FiloSottile/age/releases/download/v1.1.1/age-v1.1.1-linux-amd64.tar.gz
```

## Initial Setup

### Option 1: New dotfiles repository

```bash
# Initialize new repository
chezmoi init

# Add your first file
chezmoi add ~/.bashrc

# View the source directory
chezmoi cd
```

### Option 2: Clone existing repository

```bash
# Clone and apply existing dotfiles
chezmoi init --apply https://github.com/username/dotfiles.git

# If you have encryption setup, you may need to configure keys first
```

## Configuration Setup

### Create main config file

```bash
chezmoi edit-config
```

Add basic configuration:

```toml
[data]
    name = "Your Name"
    email = "your.email@example.com"

[git]
    autoPush = true
```

### Host-specific configuration

For different configurations per machine:

```bash
# Edit host-specific config
chezmoi edit ~/.config/chezmoi/chezmoi.toml
```

Example host-specific config:
```toml
[data]
    name = "Your Name"

[data.work]
    email = "work@company.com"

[data.personal]
    email = "personal@gmail.com"
```

## Encryption Setup

### Generate age key

```bash
# Create age key for encryption
age-keygen -o ~/.config/chezmoi/key.txt

# Note the public key output (age1...)
```

### Configure encryption in chezmoi

```bash
# Edit chezmoi config
chezmoi edit-config
```

Add encryption settings:
```toml
encryption = "age"

[age]
    identity = "~/.config/chezmoi/key.txt"
    recipient = "age1ql3z7hjy54pw3hyww5ayyfg7zqgvc7w3j2elw8zmrj2kg5sfn9aqmcac8p"
```

Replace the recipient with your actual public key from age-keygen output.

## Adding Files

### Basic files

```bash
# Add configuration files
chezmoi add ~/.bashrc
chezmoi add ~/.gitconfig
chezmoi add ~/.vimrc

# Add entire directories
chezmoi add ~/.config/nvim/
```

### Encrypted files

```bash
# Add sensitive files with encryption
chezmoi add --encrypt ~/.ssh/id_rsa
chezmoi add --encrypt ~/.config/api_tokens

# Add entire sensitive directories
chezmoi add --encrypt ~/.ssh/
```

### Template files

For files that need dynamic content:

```bash
# Add as template for dynamic content
chezmoi add --template ~/.gitconfig
```

## First Commit

```bash
# Go to source directory
chezmoi cd

# Initialize git if not already done
git init
git remote add origin https://github.com/username/dotfiles.git

# Commit and push
git add .
git commit -m "Initial dotfiles commit"
git push -u origin main
```

## Directory Structure

After setup, your source directory (`~/.local/share/chezmoi/`) will look like:

```
.local/share/chezmoi/
├── .chezmoi.toml.tmpl           # Main config template
├── .git/                        # Git repository
├── dot_bashrc                   # ~/.bashrc
├── dot_gitconfig.tmpl           # ~/.gitconfig (template)
├── dot_vimrc                    # ~/.vimrc
├── private_dot_ssh/             # ~/.ssh/ (encrypted)
│   ├── private_id_rsa.age      # SSH private key (encrypted)
│   └── id_rsa.pub              # SSH public key
└── dot_config/                  # ~/.config/
    └── nvim/                    # Neovim configuration
        ├── init.lua
        └── lua/
```

## File Naming Conventions

- `dot_` → `.` (hidden files)
- `private_` → file with 600 permissions
- `executable_` → file with execute permissions
- `.tmpl` → template file (processed)
- `.age` → encrypted file

Examples:
- `dot_bashrc` → `~/.bashrc`
- `private_dot_ssh_config` → `~/.ssh/config` (600 permissions)
- `dot_gitconfig.tmpl` → `~/.gitconfig` (processed as template)

## Verification

```bash
# Check what chezmoi will do
chezmoi diff

# Check current status
chezmoi status

# View managed files
chezmoi managed

# Test configuration
chezmoi data
```

## Next Steps

1. **[Templates](TEMPLATES.md)**: Learn to use templates for dynamic configs
2. **[Security](SECURITY.md)**: Setup encryption for sensitive files
3. **[Troubleshooting](TROUBLESHOOTING.md)**: Common issues and solutions

## Common Post-Setup Tasks

### Setup automated backups

```bash
# Add to crontab or use a script
chezmoi cd && git push
```

### Configure multiple machines

```bash
# On second machine
chezmoi init --apply https://github.com/username/dotfiles.git

# Copy encryption key securely to new machine
scp ~/.config/chezmoi/key.txt newmachine:~/.config/chezmoi/
```

### Regular maintenance

```bash
# Update dotfiles regularly
chezmoi update

# Or manually
chezmoi git pull
chezmoi apply
```