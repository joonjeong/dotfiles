# Template System Guide

Learn how to use chezmoi's template system for dynamic configuration files.

## What are Templates?

Templates allow you to create configuration files that adapt based on:
- Operating system (macOS, Linux, Windows)
- Hostname (work laptop, personal machine)
- User data (name, email, preferences)
- Environment variables

## Creating Templates

### Add template files

```bash
# Add file as template
chezmoi add --template ~/.gitconfig

# File becomes dot_gitconfig.tmpl in source directory
```

### Template file naming

- Regular file: `dot_gitconfig` → `~/.gitconfig`
- Template file: `dot_gitconfig.tmpl` → `~/.gitconfig` (processed)

## Template Syntax

Uses Go template syntax with predefined variables.

### Basic variables

```go
{{ .chezmoi.hostname }}        // Computer hostname
{{ .chezmoi.username }}        // Current username
{{ .chezmoi.os }}              // Operating system (darwin, linux, windows)
{{ .chezmoi.arch }}            // CPU architecture (amd64, arm64)
{{ .chezmoi.homeDir }}         // Home directory path
```

### Custom data variables

Define in `.chezmoi.toml.tmpl`:

```toml
[data]
    name = "Your Name"
    email = "your@email.com"

[data.work]
    email = "work@company.com"

[data.personal]
    email = "personal@gmail.com"
```

Use in templates:
```go
{{ .name }}                    // "Your Name"
{{ .work.email }}              // "work@company.com"
```

## Template Examples

### Git configuration template

`dot_gitconfig.tmpl`:
```bash
[user]
    name = {{ .name }}
{{- if eq .chezmoi.hostname "work-laptop" }}
    email = {{ .work.email }}
    signingkey = {{ .work.signing_key }}
{{- else }}
    email = {{ .personal.email }}
    signingkey = {{ .personal.signing_key }}
{{- end }}

[core]
    editor = nvim
{{- if eq .chezmoi.os "darwin" }}
    autocrlf = input
{{- else if eq .chezmoi.os "windows" }}
    autocrlf = true
{{- end }}
```

### Shell configuration template

`dot_bashrc.tmpl`:
```bash
# Common aliases
alias ll='ls -la'
alias la='ls -A'

{{- if eq .chezmoi.os "darwin" }}
# macOS specific
alias ls='ls -G'
export HOMEBREW_PREFIX="/opt/homebrew"
{{- else if eq .chezmoi.os "linux" }}
# Linux specific
alias ls='ls --color=auto'
export HOMEBREW_PREFIX="/home/linuxbrew/.linuxbrew"
{{- end }}

{{- if eq .chezmoi.hostname "work-laptop" }}
# Work environment
export HTTP_PROXY="{{ .proxy.http }}"
export HTTPS_PROXY="{{ .proxy.https }}"
{{- end }}

# User specific
export USER_NAME="{{ .name }}"
export USER_EMAIL="{{ .email }}"
```

### SSH config template

`private_dot_ssh_config.tmpl`:
```bash
# Common settings
Host *
    ServerAliveInterval 60
    ServerAliveCountMax 3

{{- if eq .chezmoi.hostname "work-laptop" }}
# Work servers
Host work-server
    HostName {{ .work.server_host }}
    User {{ .work.username }}
    IdentityFile ~/.ssh/work_rsa

Host *.company.com
    ProxyCommand nc -X connect -x {{ .proxy.host }}:{{ .proxy.port }} %h %p
{{- end }}

{{- if .personal.github_username }}
# Personal GitHub
Host github.com
    User git
    IdentityFile ~/.ssh/id_rsa
    IdentitiesOnly yes
{{- end }}
```

## Control Structures

### If statements

```go
{{- if eq .chezmoi.os "darwin" }}
macOS specific content
{{- else if eq .chezmoi.os "linux" }}
Linux specific content
{{- else }}
Other OS content
{{- end }}
```

### Loops

```go
{{- range .packages }}
install {{ . }}
{{- end }}
```

### Variables

```go
{{- $is_work := eq .chezmoi.hostname "work-laptop" }}
{{- if $is_work }}
Work configuration
{{- end }}
```

## Data Sources

### Environment variables

```go
{{ env "HOME" }}               // Environment variable
{{ env "API_KEY" | default "none" }}  // With default value
```

### Command output

```go
{{ output "hostname" }}        // Command output
{{ output "git" "config" "user.name" }}  // Git command
```

### File contents

```go
{{ include "file.txt" }}       // Include file contents
```

## Configuration Template

`.chezmoi.toml.tmpl` - Main configuration template:

