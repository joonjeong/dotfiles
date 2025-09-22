# Troubleshooting Guide

Common issues and solutions for chezmoi dotfiles management.

## Installation Issues

### Chezmoi command not found

**Problem**: `command not found: chezmoi`

**Solutions**:

```bash
# macOS - Install via Homebrew
brew install chezmoi

# Linux - Install via script
sh -c "$(curl -fsLS get.chezmoi.io)"

# Verify PATH includes chezmoi
which chezmoi
echo $PATH
```

### Age not available

**Problem**: `age: command not found`

**Solutions**:

```bash
# macOS
brew install age

# Ubuntu/Debian
sudo apt update && sudo apt install age

# Manual installation
wget https://github.com/FiloSottile/age/releases/latest/download/age-v1.1.1-linux-amd64.tar.gz
tar xzf age-v1.1.1-linux-amd64.tar.gz
sudo mv age/age* /usr/local/bin/
```

## Configuration Issues

### Template parsing errors

**Problem**: `template: parse error`

**Debug steps**:

```bash
# Check template syntax
chezmoi execute-template < dot_file.tmpl

# View data available to templates
chezmoi data

# Test specific template parts
echo '{{ .chezmoi.hostname }}' | chezmoi execute-template
```

**Common syntax issues**:

```go
// Wrong - missing spaces around pipes
{{.name|upper}}

// Correct - spaces around pipes
{{ .name | upper }}

// Wrong - undefined variable
{{ .undefined_var }}

// Correct - with default
{{ .undefined_var | default "fallback" }}
```

### Variables not defined

**Problem**: `undefined variable: .work`

**Solutions**:

1. **Check data definition** in `.chezmoi.toml.tmpl`:
   ```toml
   [data]
   [data.work]  # Make sure section exists
       email = "work@company.com"
   ```

2. **Use conditional access**:
   ```go
   {{- if .work }}
   {{- if .work.email }}
   Email: {{ .work.email }}
   {{- end }}
   {{- end }}
   ```

3. **Use default values**:
   ```go
   {{ .work.email | default "no-email" }}
   ```

### Hostname detection issues

**Problem**: Wrong hostname or hostname-based conditions not working

**Debug**:
```bash
# Check detected hostname
chezmoi data | grep hostname

# Override hostname for testing
chezmoi apply --force --config <(echo '[data]\nhostname = "test-hostname"')
```

**Solutions**:
```toml
# .chezmoi.toml.tmpl - override hostname detection
[data]
    hostname = "{{ env "CHEZMOI_HOSTNAME" | default .chezmoi.hostname }}"
```

## Encryption Issues

### Age decryption failed

**Problem**: `age: error: failed to decrypt`

**Solutions**:

1. **Check identity file exists**:
   ```bash
   ls -la ~/.config/chezmoi/key.txt
   ```

2. **Verify identity file format**:
   ```bash
   head -1 ~/.config/chezmoi/key.txt
   # Should start with: # created: 2024-xx-xx
   # Second line should start with: AGE-SECRET-KEY-
   ```

3. **Test age directly**:
   ```bash
   echo "test" | age -e -R ~/.config/chezmoi/key.txt | age -d -i ~/.config/chezmoi/key.txt
   ```

4. **Check recipient configuration**:
   ```bash
   # Get public key from private key
   age-keygen -y ~/.config/chezmoi/key.txt

   # Compare with recipient in chezmoi config
   chezmoi data | grep recipient
   ```

### Wrong recipient key

**Problem**: File encrypted for different recipient

**Solutions**:

```bash
# Re-encrypt with correct key
chezmoi forget ~/.ssh/id_rsa
chezmoi add --encrypt ~/.ssh/id_rsa

# Or manually decrypt and re-encrypt
chezmoi cat ~/.ssh/id_rsa > /tmp/id_rsa
age -e -R ~/.config/chezmoi/key.txt /tmp/id_rsa > ~/.local/share/chezmoi/private_dot_ssh_private_id_rsa.age
rm /tmp/id_rsa
```

### Missing age identity

**Problem**: `no age identity specified`

**Solutions**:

1. **Set environment variable**:
   ```bash
   export CHEZMOI_AGE_IDENTITY=~/.config/chezmoi/key.txt
   ```

2. **Fix chezmoi config**:
   ```toml
   encryption = "age"
   [age]
       identity = "~/.config/chezmoi/key.txt"
       recipient = "age1..."
   ```

## File Management Issues

### Permission denied

**Problem**: `permission denied` when applying files

**Solutions**:

```bash
# Check file ownership
ls -la ~/.ssh/

# Fix ownership
sudo chown -R $USER:$USER ~/.ssh/

# Fix permissions
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_*
chmod 644 ~/.ssh/*.pub
```

### Files not being applied

**Problem**: `chezmoi apply` does nothing

**Debug steps**:

```bash
# Check what would be changed
chezmoi diff

# Check status
chezmoi status

# Check what's managed
chezmoi managed

# Force application
chezmoi apply --force
```

### Template not processing

**Problem**: Template syntax visible in output files

**Causes**:
- File not added as template
- Template syntax errors
- Missing `.tmpl` extension

**Solutions**:

```bash
# Check if file is template
chezmoi managed | grep -E "\.tmpl$"

# Convert to template
chezmoi chattr +template ~/.gitconfig

# Or re-add as template
chezmoi forget ~/.gitconfig
chezmoi add --template ~/.gitconfig
```

