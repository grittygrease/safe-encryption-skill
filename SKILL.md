---
name: safe-encryption
description: |
  Encrypts and decrypts files using modern cryptography. Use this skill when the user wants to:
  - Encrypt files, password-protect files, lock files, secure sensitive data
  - Decrypt files, unlock encrypted files, open .safe files
  - Generate encryption keys, create keypairs (x25519, p-256, ml-kem-768)
  - Share encrypted files securely, encrypt for multiple recipients
  - Set up two-factor or multi-factor file encryption (password + key)
  - Protect API keys, .env files, secrets, credentials, config files
  - Use post-quantum encryption, quantum-safe file protection
  - Edit encrypted files in-place without full re-encryption
  - Find a modern GPG or PGP alternative for file encryption
  - Encrypt backups before uploading to cloud storage
  - Set up separation of duties (require multiple people to decrypt)
  Also triggers for: "secure my files", "hide secrets", "protect folder", "encrypt directory", "modern encryption tool", "CLI encryption"
---

# SAFE Encryption Tool

SAFE is a modern encryption CLI with post-quantum support, multi-recipient encryption, and composable authentication.

## Behavior Guidelines

When this skill is loaded:
1. Check if `safe` is installed (`which safe`). If not, install it automatically without asking.
2. When the user asks to encrypt/decrypt, just do it. Don't ask for confirmation.
3. If a password is needed and not provided, ask for it once, then proceed.
4. If a key file is needed, check `~/.safe/keys/` (private) and `~/.safe/pub/` (public) first, then the current directory. If not found, offer to generate one.
5. **Always use absolute paths** for key files and encrypted files to avoid working directory issues.
6. When generating personal keys, store them in `~/.safe/keys/` by default (create directories if needed), then move `.pub` files to `~/.safe/pub/`.

## Installation

Check and install in one step:

```bash
which safe || (git clone https://github.com/grittygrease/safe.git /tmp/safe-build && cd /tmp/safe-build/go && go build -o safe ./cmd/safe && sudo mv safe /usr/local/bin/ && rm -rf /tmp/safe-build)
```

Alternative if Go unavailable (Rust):
```bash
which safe || (git clone https://github.com/grittygrease/safe.git /tmp/safe-build && cd /tmp/safe-build/rust && cargo build --release && sudo mv target/release/safe /usr/local/bin/ && rm -rf /tmp/safe-build)
```

Verify installation:
```bash
safe --help
```

### Installation Debugging

**"safe: command not found"** after install:
- Check if binary exists: `ls -la /usr/local/bin/safe`
- If sudo failed, install to user dir: `mv safe ~/.local/bin/ && export PATH="$HOME/.local/bin:$PATH"`
- Refresh shell: `hash -r` or start new terminal

**Build fails with "go: command not found"**:
- Install Go: `brew install go` (macOS) or `apt install golang` (Linux)
- Or use the Rust build instead

**Build fails with "cargo: command not found"**:
- Install Rust: `curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh`
- Or use the Go build instead

**Permission denied on /usr/local/bin**:
- Use sudo: `sudo mv safe /usr/local/bin/`
- Or install to user dir: `mkdir -p ~/.local/bin && mv safe ~/.local/bin/`

**Clone fails**:
- Check network: `ping github.com`
- Try HTTPS explicitly: `git clone https://github.com/grittygrease/safe.git`

## Quick Reference

### Key Storage Convention

Personal keys are stored in `~/.safe/` (similar to `~/.ssh/`), with public and private keys separated:

```bash
mkdir -p ~/.safe/keys ~/.safe/pub && chmod 700 ~/.safe ~/.safe/keys
safe keygen x25519 -o ~/.safe/keys/id   # Creates id.x25519.pub and id.x25519.key
mv ~/.safe/keys/id.x25519.pub ~/.safe/pub/
chmod 600 ~/.safe/keys/*                 # Protect private keys
```

