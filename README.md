# Dotfiles with Chezmoi + Vaultwarden

Self-hosted Vaultwarden instance for secret management.

## Vaultwarden Setup

**Server:** `https://vault.int.cdcs.dev`

### Initial Setup

1. **Configure Bitwarden CLI:**
   ```bash
   bw config server https://vault.int.cdcs.dev
   ```

2. **Login:**
   ```bash
   bw login caio.cdcs@gmail.com
   ```

3. **Unlock vault (store session):**
   ```bash
   export BW_SESSION=$(bw unlock --raw)
   ```

4. **Apply dotfiles:**
   ```bash
   chezmoi apply
   ```

### How It Works

Chezmoi templates fetch secrets from Vaultwarden using:

```
{{ bitwarden "item" "Item Name" }}
```

## Examples

### SSH Private Key

1. **Store in Vaultwarden:**
   - Type: Secure Note
   - Name: "SSH Private Key"
   - Notes: Paste your `~/.ssh/id_ed25519` content

2. **Template:** `private_dot_ssh/private_id_ed25519.tmpl`
   ```
   {{- if (bitwarden "item" "SSH Private Key") -}}
   {{ (bitwarden "item" "SSH Private Key").notes }}
   {{- end -}}
   ```

### Environment Variables (NPM, GitHub, etc.)

Store tokens in Vaultwarden, use in shell:

**In `.zshrc.tmpl`:**
```bash
# GitHub Token
export GITHUB_TOKEN="{{ (bitwarden "item" "GitHub Token").password }}"

# NPM Token
export NPM_TOKEN="{{ (bitwarden "item" "NPM Token").password }}"

# JFrog (if needed later)
export JFROG_TOKEN="{{ (bitwarden "item" "JFrog Token").password }}"
```

### GPG Signing Key

**In `.gitconfig.tmpl`:**
```toml
[user]
    email = {{ .email }}
    name = "Caio Silva"
    signingkey = {{ (bitwarden "item" "GPG Signing Key ID").password }}

[commit]
    gpgsign = true
```

### AWS Credentials

**Store in Vaultwarden:**
- Name: "AWS Access Key"
- Username: Access Key ID
- Password: Secret Access Key

**In `.aws/credentials.tmpl`:**
```ini
[default]
aws_access_key_id = {{ (bitwarden "item" "AWS Access Key").username }}
aws_secret_access_key = {{ (bitwarden "item" "AWS Access Key").password }}
```

## Useful Commands

```bash
# List all items
bw list items | jq -r '.[].name'

# Search for specific item
bw get item "SSH Private Key"

# Get item by ID
bw get item <item-id>

# Test template before applying
chezmoi execute-template < template-file.tmpl

# See what would change
chezmoi diff

# Apply with verbose output
chezmoi apply -v

# Re-unlock if session expires
export BW_SESSION=$(bw unlock --raw)
```

## File Naming Convention

Chezmoi uses special prefixes:

- `dot_` → `.` (dotfile)
- `private_` → file with 0600 permissions
- `executable_` → file with executable bit
- `.tmpl` → template (processed)

Examples:
- `dot_zshrc.tmpl` → `~/.zshrc` (template)
- `private_dot_ssh/private_id_ed25519.tmpl` → `~/.ssh/id_ed25519` (0600, template)
- `dot_config/ghostty/config` → `~/.config/ghostty/config` (plain file)

## Security

✅ **Safe to commit:**
- `.chezmoi.toml.tmpl` (variables only)
- Template files (`.tmpl`)
- Server URL
- Email address

❌ **Never committed:**
- Actual secrets (in Vaultwarden)
- `BW_SESSION` token
- Private keys
- Passwords/tokens

## New Machine Setup

```bash
# Install dependencies
brew install chezmoi bitwarden-cli

# Configure server
bw config server https://vault.int.cdcs.dev

# Login
bw login caio.cdcs@gmail.com

# Clone dotfiles
chezmoi init --apply https://github.com/YOUR_USERNAME/dotfiles.git

# Unlock and apply
export BW_SESSION=$(bw unlock --raw)
chezmoi apply -v
```

## Tips

### Auto-unlock in .zshrc

Add to your shell config:
```bash
# Check if bw is unlocked, if not prompt
if command -v bw &> /dev/null; then
    if ! bw unlock --check &> /dev/null; then
        echo "🔒 Bitwarden vault locked. Unlock to load secrets."
        # Optionally auto-unlock:
        # export BW_SESSION=$(bw unlock --raw)
    fi
fi
```

### Session Timeout

Session expires after inactivity. Re-unlock:
```bash
export BW_SESSION=$(bw unlock --raw)
```

Or add to `.zshrc`:
```bash
alias bwu='export BW_SESSION=$(bw unlock --raw)'
```

---

**Your secrets stay in Vaultwarden. Templates stay in git. Perfect!** 🔒
