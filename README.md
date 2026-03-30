# ProductOS

A marketplace of AI-powered plugins for product managers — each targeting a specific PM job with a clear input, a designed process, and a useful output.

## What is this?

ProductOS is a [Claude Code](https://claude.ai/code) plugin marketplace (skills can also be used by other AI coding assistants). Each plugin installs a set of slash commands directly into your Claude Code session, giving you structured, opinionated workflows built around real PM jobs — not generic prompts.

## Installation

### Claude Cowork (recommended for non-developers)

1. Open **Customize** (bottom-left)
2. Go to **Browse plugins** → **Personal** → **+**
3. Select **Add marketplace from GitHub**
4. Enter: `alextacho/ProductOS`

All plugins install automatically. 

### Claude Code (CLI)

Either use built in `/plugin` command within a Claude Code session or install via CLI:

```bash
# Step 1: Add the marketplace
claude plugin marketplace add alextacho/ProductOS

# Step 2: Install individual plugins
claude plugin install competitive-intelligence@pProductOS
```

## Plugins

| Plugin | Job |
|--------|-----|
| [competitive-intelligence](./competitive-intelligence/) | Track competitors, collect signals, detect change, and synthesize insight |


## License

MIT © Alexander Tacho