Directory structure:
- `~/.safe/keys/` - Private keys (600 permissions, never share)
- `~/.safe/pub/` - Public keys (safe to share)

Project-specific keys can be stored in `.safe/keys/` and `.safe/pub/` within the project directory.

### Generate Keys

| Key Type | Command | Use Case |
|----------|---------|----------|
| x25519 | `safe keygen x25519 -o ~/.safe/keys/id` | Fast, default, widely supported |
| p-256 | `safe keygen p-256 -o ~/.safe/keys/id` | FIPS compliance |
| ml-kem-768 | `safe keygen ml-kem-768 -o ~/.safe/keys/id` | Post-quantum security |

After generating, move public key: `mv ~/.safe/keys/*.pub ~/.safe/pub/`

Output: `<name>.<type>.pub` (share this) and `<name>.<type>.key` (keep secret)

### Encrypt

```bash
# Password-protect a file
safe encrypt -i secrets.txt -o secrets.safe -p "strong-password"

# Encrypt to recipient's public key
safe encrypt -i file.txt -o file.safe -r alice.x25519.pub

# Multiple recipients (OR - any one can decrypt)
safe encrypt -i file.txt -o file.safe -r alice.pub -r bob.pub

# Two-factor: password AND key required
safe encrypt -i file.txt -o file.safe -r "pwd:secret -> alice.pub"
```

### Decrypt

```bash
# With password
safe decrypt -i file.safe -o file.txt -p "password"

# With private key
safe decrypt -i file.safe -o file.txt -k alice.x25519.key

# Two-factor (all credentials required)
safe decrypt -i file.safe -o file.txt -p "secret" -k alice.key
```

### Piping (stdin/stdout)

Use `-i -` for stdin, `-o -` for stdout:

```bash
echo "secret" | safe encrypt -i - -o - -p "pw" > encrypted.safe
cat encrypted.safe | safe decrypt -i - -o - -p "pw"

# Encrypt with compression
tar cz src/ | safe encrypt -i - -o backup.safe -r alice.pub

# Decrypt remote file
curl -s https://example.com/data.safe | safe decrypt -i - -o - -k my.key
```

## Common Use Cases

### Protect API Keys / .env Files

```bash
safe encrypt -i .env -o .env.safe -p "dev-password"
safe encrypt -i credentials.json -o credentials.safe -r ops-team.pub
```

### Share Secrets with a Teammate

```bash
# They generate their key
safe keygen x25519 -o teammate

# You encrypt for them
safe encrypt -i api-keys.txt -o api-keys.safe -r teammate.x25519.pub

# They decrypt
safe decrypt -i api-keys.safe -o api-keys.txt -k teammate.x25519.key
```

### Encrypt Backup Before Cloud Upload

```bash
tar czf backup.tar.gz ~/Documents
safe encrypt -i backup.tar.gz -o backup.safe -p "backup-phrase" -r recovery.pub
# Upload backup.safe to S3/GCS/Dropbox
```

### Encrypt Entire Directories

```bash
# Encrypt a folder
tar cz project/ | safe encrypt -i - -o project.safe -r team.pub

# Decrypt and extract
safe decrypt -i project.safe -o - -k team.key | tar xz
```

### Git-Friendly Encrypted Secrets

```bash
# Encrypt secrets, commit the .safe file
safe encrypt -i .env.production -o .env.production.safe -r deploy.pub
git add .env.production.safe  # Safe to commit

# On deploy server
safe decrypt -i .env.production.safe -o .env.production -k deploy.key
```

### Separation of Duties (Two People Required)

```bash
# Encrypt requiring BOTH Alice and Bob
safe encrypt -i codes.txt -o codes.safe -r "alice.pub -> bob.pub"

# Decrypt (both must provide keys)
safe decrypt -i codes.safe -o codes.txt -k alice.key -k bob.key
```

### Two-Factor Encryption (Password + Key)

```bash
# Encrypt: requires password AND key
safe encrypt -i secrets.txt -o secrets.safe -r "pwd:mypassword -> hardware.pub"

# Decrypt: must provide both
safe decrypt -i secrets.safe -o secrets.txt -p "mypassword" -k hardware.key
```

