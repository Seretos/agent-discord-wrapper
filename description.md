# agent-discord-wrapper

Turns Discord into a channel the agent can actually operate. Post updates, screenshots
or generated images straight into a channel, read back what's been said, and manage the
server — channels, roles, threads, webhooks — without leaving the conversation.

## Key features

- **Zero-setup MCP server** — the external Discord MCP server is declared inline in the
  plugin manifest and launched via `npx` on install. No manual server, no global install,
  no config file.
- **Bot token and server ID prompted at install** — both are `userConfig` options, so
  Claude Code asks for them when you enable the plugin. The token is stored in your OS
  keychain; nothing to hand-edit, nothing that can end up in a repository.
- **Text and image attachments, not just text** — post a message with a real file
  attachment (local path or URL), not only a link or an embed thumbnail.
- **Full server management** — channels, roles, threads, forums, webhooks, scheduled
  events, emojis and stickers, moderation and AutoMod, polls, and more, all through one
  MCP server.
- **Works in Claude Code and Codex** — ships both a `.claude-plugin` and a `.codex-plugin`
  manifest from one repository.

## Requirements

- **Node.js 18+** on the host machine; `npx` launches the server on demand.
- A **Discord bot application** with both privileged gateway intents (Server Members,
  Message Content) enabled, invited to the target server with the permissions it needs.
- Codex users set `DISCORD_TOKEN` / `DISCORD_GUILD_ID` in the environment; Claude Code
  prompts for both.

## A good fit if you want to

- Have the agent post status updates, build results, or generated assets into a Discord
  channel.
- Manage a Discord server's channels, roles and threads conversationally.
- Read back recent messages or reactions without opening Discord.
