# agent-discord-wrapper

A Claude Code **skill** plugin. Pairs the external [Discord MCP server](https://github.com/HardHeadHackerHead/discord-mcp) with a skill so Claude can operate a Discord server — posting text and image attachments, managing channels, roles, threads and moderation — through structured MCP operations instead of hand-driving the Discord API.

The MCP server is declared inline in both plugin manifests. When the plugin is installed, the host agent launches it automatically via `npx` — no manual server setup. The server is pinned to an exact version (`@quadslab.io/discord-mcp@2.1.1`) so `npx` resolves the same cached package on every start.

## Prerequisites

### 1. Node.js 18+

`npx` (bundled with npm) launches the server on demand without a global install.

### 2. A Discord bot application

1. Create an application at https://discord.com/developers/applications, then add a **Bot** to it.
2. Under **Bot**, click **Reset Token** and copy it — this is the value the plugin asks for at install.
3. On the same page, enable **both** privileged gateway intents: **Server Members Intent** and **Message Content Intent**. The server requests both unconditionally on every connect; if either is off, Discord closes the connection with close code `4014` and every tool call fails.
4. Under **OAuth2 → URL Generator**, select the `bot` scope and at least: `View Channel`, `Send Messages`, `Attach Files`, `Embed Links`, `Read Message History`. For the full management surface this plugin exposes (channels, roles, threads, webhooks, moderation), also add `Manage Channels`, `Manage Roles`, `Manage Messages`, `Manage Webhooks`, `Manage Events`, `Manage Emojis and Stickers`, `Kick Members`, `Ban Members`, `Moderate Members`. Use the generated URL to invite the bot to your server.
5. Enable Developer Mode in Discord (User Settings → Advanced), then right-click your server's icon and **Copy Server ID** — this is the `guild_id` the plugin also asks for.

The server binds to exactly **one** guild per install; there is no per-tool guild parameter.

## Install

```
/plugin marketplace add Seretos/agent-marketplace
/plugin install agent-discord-wrapper@agent-marketplace
```

### Claude Code — the token and guild ID are prompted for you

The manifest declares `bot_token` and `guild_id` as [`userConfig`](https://code.claude.com/docs/en/plugins-reference#user-configuration) options, so Claude Code asks for them when the plugin is enabled and injects them into the server's environment. Nothing to hand-edit.

`bot_token` is marked `sensitive`, so it is stored in the OS keychain (or `~/.claude/.credentials.json` where no keychain is available) — **not** in `settings.json`, and never in the repository. `guild_id` is not a secret and lands in `settings.json` under `pluginConfigs`.

To change or re-enter either later:

```
/plugin config agent-discord-wrapper
```

Or non-interactively at install time:

```
claude plugin install agent-discord-wrapper@agent-marketplace --config bot_token=... --config guild_id=...
```

### Codex — set both values in the environment

Codex has no `userConfig` equivalent, so the Codex manifest ships **no** `env` block and the server inherits `DISCORD_TOKEN` / `DISCORD_GUILD_ID` from the process that launches it. Export them in the shell or profile that starts Codex:

```bash
export DISCORD_TOKEN=...
export DISCORD_GUILD_ID=...
```

```powershell
$env:DISCORD_TOKEN = "..."
$env:DISCORD_GUILD_ID = "..."
```

## Posting images and files

`send_message_with_file` accepts **either** an HTTPS URL **or an absolute local file path** in its `fileUrl` argument, despite the parameter name and its own schema description saying "URL". Pass `fileName` with the right extension (e.g. `screenshot.png`) so Discord renders it inline. For an image whose only source is a URL and you don't need it as a real attachment, `send_embed`'s `image`/`thumbnail` fields take a URL directly. See `skills/discord-wrapper/SKILL.md` for the full behavior.

## What the skill teaches

See `skills/discord-wrapper/SKILL.md`. It documents only the server's mechanics that are not obvious from the tool schemas — the single-guild binding, the intent requirement, the local-path behavior of `send_message_with_file`, a compact map of the ~140 tools by category, and troubleshooting. It deliberately does not prescribe use cases or workflows — those depend on what you're using the plugin for.

## Troubleshooting

**Every tool fails right after connecting, or the server won't stay connected.** Almost always close code `4014` — one or both privileged intents (Server Members, Message Content) are disabled in the Developer Portal. Enable both under Bot → Privileged Gateway Intents.

**"Missing Permissions" on a specific tool.** Run `npx -y @quadslab.io/discord-mcp@2.1.1 check` to see which permissions the invited bot is missing for the target guild, then re-invite with the missing scope or ask a server admin to grant the role.

**A channel-name argument hit the wrong channel.** Channel names are fuzzy-matched. Pass the channel ID instead — right-click the channel with Developer Mode enabled → Copy Channel ID.

**"Server not loaded" right after install.** Cold `npx` start: the package downloads before the first JSON-RPC byte, which can exceed the host's handshake timeout. Reconnect the MCP server (`/mcp` → reconnect) — it starts cleanly once npm has cached it. To warm the cache ahead of time:

```
npx -y @quadslab.io/discord-mcp@2.1.1 --help
```

Do not otherwise run the server manually — the host owns that process.