## Git Repository Issues

### Remote repository issues

**Problem**: `git push` fails or remote not set

**Solutions**:

```bash
# Check remote configuration
chezmoi cd
git remote -v

# Add remote if missing
git remote add origin https://github.com/username/dotfiles.git

# Fix authentication
git config --global credential.helper store
```

### Merge conflicts

**Problem**: Git merge conflicts when updating

**Solutions**:

```bash
# Check conflicts
chezmoi cd
git status

# Resolve manually
git mergetool

# Or reset to remote
git reset --hard origin/main
exit
chezmoi apply
```

### Large binary files

**Problem**: Repository too large due to binary files

**Solutions**:

```bash
# Remove binary files from git history
chezmoi cd
git filter-branch --tree-filter 'rm -f large-file.bin' HEAD

# Use .chezmoiignore for future binaries
echo "*.bin" >> .chezmoiignore
echo "*.dmg" >> .chezmoiignore
```

## Performance Issues

### Slow apply operations

**Problem**: `chezmoi apply` takes too long

**Solutions**:

```bash
# Use dry run to identify slow operations
time chezmoi apply --dry-run

# Apply specific files only
chezmoi apply ~/.bashrc ~/.gitconfig

# Check for large template processing
chezmoi execute-template --init < .chezmoi.toml.tmpl
```

### Large source directory

**Problem**: Source directory growing too large

**Solutions**:

```bash
# Check directory size
du -sh ~/.local/share/chezmoi/

# Remove unnecessary files
echo "node_modules/" >> .chezmoiignore
echo ".git/" >> .chezmoiignore
echo "*.log" >> .chezmoiignore

# Clean up removed files
chezmoi cd
git gc --aggressive
```

## Cross-Platform Issues

### Line ending problems

**Problem**: CRLF/LF conversion issues

**Solutions**:

```bash
# Configure git line endings
git config --global core.autocrlf input  # macOS/Linux
git config --global core.autocrlf true   # Windows

# Fix existing files
chezmoi cd
git config core.autocrlf false
git rm --cached -r .
git reset --hard
```

### Path separator issues

**Problem**: Windows path issues in templates

**Solutions**:

```go
{{- if eq .chezmoi.os "windows" }}
set PATH=%PATH%;C:\tools
{{- else }}
export PATH="$PATH:/usr/local/bin"
{{- end }}
```

### Unicode handling

**Problem**: Special characters not displaying correctly

**Solutions**:

```bash
# Check locale settings
locale

# Set UTF-8 encoding
export LANG=en_US.UTF-8
export LC_ALL=en_US.UTF-8
```

## Debug Mode

### Enable verbose output

```bash
# Verbose mode
chezmoi -v apply

# Debug mode
chezmoi --debug apply

# Dry run with verbose
chezmoi apply --dry-run --verbose
```

### Template debugging

```bash
# Execute template with debug info
chezmoi execute-template --init --debug < .chezmoi.toml.tmpl

# Check available template data
chezmoi data --debug

# Test template syntax
echo '{{ .chezmoi | toJson }}' | chezmoi execute-template
```

## Recovery Procedures

### Reset to clean state

```bash
# Backup current state
chezmoi archive > backup-$(date +%Y%m%d).tar.gz

# Remove all managed files
chezmoi managed | xargs rm -f

# Re-apply from source
chezmoi apply --force
```

### Restore from backup

```bash
# Extract backup
tar xzf backup-20241219.tar.gz

# Copy back to home directory
cp -r backup/* ~/

# Re-initialize chezmoi
chezmoi init --apply
```

### Emergency procedures

If chezmoi is completely broken:

```bash
# Manual file extraction
cd ~/.local/share/chezmoi
find . -name "dot_*" -type f | while read file; do
    target=$(echo "$file" | sed 's/^dot_/~\/./')
    cp "$file" "$target"
done

# Fix SSH permissions
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_*
```

## Getting Help

### Chezmoi diagnostics

```bash
# System information
chezmoi doctor

# Configuration dump
chezmoi dump

# Version information
chezmoi --version
```

### Log files

```bash
# Check system logs
journalctl --user -u chezmoi

# macOS console logs
log show --predicate 'process == "chezmoi"' --last 1h
```

### Community resources

- [Chezmoi GitHub Issues](https://github.com/twpayne/chezmoi/issues)
- [Chezmoi Discussions](https://github.com/twpayne/chezmoi/discussions)
- [Chezmoi Documentation](https://www.chezmoi.io/)

## Preventive Measures

### Regular maintenance

```bash
# Weekly: Update and check status
chezmoi update
chezmoi status

# Monthly: Clean up and backup
chezmoi cd
git gc
git push

# Create backup
chezmoi archive > monthly-backup-$(date +%Y%m).tar.gz
```

### Configuration validation

```bash
# Test configuration before applying
chezmoi diff | less
chezmoi apply --dry-run

# Validate templates
find ~/.local/share/chezmoi -name "*.tmpl" -exec chezmoi execute-template {} \;
```

### Monitoring

```bash
# Check for issues
chezmoi doctor
chezmoi verify

# Monitor file changes
chezmoi status
git status --porcelain
```

## Related

- **[Setup Guide](SETUP.md)**: Initial setup procedures
- **[Templates](TEMPLATES.md)**: Template syntax and debugging
- **[Security](SECURITY.md)**: Encryption troubleshooting