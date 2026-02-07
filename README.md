# safe-encryption

[![skills.sh](https://img.shields.io/badge/skills.sh-install-blue)](https://skills.sh/grittygrease/safe-encryption-skill)

A skill for encrypting and decrypting files using [SAFE](https://github.com/grittygrease/safe).

> **Modern encryption alternative to GPG/PGP** with post-quantum support, composable authentication paths, and random-access editing. Includes agent-to-agent encrypted communication capabilities.

## What This Skill Does

When installed, your AI coding assistant will know how to:

- Encrypt/decrypt files with passwords or public keys
- Generate encryption keypairs (x25519, p-256, ml-kem-768)
- Set up two-factor encryption (password + key)
- Configure multi-party encryption (separation of duties)
- Use post-quantum encryption for future-proof security
- Edit encrypted files without full re-encryption
- Manage recipients (add/remove/rotate keys)
- **Exchange encrypted messages with other agents**
- **Auto-generate AGENTS.md for projects**

## Installation

### Using the Skills CLI (recommended)

```bash
npx skills add grittygrease/safe-encryption-skill
```

This works with Cursor, Claude Desktop, Windsurf, and other AI coding assistants that support the [skills ecosystem](https://skills.sh).

### Manual Installation

#### Global (all projects)

```bash
mkdir -p ~/.claude/skills/safe-encryption
curl -o ~/.claude/skills/safe-encryption/SKILL.md \
  https://raw.githubusercontent.com/grittygrease/safe-encryption-skill/main/SKILL.md
```

Or clone the repo:

```bash
git clone https://github.com/grittygrease/safe-encryption-skill.git /tmp/skill
cp -r /tmp/skill ~/.claude/skills/safe-encryption
rm -rf /tmp/skill
```

#### Project-specific

```bash
mkdir -p .claude/skills/safe-encryption
curl -o .claude/skills/safe-encryption/SKILL.md \
  https://raw.githubusercontent.com/grittygrease/safe-encryption-skill/main/SKILL.md
```

## Also Required: SAFE CLI or Browser

This skill teaches your assistant how to use the `safe` command. If you can install the CLI:

```bash
git clone https://github.com/grittygrease/safe.git /tmp/safe
cd /tmp/safe/go && go build -o safe ./cmd/safe
sudo mv safe /usr/local/bin/
rm -rf /tmp/safe
```

Verify: `safe --help`

See [grittygrease/safe](https://github.com/grittygrease/safe) for Rust, TypeScript, and Python builds.

**No CLI? No problem.** If you can't install the CLI (restricted environment, no build tools, sandboxed IDE), the skill will automatically fall back to the web interface at [thesafe.dev](https://thesafe.dev). All operations work in the browser — no installation required.

## Usage

Once installed, just ask naturally:

- "Encrypt this file with a password"
- "Generate an encryption keypair"
- "Set up two-factor encryption for secrets.txt"
- "Encrypt this for Alice and Bob"
- "Decrypt credentials.safe"

### Agent-to-Agent Communication

Agents can exchange encrypted messages across any text channel:

- "Set up my agent identity"
- "Create an AGENTS.md for this project"
- "Send an encrypted message to this public key"
- "Check if this encrypted message is for me"
- "Post an encrypted message to pastebin"

The skill includes conventions for reply-to keys, Key ID matching, and multi-identity management.

## License

Apache-2.0