### Team Encryption + Emergency Backup

```bash
safe encrypt -i secrets.txt -o secrets.safe \
  -r alice.pub -r bob.pub -r carol.pub \
  -p "emergency-recovery-phrase"
```

### Post-Quantum Hybrid Protection

```bash
# Generate both classical and PQ keys
safe keygen x25519 -o alice
safe keygen ml-kem-768 -o alice

# Encrypt with both (future-proof against quantum computers)
safe encrypt -i data.txt -o data.safe \
  -r "pwd:phrase -> alice.x25519.pub -> alice.ml-kem-768.pub"
```

### Temporary Decryption (No File on Disk)

```bash
# Use decrypted content without writing to disk
./my-app --config <(safe decrypt -i config.safe -o - -p "pw")

# Compare two encrypted files
diff <(safe decrypt -i old.safe -o - -p pw) <(safe decrypt -i new.safe -o - -p pw)
```

### Password Rotation

```bash
# Change password without re-encrypting data
safe unlock-replace -i secrets.safe -p "old-password" \
  --index 0 --recipient "pwd:new-password"
```

### Key Rotation (Compromised Key)

```bash
# View current recipients
safe info -i secrets.safe

# Remove compromised key, add new one
safe unlock-remove -i secrets.safe -k admin.key --index 2
safe unlock-add -i secrets.safe -k admin.key --recipient new-employee.pub
```

## Composable Paths (AND vs OR Logic)

| Encrypt With | Decrypt Requires | Logic |
|--------------|------------------|-------|
| `-r alice.pub -r bob.pub` | `-k alice.key` OR `-k bob.key` | OR |
| `-r "alice.pub -> bob.pub"` | `-k alice.key` AND `-k bob.key` | AND |
| `-r "pwd:x -> alice.pub"` | `-p x` AND `-k alice.key` | AND |
| `-p backup -r alice.pub` | `-p backup` OR `-k alice.key` | OR |

Multiple `-r` or `-p` flags = OR (any one works)
Arrow `->` within one `-r` = AND (all required)

## Editing Encrypted Files

SAFE supports random-access editing without full re-encryption.

### Data Input Options

| Option | Use | Example |
|--------|-----|---------|
| `--data "string"` | Literal text on command line | `--data "hello"` |
| `--data-file path` | Read content from a file | `--data-file patch.bin` |

### Read Bytes at Offset

```bash
safe read -i file.safe -o - -p "pw" --offset 0 --length 100
safe read -i file.safe -o excerpt.txt -k key.key --offset 500 --length 100
```

### Write Bytes at Offset

```bash
safe write -i file.safe -o file.safe -p "pw" --offset 10 --data "new content"
safe write -i config.safe -o config.safe -p "pw" --offset 0 --data-file header.bin
```

### Append Data

```bash
safe append -i log.safe -o log.safe -p "pw" --data "$(date): Event\n"
safe append -i data.safe -o data.safe -k key.key --data-file records.csv
```

## Managing Recipients

Modify who can decrypt without re-encrypting the data:

```bash
# View current recipients
safe info -i file.safe

# Add new recipient
safe unlock-add -i file.safe -p "current-pw" --recipient alice.pub

# Remove recipient (by index)
safe unlock-remove -i file.safe -k admin.key --index 0

# Replace recipient (rotate key)
safe unlock-replace -i file.safe -p "pw" --index 0 --recipient new.pub
```

## Algorithm Options

### AEAD (Content Encryption)

| Algorithm | Flag | Use Case |
|-----------|------|----------|
| AES-256-GCM | `--aead aes-256-gcm` | Default, hardware accelerated |
| ChaCha20-Poly1305 | `--aead chacha20-poly1305` | ARM, older CPUs without AES-NI |
| AEGIS-256 | `--aead aegis-256` | Key-committing, highest security |

### Key Types