```toml
{{- $hostname := .chezmoi.hostname -}}
{{- $is_work := eq $hostname "work-laptop" -}}
{{- $is_personal := eq $hostname "personal-mac" -}}

[data]
    name = "Your Name"

{{- if $is_work }}
    email = "work@company.com"
    signing_key = "WORK_KEY_ID"

    [data.proxy]
        http = "http://proxy.company.com:8080"
        https = "http://proxy.company.com:8080"
        host = "proxy.company.com"
        port = 8080

    [data.work]
        username = "work_user"
        server_host = "work-server.company.com"

{{- else if $is_personal }}
    email = "personal@gmail.com"
    signing_key = "PERSONAL_KEY_ID"

    [data.personal]
        github_username = "your_github"

{{- end }}

{{- if eq .chezmoi.os "darwin" }}
    [data.homebrew]
        prefix = "/opt/homebrew"
{{- else if eq .chezmoi.os "linux" }}
    [data.homebrew]
        prefix = "/home/linuxbrew/.linuxbrew"
{{- end }}

[data.packages]
    common = ["git", "nvim", "tmux"]
{{- if $is_work }}
    work = ["docker", "kubectl", "terraform"]
{{- end }}
```

## Template Functions

### String functions

```go
{{ .name | upper }}            // UPPERCASE
{{ .name | lower }}            // lowercase
{{ .email | quote }}           // "quoted"
```

### Comparison functions

```go
{{ eq .chezmoi.os "darwin" }}  // Equals
{{ ne .chezmoi.os "windows" }} // Not equals
{{ and (eq .os "linux") .is_work }}  // AND logic
{{ or (eq .os "darwin") (eq .os "linux") }}  // OR logic
```

## Testing Templates

### View processed output

```bash
# See what template produces
chezmoi cat ~/.gitconfig

# Execute template directly
chezmoi execute-template < dot_gitconfig.tmpl
```

### Debug variables

```bash
# View all available data
chezmoi data

# View specific template data
chezmoi execute-template '{{ .chezmoi | toJson }}'
```

### Dry run

```bash
# See what would be applied
chezmoi diff
chezmoi apply --dry-run
```

## Advanced Examples

### Conditional file inclusion

Use multiple template files for different scenarios:

```
dot_bashrc.tmpl              # Base configuration
dot_bashrc_work.tmpl         # Work-specific additions
dot_bashrc_personal.tmpl     # Personal additions
```

In main template:
```bash
# Base configuration
source ~/.bashrc_base

{{- if eq .chezmoi.hostname "work-laptop" }}
source ~/.bashrc_work
{{- else }}
source ~/.bashrc_personal
{{- end }}
```

### Dynamic package lists

In `.chezmoi.toml.tmpl`:
```toml
[data.packages]
    base = ["git", "nvim", "tmux"]

{{- if eq .chezmoi.os "darwin" }}
    os_specific = ["brew", "cask"]
{{- else if eq .chezmoi.os "linux" }}
    os_specific = ["apt", "snap"]
{{- end }}

{{- if eq .chezmoi.hostname "work-laptop" }}
    work = ["docker", "kubectl"]
{{- end }}
```

In script template:
```bash
#!/bin/bash

# Install base packages
{{- range .packages.base }}
install_package {{ . }}
{{- end }}

# Install OS-specific packages
{{- range .packages.os_specific }}
install_package {{ . }}
{{- end }}

{{- if .packages.work }}
# Install work packages
{{- range .packages.work }}
install_package {{ . }}
{{- end }}
{{- end }}
```

## Best Practices

1. **Start simple**: Begin with basic if/else conditions
2. **Use variables**: Define reusable variables for complex conditions
3. **Test thoroughly**: Use `chezmoi cat` to verify template output
4. **Comment templates**: Document complex logic
5. **Validate data**: Use defaults for optional values
6. **Keep organized**: Group related configuration in data sections

## Common Patterns

### Host detection
```go
{{- $hostname := .chezmoi.hostname -}}
{{- $is_work := or (eq $hostname "work-laptop") (eq $hostname "work-desktop") -}}
```

### OS detection with fallback
```go
{{- if eq .chezmoi.os "darwin" -}}
macOS config
{{- else if eq .chezmoi.os "linux" -}}
Linux config
{{- else -}}
Generic config
{{- end -}}
```

### Safe variable access
```go
{{- if .work -}}
{{- if .work.email -}}
Work email: {{ .work.email }}
{{- end -}}
{{- end -}}
```

## Related

- **[Setup Guide](SETUP.md)**: Initial configuration
- **[Security](SECURITY.md)**: Encrypting template files
- **[Troubleshooting](TROUBLESHOOTING.md)**: Template debugging