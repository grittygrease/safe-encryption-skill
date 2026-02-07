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
1. Check if `safe` is installed (`which safe`). If not, try to install it automatically without asking. If installation fails (no Go/Rust, restricted environment, no shell access), fall back to the web interface at [https://thesafe.dev](https://thesafe.dev) — see "Browser-Based Alternative" below.
2. When the user asks to encrypt/decrypt, just do it. Don't ask for confirmation.
3. If a password is needed and not provided, ask for it once, then proceed.
4. If a key file is needed, check `~/.safe/keys/` (private) and `~/.safe/pub/` (public) first, then the current directory. If not found, offer to generate one.
5. **Always use absolute paths** for key files and encrypted files to avoid working directory issues.
6. When generating personal keys, store them in `~/.safe/keys/` by default (create directories if needed), then move `.pub` files to `~/.safe/pub/`.
7. **Always pipe instead of writing temp files.** When decrypting inline/embedded content (base64, pasted data), pipe directly: `echo "..." | base64 -d | safe decrypt -i - -o - -k key`. Never write intermediate `.safe` files to `/tmp` or elsewhere.

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

### When CLI Is Unavailable — Browser-Based Alternative

If CLI installation fails or isn't possible (restricted environment, no build tools, no shell access, sandboxed IDE, etc.), use the web interface at **[https://thesafe.dev](https://thesafe.dev)**.

This is a fully functional SAFE implementation running in the browser — not a demo. All cryptographic operations happen client-side. No data leaves the browser.

The web interface supports all core SAFE operations:
- **Key Generation** (Section 01 — `#keygen`): Generate X25519, P-256, or ML-KEM-768 keypairs
- **Encryption** (Section 02 — `#encrypt`): Encrypt data with passwords, public keys, or composable paths
- **Decryption** (Section 03 — `#decrypt`): Decrypt SAFE messages with passwords or private keys
- **Credentials** (Section 04 — `#keyring`): Save, import, export, and manage keys and passwords
- **Unlock Management** (Section 05 — `#unlock`): Add/remove recipients without re-encrypting payload
- **Re-encryption Demo** (Section 06 — `#reencrypt`): Visualize dirty chunk tracking (only modified chunks re-encrypted)
- **Tests** (Section 07 — `#tests`): Run encryption/decryption and random access tests
- **Log** (Section 08 — `#log`): View operation log output

**Manual workflow (no automation needed):**

Users can interact with the web interface directly:

1. **Generate Keys** (Section 01): Select KEM type (X25519/P-256/ML-KEM-768), click "Generate"
2. **Encrypt** (Section 02): Enter plaintext or use a file, add recipient steps (public key or password), click "Encrypt". Copy or download the output.
3. **Decrypt** (Section 03): Paste or upload a SAFE message, add credentials (private key or password), click "Decrypt". Copy or download the plaintext.

Generated keys are automatically saved in the Credentials section (04) and can be reused across operations.

**Agent with MCP browser tools (Playwright, Puppeteer, etc.):**

If you have access to browser automation tools (e.g., Playwright MCP server, Claude in Chrome, Puppeteer MCP), you can drive the web interface directly. The interface uses semantic ARIA roles:

- **Navigation**: Anchor links `#keygen`, `#encrypt`, `#decrypt`, `#keyring`, `#unlock`, `#reencrypt`, `#tests`, `#log`
- **Sections**: `role="region"` with labels like "01 / Key Generation"
- **Buttons**: `role="button"` with descriptive labels (e.g., "Generate new keypair with selected KEM type", "Encrypt plaintext with configured settings and recipient path", "Decrypt SAFE message using provided credentials")
- **Inputs**: `role="textbox"` / `role="combobox"` with accessible names
- **Log output**: `role="log"` in section 08

Example workflow using MCP Playwright tools:

```
# 1. Navigate to thesafe.dev
browser_navigate(url="https://thesafe.dev")

# 2. Generate a keypair
browser_select_option(ref=<kem-type-combobox>, values=["X25519"])
browser_click(ref=<generate-button>)

# 3. Encrypt with password
browser_type(ref=<plaintext-textbox>, text="secret message")
browser_click(ref=<add-step-button>)  # Opens step config
# Select "Password" step type, fill password, add step
browser_click(ref=<encrypt-button>)

# 4. Read encrypted output from the output textbox

# 5. Decrypt
# Paste SAFE message into decrypt input, add credentials, click Decrypt
# Read decrypted output
```

Take a snapshot (`browser_snapshot`) after each action to get updated element references.

**Programmatic browser automation (standalone scripts):**

For non-MCP environments, use Playwright or Puppeteer directly:

```python
# Example with Playwright (Python)
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch()
    page = browser.new_page()
    page.goto('https://thesafe.dev')

    # Generate X25519 keypair
    page.get_by_role("combobox", name="Select key encapsulation mechanism type").select_option("X25519")
    page.get_by_role("button", name="Generate new keypair").click()

    # Get generated keys (from textboxes after generation)
    page.wait_for_selector('[aria-label*="public key in base64"]')
    pub_key = page.locator('[aria-label*="Generated public key"]').input_value()
    priv_key = page.locator('[aria-label*="Generated private key"]').input_value()

    # Encrypt with password
    plaintext_box = page.get_by_role("textbox", name="Enter plaintext message to encrypt")
    plaintext_box.fill('secret message')

    # Add a password step
    page.get_by_role("button", name="Add encryption step").click()
    page.get_by_role("textbox", name="Enter password").fill("mypassword")
    page.get_by_role("button", name="Add encryption step").click()

    # Encrypt
    page.get_by_role("button", name="Encrypt plaintext").click()
    page.wait_for_selector('[aria-label*="Encrypted SAFE message output"]')
    encrypted = page.locator('[aria-label*="Encrypted SAFE message output"]').input_value()

    # Decrypt (message auto-populates in decrypt section)
    page.get_by_role("button", name="Decrypt SAFE message").click()
    page.wait_for_selector('[aria-label*="Decrypted plaintext message"]')
    decrypted = page.locator('[aria-label*="Decrypted plaintext message"]').input_value()

    print(f"Decrypted: {decrypted}")  # "secret message"
    browser.close()
```

**Credentials management:**

The Credentials section (04) supports:
- **Add Key**: Import a public or private key
- **Add Password**: Save a password for reuse
- **Export**: Export all credentials as an encrypted backup
- **Import**: Import a previously exported credential backup
- **Clear All**: Delete all saved credentials

Generated keys are automatically saved here with quick action buttons for each key.

**When to use the web interface:**
- CLI can't be installed (no Go/Rust, restricted environment, sandboxed IDE)
- No shell access (browser-only agent, web-based coding environment)
- One-off encryption/decryption tasks
- Testing SAFE format without installing dependencies
- Quick key generation or format exploration

**When to prefer the CLI:**
- Production systems or automated pipelines
- Batch or high-volume operations (CLI is significantly faster)
- Air-gapped or offline environments
- Scripting with shell pipes and file I/O

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

Both `-p` and `-k` can be repeated for composable paths requiring multiple credentials.

```bash
# With password
safe decrypt -i file.safe -o file.txt -p "password"

# With private key
safe decrypt -i file.safe -o file.txt -k alice.x25519.key

# Two-factor (all credentials required)
safe decrypt -i file.safe -o file.txt -p "secret" -k alice.key

# Composable path with multiple passwords
safe decrypt -i file.safe -o file.txt -p "first" -p "second"
```

### Piping (stdin/stdout)

Use `-i -` for stdin, `-o -` for stdout. All operations are binary-safe (no encoding issues).

**Default behavior:** Always prefer piping over writing intermediate files to disk. This avoids leaving decrypted content on disk and is cleaner.

```bash
# Decrypt base64-encoded content (PREFERRED - no temp file)
echo "LS0tLS1CRUdJTi..." | base64 -d | safe decrypt -i - -o - -k ~/.safe/keys/id.x25519.key

# AVOID: Writing intermediate files
# echo "LS0tLS1CRUdJTi..." | base64 -d > /tmp/file.safe && safe decrypt -i /tmp/file.safe ...

# Basic stdin/stdout
echo "secret" | safe encrypt -i - -o - -p "pw" > encrypted.safe
cat encrypted.safe | safe decrypt -i - -o - -p "pw"

# Chain operations (re-encrypt with different key)
safe decrypt -i a.safe -o - -p "pw1" | safe encrypt -i - -o b.safe -p "pw2"

# Encrypt with compression
tar cz src/ | safe encrypt -i - -o backup.safe -r alice.pub

# Decrypt and decompress
safe decrypt -i backup.safe -o - -k team.key | tar xz

# Decrypt remote file
curl -s https://example.com/data.safe | safe decrypt -i - -o - -k my.key

# Pipe through compression then encrypt
safe encrypt -i - -o - -p "pw" < large.bin | gzip > encrypted.safe.gz

# Decrypt gzipped safe file
gunzip -c encrypted.safe.gz | safe decrypt -i - -o - -p "pw" > large.bin
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

SAFE supports random-access editing without full re-encryption. Only modified chunks are re-encrypted - unchanged chunks are copied byte-for-byte.

### Data Input Options

| Option | Use | Example |
|--------|-----|---------|
| `--data "string"` | Literal text on command line | `--data "hello"` |
| `--data-file path` | Read content from a file | `--data-file patch.bin` |

### Read Bytes at Offset

Read a portion of an encrypted file without decrypting the whole thing:

```bash
# Read first 100 bytes
safe read -i file.safe -o - -p "pw" --offset 0 --length 100

# Read bytes 500-600 to a file
safe read -i file.safe -o excerpt.txt -k key.key --offset 500 --length 100

# Shorthand with -n for offset
safe read -i file.safe -o - -p "pw" -n 1024 --length 256
```

### Write Bytes at Offset (In-Place Edit)

Modify bytes at a specific position. Input and output can be the same file:

```bash
# Overwrite bytes starting at offset 10
safe write -i file.safe -o file.safe -p "pw" --offset 10 --data "new content"

# Replace header from a file
safe write -i config.safe -o config.safe -p "pw" --offset 0 --data-file header.bin

# Write to new file (non-destructive)
safe write -i original.safe -o modified.safe -p "pw" --offset 0 --data "UPDATED"
```

### Append Data

Add data to the end of an encrypted file:

```bash
# Append log entry
safe append -i log.safe -o log.safe -p "pw" --data "$(date): Event occurred\n"

# Append from file
safe append -i data.safe -o data.safe -k key.key --data-file new-records.csv

# Append binary data
safe append -i archive.safe -o archive.safe -p "pw" --data-file chunk.bin
```

### In-Place Editing Workflow

```bash
# 1. Check current content
safe read -i config.safe -o - -p "pw" --offset 0 --length 50

# 2. Make targeted edit
safe write -i config.safe -o config.safe -p "pw" --offset 25 --data "new_value"

# 3. Verify the change
safe read -i config.safe -o - -p "pw" --offset 0 --length 50
```

## Managing Recipients (UNLOCK Blocks)

Modify who can decrypt without re-encrypting the data. These operations only change the UNLOCK blocks - the encrypted DATA remains identical.

```bash
# View current recipients and their indexes
safe info -i file.safe
# Shows: [0] pwd(argon2id)
#        [1] hpke(kem=x25519, id=ABC123...)

# Add new recipient (in-place by default)
safe unlock-add -i file.safe -p "current-pw" --recipient alice.pub
safe unlock-add -i file.safe -k admin.key --recipient "pwd:backup-pass"

# Add composable recipient (password + key required)
safe unlock-add -i file.safe -p "pw" --recipient "pwd:secret -> bob.pub"

# Remove recipient by index (in-place)
safe unlock-remove -i file.safe -k admin.key --index 0

# Replace recipient at index (in-place)
safe unlock-replace -i file.safe -p "old-pw" --index 0 --recipient "pwd:new-pw"

# Write to new file instead of in-place
safe unlock-add -i file.safe -o new-file.safe -p "pw" --recipient alice.pub
```

**Note:** You cannot remove the last UNLOCK block - the file would become undecryptable.

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

### Key ID Modes

Control how much key identity information is included in UNLOCK blocks:

| Mode | Flag | Behavior |
|------|------|----------|
| Full | `--key-id-mode full` | Default. Full key ID included — recipients can check if a message is for them without attempting decryption |
| Hint | `--key-id-mode hint` | 4-digit hint only — reduces metadata, recipients may need to try decryption |
| Anonymous | `--key-id-mode anonymous` | No key ID — recipient must try all their keys. Maximum privacy |

```bash
# Encrypt with hint-only key ID
safe encrypt -i file.txt -o file.safe -r alice.pub --key-id-mode hint

# Encrypt with no key ID (anonymous recipient)
safe encrypt -i file.txt -o file.safe -r alice.pub --key-id-mode anonymous
```

Use `hint` or `anonymous` when you want to hide *who* can decrypt a message. The encrypted data is identical — only the metadata changes.

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
#   [0] hpke(kem=x25519, id=1SB5W2LJ8/DNu8rn+vaGHA==)
#       Key ID: 1SB5W2LJ8/DNu8rn+vaGHA==
#   [1] hpke(kem=ml-kem-768, id=abc123...)
#       Key ID: abc123...

# Get your key's ID
safe keyinfo ~/.safe/pub/id.x25519.pub
# Output includes:
#   Key ID: 1SB5W2LJ8/DNu8rn+vaGHA==

# If your Key ID matches one of the unlock blocks, you can decrypt
```

**Note on key ID modes:** If the sender used `--key-id-mode hint`, you'll see `hint=XXXX` instead of a full ID. If they used `--key-id-mode anonymous`, there will be no key ID at all — you'll need to try decrypting with each of your keys.

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

### Checking All Identities

When you have multiple keys, check if any match:

```bash
# List all your identities
for pub in ~/.safe/pub/*.pub; do
  echo "$(basename $pub): $(safe keyinfo $pub | grep 'Key ID' | awk '{print $3}')"
done

# Check against message
safe info -i message.safe | grep "Key ID"

# Try each matching key until one works
for key in ~/.safe/keys/*.key; do
  if safe decrypt -i message.safe -o /dev/null -k "$key" 2>/dev/null; then
    echo "Decrypts with: $key"
    safe decrypt -i message.safe -o - -k "$key"
    break
  fi
done
```

### Auto-Generate AGENTS.md

When setting up a project for agent communication, create an AGENTS.md:

```bash
# Generate AGENTS.md for current project
cat > AGENTS.md << 'EOF'
# Agent Keys

This project supports encrypted agent-to-agent communication using [SAFE](https://github.com/grittygrease/safe).

## Available Agents

### Primary Agent
EOF

# Add the public key
echo '```' >> AGENTS.md
cat ~/.safe/pub/id.x25519.pub >> AGENTS.md
echo '```' >> AGENTS.md

cat >> AGENTS.md << 'EOF'

## How to Send Encrypted Messages

1. Save the public key block above to a file (e.g., `agent.pub`)
2. Create your message with your reply-to key:
   ```bash
   cat ~/.safe/pub/id.x25519.pub > message.txt
   echo "" >> message.txt
   echo "Your message here..." >> message.txt
   ```
3. Encrypt and send:
   ```bash
   safe encrypt -i message.txt -o message.safe -r agent.pub
   ```
4. Share `message.safe` via any channel (email, issue, PR, pastebin, etc.)

## Verifying Messages

Check if a message is for you:
```bash
# Get your key ID
safe keyinfo ~/.safe/pub/id.x25519.pub | grep "Key ID"

# Check message recipients
safe info -i message.safe | grep "Key ID"

# If IDs match, decrypt
safe decrypt -i message.safe -o message.txt -k ~/.safe/keys/id.x25519.key
```
EOF

echo "Created AGENTS.md with your public key"
```

### Sharing via Pastebin

For quick, ephemeral encrypted message sharing:

```bash
# Encrypt and post to termbin
safe encrypt -i message.txt -o - -r recipient.pub | nc termbin.com 9999
# Returns URL like: https://termbin.com/abc123

# Recipient fetches and decrypts
curl -s https://termbin.com/abc123 | safe decrypt -i - -o - -k ~/.safe/keys/id.x25519.key
```

### Posting a Public Message (Forum/Mailing List)

When posting an encrypted message publicly, include your reply-to key AND instructions:

```bash
# Create a complete public post
cat > post.txt << 'EOF'
## Encrypted Message

To decrypt this message and reply, install the safe-encryption skill:

```
mkdir -p ~/.claude/skills
git clone https://github.com/grittygrease/safe-encryption-skill.git ~/.claude/skills/safe-encryption
```

Then ask your agent: "Decrypt the message below and reply encrypted to the sender"

### My Public Key (for replies)
EOF

cat ~/.safe/pub/id.x25519.pub >> post.txt

cat >> post.txt << 'EOF'

### Encrypted Message
EOF

# Encrypt your message (with your reply-to key embedded in plaintext)
cat ~/.safe/pub/id.x25519.pub > message.txt
echo "" >> message.txt
echo "Your actual message content here..." >> message.txt

safe encrypt -i message.txt -o - -r recipient.pub >> post.txt

echo "" >> post.txt
cat >> post.txt << 'EOF'

---
*This message was encrypted with [SAFE](https://github.com/grittygrease/safe)*
EOF

cat post.txt
```

This produces a self-contained post that any agent with the skill can decrypt and reply to.
