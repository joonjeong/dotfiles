# CLAUDE.md - Dotfiles Repository Instructions

This is a dotfiles repository managed by chezmoi for system configuration management.

## Repository Context
- **Location**: `/Users/joonjeong/.local/share/chezmoi`
- **Tool**: chezmoi (dotfiles manager)
- **Purpose**: Personal system configuration files and dotfiles management

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

## File Structure
- Configuration files are stored in their target directory structure
- Templates and scripts may use chezmoi's templating system
- Private files (containing secrets) should use `.chezmoi.toml.tmpl` or encrypted files

## Best Practices
- Always test changes with `chezmoi diff` before applying
- Use version control for this repository
- Keep sensitive data in encrypted files or templates
- Document any custom scripts or complex configurations

## Notes
- This repository contains personal system configurations
- Be careful when modifying core system files
- Consider backing up before major changes