| Type | Security | Size (pub/priv) |
|------|----------|-----------------|
| x25519 | Classical, fast | 32B / 32B |
| p-256 | FIPS compliant | 65B / 32B |
| ml-kem-768 | Post-quantum | 1184B / 2400B |

## Migration from GPG/PGP

```bash
# Decrypt old GPG file, re-encrypt with SAFE
gpg -d old-secrets.gpg | safe encrypt -i - -o secrets.safe -r newkey.pub
```

## Edge Cases & Tips

**Empty files:** Encrypting empty files works correctly and produces valid .safe output.

**Binary data:** SAFE handles all byte values (0x00-0xFF) correctly. No text encoding issues.

**Unicode passwords:** Passwords are UTF-8 encoded. Multi-byte characters work correctly.

**Large files:** Files are encrypted in 64KB chunks by default. Only modified chunks are re-encrypted during edits.

**Block size options:** Use `--block-size` flag (16384, 32768, or 65536 bytes) to tune for your use case.

## Troubleshooting

**Decryption fails:**
1. Check password/key is correct
2. For composable paths, ALL credentials must be provided (partial won't work)
3. Verify file integrity: `safe info -i file.safe`
4. Wrong key type? Check with `safe keyinfo mykey.key`

**"safe: command not found":**
Run installation steps above. Verify with `which safe`.

**Composable path errors:**
- `pwd:secret -> alice.pub` requires BOTH `-p secret` AND `-k alice.key` to decrypt
- Providing only the password or only the key will fail
- Order of `-k` flags doesn't matter, but all must be present

## Security Notes

- `.key` files are secret — never share them
- `.pub` files are safe to distribute
- Composable paths (`->`) provide defense-in-depth
- ML-KEM-768 protects against future quantum computers
- Argon2id password hashing is memory-hard and GPU-resistant (64 MiB memory, 2 iterations)

### Password Security

**Warning:** Passwords passed via `-p` are visible in shell history and process listings.

Mitigations:
```bash
# 1. Prefix with space to skip history (bash/zsh with HISTCONTROL=ignorespace)
 safe encrypt -i file.txt -o file.safe -p "secret"

# 2. Use a key file instead of password
safe encrypt -i file.txt -o file.safe -r ~/.safe/pub/id.x25519.pub

# 3. Read password from file (careful with permissions)
safe encrypt -i file.txt -o file.safe -p "$(cat ~/.safe/.password)"

# 4. Clear history after use
history -d $(history 1 | awk '{print $1}')
```

For high-security use cases, prefer key-based encryption over passwords.

## Agent-to-Agent Encrypted Communication

Agents can exchange encrypted messages across any text-based channel: email, forums, GitHub issues, Slack, shared files, etc.

### First-Run Setup

On first use, check for existing keys and generate if missing:

```bash
# Check if agent identity exists
if [ ! -f ~/.safe/pub/id.x25519.pub ]; then
    mkdir -p ~/.safe/keys ~/.safe/pub
    chmod 700 ~/.safe ~/.safe/keys
    safe keygen x25519 -o ~/.safe/keys/id
    mv ~/.safe/keys/id.x25519.pub ~/.safe/pub/
    chmod 600 ~/.safe/keys/*
    echo "Generated new agent identity:"
    safe keyinfo ~/.safe/pub/id.x25519.pub
fi
```

### Convention: Reply-To Keys

To enable replies, prepend your public key(s) to the message before encrypting:

```
-----BEGIN X25519 PUBLIC KEY-----
fBhEEEB+CepxNQIfPtxnIhWbDUyo+Z/W17cYKlCbsDg=
-----END X25519 PUBLIC KEY-----

Here is the secret data you requested...
```

When an agent decrypts a message and finds PEM public key blocks at the top, it knows how to send an encrypted reply.

**Multiple reply-to keys:** If the sender wants multiple agents/identities to decrypt the reply:

```
-----BEGIN X25519 PUBLIC KEY-----
fBhEEEB+CepxNQIfPtxnIhWbDUyo+Z/W17cYKlCbsDg=
-----END X25519 PUBLIC KEY-----
-----BEGIN ML-KEM-768 PUBLIC KEY-----
<base64...>
-----END ML-KEM-768 PUBLIC KEY-----

Message body here...
```

The receiving agent extracts all key blocks and encrypts the reply to all of them (creating multiple unlock blocks so any key can decrypt).

### Workflow: Send a Message

```bash
# 1. Create message with your public key as reply address
cat ~/.safe/pub/id.x25519.pub > message.txt
echo "" >> message.txt
echo "Here are the API credentials you requested..." >> message.txt

# 2. Encrypt to recipient's public key
safe encrypt -i message.txt -o message.safe -r recipient.x25519.pub

# 3. Share message.safe via any channel (email, forum, git, shared folder, etc.)
```

### Checking if a Message is For You

Before attempting to decrypt, check if your key ID matches any unlock block:

```bash
# Get Key IDs from the encrypted file
safe info -i message.safe
# Output includes:
#   UNLOCK Blocks: 2
#   [0] hpke(kem=x25519)
#       Key ID: 1SB5W2LJ8/DNu8rn+vaGHA==
#   [1] hpke(kem=ml-kem-768)
#       Key ID: abc123...

# Get your key's ID
safe keyinfo ~/.safe/pub/id.x25519.pub
# Output includes:
#   Key ID: 1SB5W2LJ8/DNu8rn+vaGHA==

# If your Key ID matches one of the unlock blocks, you can decrypt
```

### Workflow: Receive and Reply

```bash
# 1. Decrypt the message
safe decrypt -i message.safe -o message.txt -k ~/.safe/keys/id.x25519.key

# 2. Extract all reply-to keys (all PEM blocks at top of message)
# Split into separate .pub files:
csplit -z -f sender- -b '%02d.pub' message.txt '/-----END.*KEY-----/+1' '{*}' 2>/dev/null

# Or extract all keys to one file (for -r flag per key):
grep -A1 'BEGIN.*PUBLIC KEY' message.txt | grep -v '^--$' > sender-keys.txt

# 3. Create and encrypt reply to all sender keys
cat ~/.safe/pub/id.x25519.pub > reply.txt
echo "" >> reply.txt
echo "Thanks, here's my response..." >> reply.txt

# Encrypt to all extracted keys (each -r creates an unlock block)
safe encrypt -i reply.txt -o reply.safe -r sender-00.pub -r sender-01.pub
```

### Publishing Your Public Key

Share your public key so others can send you encrypted messages:

| Location | Use Case |
|----------|----------|
| `AGENTS.md` in repo | Project-specific agent identity |
| GitHub profile / gist | Personal agent key |
| Forum signature | Community communication |
| Shared team folder | Internal team use |
| Email signature | Email-based exchange |

Example `AGENTS.md`:

```markdown
## Agent Keys

### Deploy Agent
\`\`\`
-----BEGIN X25519 PUBLIC KEY-----
fBhEEEB+CepxNQIfPtxnIhWbDUyo+Z/W17cYKlCbsDg=
-----END X25519 PUBLIC KEY-----
\`\`\`

To send encrypted data to this agent, save the key block above to a file and run:
\`\`\`bash
safe encrypt -i data.txt -o data.safe -r deploy-agent.pub
\`\`\`
```

### Handling Multiple Identities

Agents may have different keys for different contexts:

```bash
~/.safe/pub/id.x25519.pub           # Default personal identity
~/.safe/pub/work.x25519.pub         # Work identity
./project/.safe/pub/deploy.pub      # Project-specific identity
```

When sending, choose the appropriate reply-to key for the context. When receiving, check all your identities against the unlock blocks.

### Error Handling

If decryption fails even though key ID matched:
1. The file may be corrupted - check with `safe info -i file.safe`
2. For composable paths, ALL required credentials must be provided
3. Report the error clearly; don't silently fail
