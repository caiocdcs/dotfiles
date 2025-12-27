# Dotfiles

Complete Mac setup with chezmoi + Vaultwarden for secret management.

## What's Included

- **Shell configs:** .zshrc, .zprofile, .p10k.zsh (Powerlevel10k)
- **Git config:** .gitconfig with signing enabled
- **App configs:** ghostty, fish, nushell, mise, jj, gh
- **SSH config:** .ssh/config
- **Brewfile:** All 76+ packages
- **Secrets:** Managed via self-hosted Vaultwarden

## Quick Setup (New Mac)

```bash
# Install Homebrew
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Install chezmoi and bitwarden-cli
brew install chezmoi bitwarden-cli

# Configure Vaultwarden server
bw config server https://vault.int.cdcs.dev
bw login caio@cdcs.dev

# Clone and apply dotfiles
chezmoi init --apply git@github.com:caiocdcs/dotfiles.git

# Unlock Vaultwarden and apply secrets
export BW_SESSION=$(bw unlock --raw)
chezmoi apply -v

# Install all packages
brew bundle --global
```

Done! Your Mac is set up exactly like your current one.

## Vaultwarden Integration

**Server:** `https://vault.int.cdcs.dev`

### How It Works

Templates in this repo use Vaultwarden to fetch secrets:

```
{{ bitwarden "item" "Item Name" }}
```

Secrets stay in Vaultwarden. Templates are in git. Perfect security! 

### Examples

#### SSH Private Key

**Store in Vaultwarden:**
- Type: Secure Note
- Name: "SSH Private Key"
- Notes: Paste your private key

**Template:** `private_dot_ssh/private_id_ed25519.tmpl`
```
{{- if (bitwarden "item" "SSH Private Key") -}}
{{ (bitwarden "item" "SSH Private Key").notes }}
{{- end -}}
```

#### Environment Variables

**In `.zshrc.tmpl`:**
```bash
export GITHUB_TOKEN="{{ (bitwarden "item" "GitHub Token").password }}"
export NPM_TOKEN="{{ (bitwarden "item" "NPM Token").password }}"
```

## Common Commands

```bash
# Package management
brew bundle dump --global --force   # Update Brewfile
brew bundle --global                 # Install from Brewfile

# Dotfile management
chezmoi edit ~/.zshrc               # Edit file
chezmoi diff                        # See changes
chezmoi apply -v                    # Apply changes
chezmoi add ~/.newconfig            # Track new file

# Bitwarden
export BW_SESSION=$(bw unlock --raw)  # Unlock vault
bw list items | jq -r '.[].name'      # List items
alias bwu='export BW_SESSION=$(bw unlock --raw)'  # Quick unlock

# Git
chezmoi cd && git status            # Check repo status
chezmoi cd && git push              # Push changes
```

## File Structure

```
~/.local/share/chezmoi/
├── .chezmoi.toml.tmpl              # Config with Vaultwarden server
├── Brewfile                        # All packages (76+)
├── README.md                       # This file
├── dot_zshrc                       # Shell config
├── dot_gitconfig                   # Git config
├── dot_config/
│   ├── ghostty/config              # Terminal
│   ├── mise/config.toml            # Tool versions
│   ├── jj/config.toml              # Jujutsu VCS
│   └── ...
└── private_dot_ssh/
    └── private_id_ed25519.tmpl     # SSH key (from Vaultwarden)
```

## Security

✅ **Safe to commit:**
- Templates (`.tmpl` files)
- Config with server URL
- Brewfile

❌ **Never in git:**
- Actual secrets (in Vaultwarden)
- Private keys (generated from templates)
- Session tokens

## Tips

### Auto-update Brewfile

Add to `.zshrc`:
```bash
alias brewup='brew update && brew upgrade && brew bundle dump --global --force && chezmoi re-add Brewfile'
```

### Bitwarden session helper

Add to `.zshrc`:
```bash
# Check if vault is unlocked
if command -v bw &> /dev/null; then
    if ! bw unlock --check &> /dev/null; then
        echo "Vaultwarden locked. Run: bwu"
    fi
fi

alias bwu='export BW_SESSION=$(bw unlock --raw)'
```

---

**Everything in one repo. Simple. Secure. Reproducible.**
