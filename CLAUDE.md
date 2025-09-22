# CLAUDE.md - Dotfiles Repository Instructions

Chezmoi dotfiles repository with multi-environment support, encryption, and comprehensive documentation.

## Repository Context
- **Location**: `/Users/joonjeong/.local/share/chezmoi`
- **Tool**: chezmoi (dotfiles manager)
- **Features**: Work/personal detection, age encryption, templates

## Key Commands
- `chezmoi apply` - Apply dotfiles to home directory
- `chezmoi add <file>` - Add file to dotfiles management
- `chezmoi edit <file>` - Edit managed file
- `chezmoi status` - Show differences between dotfiles and actual files
- `chezmoi diff` - Show detailed differences

## Common Operations
- **Adding new dotfiles**: Use `chezmoi add ~/.config/filename`
- **Editing configurations**: Use `chezmoi edit ~/.config/filename`
- **Applying changes**: Use `chezmoi apply` after edits
- **Checking status**: Use `chezmoi status` to see what needs updating

## Repository Structure
```
~/.local/share/chezmoi/
├── README.md                 # Repository overview
├── CLAUDE.md                 # This file - Claude instructions
├── .chezmoiignore           # Files to exclude from apply
├── .chezmoi.toml.tmpl       # Main configuration template
├── docs/                     # Documentation (ignored by apply)
│   ├── SETUP.md             # Installation and setup guide
│   ├── TEMPLATES.md         # Template system usage
│   ├── SECURITY.md          # Encryption and security
│   └── TROUBLESHOOTING.md   # Common issues and solutions
├── dot_gitconfig.tmpl       # Git configuration template
├── dot_bashrc               # Bash configuration
├── private_dot_ssh/         # SSH keys (encrypted)
└── dot_config/              # Application configs
    └── nvim/                # Neovim configuration
```

## File Types
- **Templates** (`.tmpl`): Dynamic configs based on hostname, OS, environment
- **Private files** (`private_`): Files with restricted permissions (600/700)
- **Encrypted files** (`.age`): Sensitive data encrypted with age
- **Ignored files**: Documentation stays in repo but doesn't apply to home

## Documentation
- **docs/SETUP.md**: Installation and initial setup
- **docs/TEMPLATES.md**: Template system with examples
- **docs/SECURITY.md**: Age encryption and multi-device setup
- **docs/TROUBLESHOOTING.md**: Common issues and debugging
- **.chezmoi.toml.tmpl**: Config with work/personal detection
- **.chezmoiignore**: Excludes docs from home directory

## Best Practices
- Test with `chezmoi diff` before applying
- Keep sensitive data encrypted with age
- Use templates for environment-specific configs
- Check docs/ for detailed setup and troubleshooting

## Template System
- `.chezmoi.toml.tmpl`: Environment detection (work/personal/OS)
- Template files: `.tmpl` extension with Go syntax
- Access data: `{{ .data.name }}`, `{{ .chezmoi.hostname }}`

## Security
- Private files: 600/700 permissions automatically
- Encryption: Age (`brew install age`)
- SSH keys and tokens should be encrypted