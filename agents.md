# Using Skills with AI Agents

This repo contains skills (contextual knowledge and tools) that can be used by AI coding agents like Claude Code and OpenAI Codex.

## Directory Structure

```
/workspace/
├── skills/                    # Main skills directory
│   ├── cast/                  # Foundry Cast CLI reference
│   ├── makina-contracts/      # Makina smart contract interfaces
│   └── test-machines/         # Test Machine addresses
├── .claude-plugin/
│   └── plugin.json            # Claude Code plugin manifest
```

## Claude Code

### Plugin Setup

The `/.claude-plugin/plugin.json` registers skills as a plugin:

### Using Skills

Claude Code automatically loads skills when relevant. You can also explicitly invoke them:

```
/makina-contracts    # Load Makina contract interfaces
/cast               # Load Foundry Cast CLI reference
/test-machines      # Load test Machine addresses
```

## OpenAI Codex

....