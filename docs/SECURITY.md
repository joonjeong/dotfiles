# Security Guide

Complete guide for secure dotfiles management with encryption and multi-device setup.

## Overview

Chezmoi provides robust security features for managing sensitive data:
- **Age encryption**: Modern, secure file encryption
- **Private files**: Automatic permission management
- **Secret separation**: Keep sensitive data separate from regular configs
- **Multi-device**: Secure synchronization across machines

## Encryption Setup

### Install age

**macOS:**
```bash
brew install age
```

**Linux:**
```bash
# Ubuntu/Debian
sudo apt install age

# Or download binary
wget https://github.com/FiloSottile/age/releases/latest/download/age-v1.1.1-linux-amd64.tar.gz
tar xzf age-v1.1.1-linux-amd64.tar.gz
sudo mv age/age* /usr/local/bin/
```

### Generate encryption key

```bash
# Create chezmoi config directory
mkdir -p ~/.config/chezmoi

# Generate age key pair
age-keygen -o ~/.config/chezmoi/key.txt

# Note the public key output (starts with age1...)
# Example: age1ql3z7hjy54pw3hyww5ayyfg7zqgvc7w3j2elw8zmrj2kg5sfn9aqmcac8p
```

**Important**: Save the public key - you'll need it for configuration.

### Configure chezmoi encryption

Edit chezmoi configuration:
```bash
chezmoi edit-config
```

Add encryption settings:
```toml
encryption = "age"

[age]
    identity = "~/.config/chezmoi/key.txt"
    recipient = "age1ql3z7hjy54pw3hyww5ayyfg7zqgvc7w3j2elw8zmrj2kg5sfn9aqmcac8p"
```

Replace the recipient with your actual public key from age-keygen.

## Managing Encrypted Files

### Add encrypted files

```bash
# Encrypt SSH private keys
chezmoi add --encrypt ~/.ssh/id_rsa
chezmoi add --encrypt ~/.ssh/id_ed25519

# Encrypt API token files
chezmoi add --encrypt ~/.config/api_tokens
chezmoi add --encrypt ~/.aws/credentials

# Encrypt entire directories
chezmoi add --encrypt ~/.ssh/
chezmoi add --encrypt ~/.gnupg/
```

### File naming conventions

Encrypted files get `.age` extension in source directory:

- `~/.ssh/id_rsa` → `private_dot_ssh_private_id_rsa.age`
- `~/.aws/credentials` → `private_dot_aws_credentials.age`
- `~/.gnupg/` → `private_dot_gnupg/` (entire directory encrypted)

### Edit encrypted files

```bash
# Edit encrypted file (decrypted temporarily)
chezmoi edit ~/.ssh/id_rsa

# View encrypted file content
chezmoi cat ~/.ssh/id_rsa
```

## Private Files (Permissions)

Files with restricted permissions (600, 700) are marked as private.

### Automatic private detection

Chezmoi automatically detects and marks files as private based on permissions:

```bash
# These become private_ automatically
chmod 600 ~/.ssh/id_rsa
chezmoi add ~/.ssh/id_rsa
# Creates: private_dot_ssh_private_id_rsa

chmod 700 ~/.ssh/
chezmoi add ~/.ssh/
# Creates: private_dot_ssh/ directory
```

### Manual private designation

```bash
# Force file to be private (600 permissions)
chezmoi add --private ~/.ssh/config

# Combines with encryption
chezmoi add --encrypt --private ~/.ssh/id_rsa
```

## Secure Configuration Templates

### Encrypted template data

Store sensitive template data in encrypted format:

`.chezmoi.toml.tmpl`:
```toml
{{- $hostname := .chezmoi.hostname -}}

[data]
    name = "Your Name"

{{- if eq $hostname "work-laptop" }}
    # Work configuration (could be in encrypted file)
    email = "{{ includeTemplate "encrypted_work_email.age" . }}"

    [data.work]
        api_key = "{{ includeTemplate "encrypted_work_api.age" . }}"
        server = "work-server.company.com"

{{- else }}
    email = "personal@example.com"
{{- end }}
```

### Environment-based secrets

Use environment variables for secrets:

```toml
[data]
    api_key = "{{ env "API_KEY" }}"
    db_password = "{{ env "DB_PASSWORD" }}"
```

Set environment variables securely:
```bash
export API_KEY="secret_value"
chezmoi apply
```

## Multi-Device Setup

### Initial setup on first machine

```bash
# Generate and configure encryption key
age-keygen -o ~/.config/chezmoi/key.txt

# Configure chezmoi with the public key
chezmoi edit-config
# Add encryption settings as shown above

# Add encrypted files
chezmoi add --encrypt ~/.ssh/id_rsa

# Commit to repository
chezmoi cd
git add .
git commit -m "Add encrypted SSH keys"
git push
```

### Setup on additional machines

```bash
# Clone dotfiles
chezmoi init https://github.com/username/dotfiles.git

# Securely transfer encryption key to new machine
# Option 1: Secure copy from existing machine
scp existing-machine:~/.config/chezmoi/key.txt ~/.config/chezmoi/

# Option 2: Use encrypted storage/password manager
# Option 3: Generate new key and re-encrypt (advanced)

# Apply dotfiles (will decrypt files)
chezmoi apply
```

### Key management strategies

**Single key (simpler):**
- Use same age key on all machines
- Transfer key securely between machines
- Keep backup of key in secure location

**Per-machine keys (more secure):**
- Generate unique key for each machine
- Add all public keys as recipients in chezmoi config
- Files encrypted for multiple recipients

