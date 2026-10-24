# Optscan MCP

Connect your AI assistant (Claude Desktop, Claude Code, or any [MCP](https://modelcontextprotocol.io) host) to your [Optscan](https://optscan.io) account so it can look up market and options data for you. Ask questions in plain English, like:

- *"Any $1M+ flows in TSLA expiring in the next 14 days?"*
- *"Which high-IV tickers are good candidates for selling cash-secured puts?"*
- *"Was there unusual call buying in CART before it moved?"*

`optscan-mcp` is a small program that runs on your computer and talks to Optscan on your behalf. **It is read-only:** it cannot create, change or delete anything in your account.

## Requirements

- An Optscan **Basic** or **Pro** plan (the 14-day trial also works).
- **Basic:** 300 tool calls per day. **Pro and trial:** unlimited.
- macOS (Apple Silicon or Intel) or Linux (x86-64).

## 1. Install

Pick the line for your computer. It downloads the program and makes it runnable.

**macOS, Apple Silicon (M1 and later)**

```bash
sudo curl -fL -o /usr/local/bin/optscan-mcp https://github.com/optscan-labs/optscan-mcp-releases/releases/latest/download/optscan-mcp-darwin-arm64 && sudo chmod +x /usr/local/bin/optscan-mcp
```

**macOS, Intel**

```bash
sudo curl -fL -o /usr/local/bin/optscan-mcp https://github.com/optscan-labs/optscan-mcp-releases/releases/latest/download/optscan-mcp-darwin-amd64 && sudo chmod +x /usr/local/bin/optscan-mcp
```

**Linux (x86-64)**

```bash
sudo curl -fL -o /usr/local/bin/optscan-mcp https://github.com/optscan-labs/optscan-mcp-releases/releases/latest/download/optscan-mcp-linux-amd64 && sudo chmod +x /usr/local/bin/optscan-mcp
```

> If you download the file in a browser instead, macOS may block it as coming from an unidentified developer. Run `xattr -d com.apple.quarantine /usr/local/bin/optscan-mcp` once to allow it. Using the `curl` commands above avoids this.

To check the download, compare its SHA-256 with `SHA256SUMS` on the [release page](https://github.com/optscan-labs/optscan-mcp-releases/releases/latest): `shasum -a 256 /usr/local/bin/optscan-mcp`.

## 2. Create a token

1. Log in to Optscan and open **Account**.
2. In **Personal Access Tokens**, enter a name (for example "Claude Desktop") and click **Generate token**.
3. Copy the token now. It starts with `osk_live_` and is shown only once. If you lose it, revoke it and create a new one.

The Account page shows the two configuration snippets below with your token already filled in, so you can copy them from there.

## 3. Connect your assistant

**Claude Code**

```bash
claude mcp add optscan --scope user -e OPTSCAN_TOKEN=<your token> -- /usr/local/bin/optscan-mcp
```

**Claude Desktop**: add this to `claude_desktop_config.json` (Settings, Developer, Edit Config), then restart the app:

```json
{
  "mcpServers": {
    "optscan": {
      "command": "/usr/local/bin/optscan-mcp",
      "env": {
        "OPTSCAN_TOKEN": "<your token>"
      }
    }
  }
}
```

After restarting, your assistant should list the Optscan tools below.

## What it can do

| Tool | What it does | Plan |
|---|---|---|
| `optscan_get_quotes` | Real-time quotes for up to 25 symbols | Basic and Pro |
| `optscan_get_option_chain` | Option chain for a symbol, with Greeks and IV | Basic and Pro |
| `optscan_scan_iv` | IV rank and percentile scanner | Basic and Pro |
| `optscan_scan_premium_selling` | Put and call premium-selling candidates | Basic and Pro |
| `optscan_get_market_movers` | Top gainers, losers and market breadth | Basic and Pro |
| `optscan_get_top_activity` | Highest-volume option contracts per expiry for up to 10 symbols | Basic and Pro |
| `optscan_get_flow_alerts` | Unusual options flow alerts, filterable by ticker, premium, expiry and more. Basic plans see today's flow only. | Basic and Pro |
| `optscan_scan_flow_aggregate` | Per-ticker rolled-up options flow with a conviction score | Basic and Pro |
| `optscan_get_earnings` | Upcoming earnings with report time, expected move and flow sentiment. Basic plans see the current week. | Basic and Pro |
| `optscan_get_daily_option_history` | A ticker's day-by-day options activity: call and put volume, IV, and unusually large volume-versus-open-interest prints | Pro |

## Troubleshooting

When something is wrong with your account, the assistant should relay a short message with a code:

| Code | Meaning | What to do |
|---|---|---|
| `INVALID_TOKEN` | The token is wrong or was revoked. | Create a new token on the Account page and update your config. |
| `SUBSCRIPTION_REQUIRED` | Your subscription isn't active. | Renew at [optscan.io/pricing](https://optscan.io/pricing). |
| `FEATURE_REQUIRES_UPGRADE` | Your plan doesn't include that tool. | Upgrade to Pro at [optscan.io/pricing](https://optscan.io/pricing). |
| `QUOTA_EXCEEDED` | You've reached your plan's daily limit. | It resets at midnight UTC, or upgrade to Pro for unlimited calls. |

If the assistant says it can't start the server: check that the path in your config is right (`ls -l /usr/local/bin/optscan-mcp`), and that `OPTSCAN_TOKEN` is set inside the config's `env` block, not just in your terminal.

## Updating

Run the install command for your platform again, then restart your assistant. The latest version is always on the [Releases page](https://github.com/optscan-labs/optscan-mcp-releases/releases).

## Privacy and safety

- Read-only: no tool can place trades or change your account.
- Your token is stored only in your assistant's config on your own computer. Revoke it any time from the Account page.
- Questions or problems: use the feedback button in Optscan.
