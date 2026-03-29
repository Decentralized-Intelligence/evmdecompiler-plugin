# EVM Decompiler — Codex Plugin

A [Codex](https://openai.com/codex) plugin that decompiles EVM smart contract bytecode into human-readable Solidity source code, powered by [evmdecompiler.com](https://www.evmdecompiler.com).

## What it does

Give Codex raw EVM bytecode **or a contract address** and this plugin will:

1. Fetch bytecode from a public RPC if an address is provided
2. Submit the bytecode to the decompiler API
3. Poll for the result automatically
4. Return decompiled Solidity source code

It tries the latest model first, and falls back to the default model if the first attempt fails or times out. Supports Ethereum, Optimism, BSC, Polygon, Base, Arbitrum, and Avalanche.

## Install

### Option 1: Clone into your personal plugins directory

```bash
mkdir -p ~/.codex/plugins
git clone https://github.com/Decentralized-Intelligence/evmdecompiler-plugin.git ~/.codex/plugins/evmdecompiler-plugin
```

Then add a personal marketplace file at `~/.agents/plugins/marketplace.json`:

```json
{
  "name": "local-plugins",
  "interface": { "displayName": "Local Plugins" },
  "plugins": [
    {
      "name": "evmdecompiler",
      "source": {
        "source": "local",
        "path": "./.codex/plugins/evmdecompiler-plugin"
      },
      "policy": {
        "installation": "AVAILABLE",
        "authentication": "ON_INSTALL"
      },
      "category": "Developer Tools"
    }
  ]
}
```

Restart Codex, open the plugin directory, select **Local Plugins**, and install.

### Option 2: Add to a repo

Copy the plugin into your repo:

```bash
cp -R evmdecompiler-plugin ./plugins/evmdecompiler-plugin
```

Then add a repo marketplace at `.agents/plugins/marketplace.json`:

```json
{
  "name": "repo-plugins",
  "interface": { "displayName": "Repo Plugins" },
  "plugins": [
    {
      "name": "evmdecompiler",
      "source": {
        "source": "local",
        "path": "./plugins/evmdecompiler-plugin"
      },
      "policy": {
        "installation": "AVAILABLE",
        "authentication": "ON_INSTALL"
      },
      "category": "Developer Tools"
    }
  ]
}
```

Restart Codex and install from the plugin directory.

## Usage

Once installed, just ask Codex:

> Decompile this EVM bytecode: 0x6060604052...

or

> Decompile the bytecode of the contract at address 0x... on Ethereum mainnet

## Links

- [evmdecompiler.com](https://www.evmdecompiler.com)
- [Codex Plugin Docs](https://developers.openai.com/codex/plugins/build)