Example multi-recipient config:
```toml
encryption = "age"

[age]
    identity = "~/.config/chezmoi/key.txt"
    recipients = [
        "age1work123...",     # Work laptop key
        "age1personal456...", # Personal laptop key
        "age1desktop789..."   # Desktop key
    ]
```

## Security Best Practices

### Key security

1. **Backup encryption keys securely**
   ```bash
   # Backup key to secure location
   cp ~/.config/chezmoi/key.txt /secure/backup/location/
   ```

2. **Use strong key storage**
   - Store keys in password manager
   - Use hardware security keys if possible
   - Never commit keys to git repository

3. **Key rotation**
   ```bash
   # Generate new key
   age-keygen -o ~/.config/chezmoi/key-new.txt

   # Re-encrypt files with new key
   # Update chezmoi config with new recipient
   # Remove old key after verification
   ```

### Repository security

1. **Use private repositories** for dotfiles containing encrypted data
2. **Review commits** before pushing to ensure no secrets leaked
3. **Enable branch protection** on main branch
4. **Use signed commits** for authenticity

### File organization

```
~/.local/share/chezmoi/
├── .chezmoi.toml.tmpl           # Main config (may contain some secrets)
├── dot_gitconfig.tmpl           # Public config template
├── private_dot_ssh/             # SSH directory (private)
│   ├── config                   # SSH config (readable)
│   ├── private_id_rsa.age      # Private key (encrypted)
│   ├── private_id_ed25519.age  # Private key (encrypted)
│   └── id_rsa.pub              # Public key (not encrypted)
├── private_dot_gnupg/           # GPG directory (private, encrypted)
│   ├── private_gpg-agent.conf.age
│   └── private_trustdb.gpg.age
└── dot_config/
    ├── git/
    │   └── config.tmpl
    └── encrypted_api_tokens.age # Encrypted API tokens
```

## Working with Secrets

### API tokens and credentials

Store in dedicated encrypted files:

```bash
# Create API tokens file
echo 'GITHUB_TOKEN=ghp_xxxxxxxxxxxx
AWS_ACCESS_KEY_ID=AKIAXXXXXXXX
AWS_SECRET_ACCESS_KEY=xxxxxxxx' > ~/.config/api_tokens

# Add encrypted
chezmoi add --encrypt ~/.config/api_tokens
```

Use in templates:
```bash
# dot_bashrc.tmpl
{{- $tokens := includeTemplate "encrypted_api_tokens.age" . | fromIni -}}
export GITHUB_TOKEN="{{ $tokens.GITHUB_TOKEN }}"
export AWS_ACCESS_KEY_ID="{{ $tokens.AWS_ACCESS_KEY_ID }}"
```

### Database passwords

```bash
# Store in template data
# .chezmoi.toml.tmpl
[data.database]
    host = "db.example.com"
    username = "user"
    password = "{{ env "DB_PASSWORD" }}"
```

### SSH key management

```bash
# Add all SSH keys encrypted
chezmoi add --encrypt ~/.ssh/id_rsa
chezmoi add --encrypt ~/.ssh/id_ed25519

# SSH config template for dynamic host management
# private_dot_ssh_config.tmpl
Host github.com
    User git
    IdentityFile ~/.ssh/id_ed25519
    IdentitiesOnly yes

{{- if eq .chezmoi.hostname "work-laptop" }}
Host *.company.com
    User {{ .work.username }}
    IdentityFile ~/.ssh/id_rsa
    ProxyCommand nc -X connect -x {{ .proxy.host }}:{{ .proxy.port }} %h %p
{{- end }}
```

## Troubleshooting Security

### Age encryption issues

```bash
# Test age key
age -d -i ~/.config/chezmoi/key.txt encrypted_file.age

# Verify public key matches
age-keygen -y ~/.config/chezmoi/key.txt

# Check chezmoi encryption status
chezmoi status | grep -E "(encrypted|age)"
```

### Permission issues

```bash
# Fix SSH permissions
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_*
chmod 644 ~/.ssh/*.pub

# Check chezmoi private file handling
chezmoi diff | grep -E "(private|600|700)"
```

### Key recovery

If you lose your encryption key:

1. **From backup**: Restore key from secure backup
2. **From another machine**: Copy key from working machine
3. **Re-encrypt**: Generate new key and re-add all encrypted files

```bash
# Re-encrypt with new key (if old key lost)
# Remove encrypted files from chezmoi
chezmoi forget ~/.ssh/id_rsa

# Generate new key
age-keygen -o ~/.config/chezmoi/key-new.txt

# Update chezmoi config with new recipient
# Re-add files with new encryption
chezmoi add --encrypt ~/.ssh/id_rsa
```

## Backup and Recovery

### Regular backups

```bash
# Backup encryption key
cp ~/.config/chezmoi/key.txt ~/secure-backup/

# Export all dotfiles (including encrypted)
chezmoi archive > dotfiles-backup-$(date +%Y%m%d).tar.gz

# Backup git repository
chezmoi cd
git bundle create dotfiles-backup.bundle HEAD main
```

### Recovery procedure

```bash
# New machine recovery
# 1. Install chezmoi and age
# 2. Restore encryption key
mkdir -p ~/.config/chezmoi
cp /backup/location/key.txt ~/.config/chezmoi/

# 3. Clone and apply dotfiles
chezmoi init --apply https://github.com/username/dotfiles.git

# 4. Verify encrypted files
chezmoi verify
```

## Related

- **[Setup Guide](SETUP.md)**: Initial chezmoi setup
- **[Templates](TEMPLATES.md)**: Using templates with encrypted data
- **[Troubleshooting](TROUBLESHOOTING.md)**: Security-related issues