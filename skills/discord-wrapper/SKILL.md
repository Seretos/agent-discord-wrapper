---
name: discord-wrapper
description: Operate a Discord server through the Discord MCP server - send text messages and image/file attachments, read channel history, manage channels, roles, threads, webhooks and moderation. Use when the user wants to post something to Discord, share a screenshot or generated image in a channel, check what was said in a Discord channel, or manage a Discord server's channels, roles or threads. Also for requests like "post this to Discord", "send this image to the channel", "was hat jemand in Discord geschrieben", "poste das Bild in Discord", "leg einen Discord-Kanal an", "Discord-Rolle vergeben".
---

# discord-wrapper

Drives a Discord server through the [Discord MCP server](https://github.com/HardHeadHackerHead/discord-mcp) (`@quadslab.io/discord-mcp`): roughly 140 tools across 20 categories, covering messages, channels, roles, members, threads, forums, webhooks, moderation, events and more.

This skill does not prescribe workflows or use cases — those depend on what the plugin is being used for. What follows is only the mechanics that are not obvious from the tool schemas: the connection model, and the one place a tool's own description is misleading.

## Connection model

- **One guild per connection.** The server is configured with a single `DISCORD_GUILD_ID` at connect time; there is no per-tool guild parameter. All tool calls operate on that one Discord server.
- **Two privileged gateway intents are always required:** Server Members and Message Content. If either is disabled in the bot's Developer Portal settings, the gateway connection closes with code `4014` and every tool call fails, not just the ones that would need the intent. This is a Discord-side bot configuration, not something a tool call can work around — if everything is failing at once, check intents before anything else.
- **The bot only has the permissions it was invited with.** A missing permission surfaces as a normal tool error, not a connection failure. Run `npx -y @quadslab.io/discord-mcp@2.1.1 check` (in a terminal, not as an MCP tool call) to see the invited bot's actual permissions against the connected guild.

## Sending attachments — read this before posting an image

Two tools can carry an image; they are not interchangeable.

- **`send_message_with_file`** — sends a real file attachment. Its `fileUrl` argument accepts **either an HTTPS URL or an absolute local filesystem path**, despite the parameter name and the tool's own schema description ("URL of the file to attach"). Internally it resolves anything that isn't `http(s)://` as a path on disk. To post a locally generated image (a screenshot, a render, a chart) pass its absolute path directly as `fileUrl`, and set `fileName` with a real image extension (e.g. `render.png`) so Discord renders it inline rather than as a generic file chip.
- **`send_embed`** — its `image` and `thumbnail` fields take a URL only. There is no local-path option here. Use this for a rich card (title, description, color, fields, footer) that references an image already hosted somewhere, not for uploading a local file.

If asked to "post this image" and the image is a local file with no public URL, `send_message_with_file` with the local path is the only correct tool — do not attempt to invent a URL or describe the image in text instead.

## Addressing channels

Channel arguments accept a name, a mention, or an ID, and names are matched fuzzily. A close-but-wrong name can silently resolve to the wrong channel. Prefer passing the channel ID once it's known (from a prior `get_messages`/list call, from the user, or from Developer Mode → right-click channel → Copy Channel ID) over repeatedly matching by name.

## Tool categories

The exact tool list is visible in the connected MCP session (139 tools total); this is the map for choosing *where* to look, not a substitute for reading a tool's own schema before calling it.

| Category | Count | Covers |
|---|---|---|
| Messages | 14 | `send_message`, `send_embed`, `send_message_with_file`, `get_messages`/`get_message`, `edit_message`, `delete_message`, `bulk_delete_messages`, `crosspost_message`, pin/unpin, reactions |
| Channels | 20 | create/edit/delete channels, categories, permission overwrites, channel settings |
| Roles | 11 | create/edit/delete roles, assign/remove role on a member, role permissions |
| Members | 15 | list/inspect members, kick, ban, timeout, nickname, role membership |
| Threads | 15 | create/archive/manage threads, thread members |
| Forums | 5 | forum channel posts and tags |
| Server Admin | 16 | guild settings, audit log, invites, bans list |
| Webhooks | 4 | create/list/send-via/delete webhook |
| Emojis & Stickers | 7 | create/list/delete, including from an image URL |
| Scheduled Events | 5 | create/list/manage server events |
| AutoMod | 4 | automod rule management |
| Polls | 3 | create/inspect a native Discord poll |
| Stage | 3 | stage-channel controls |
| Guild | 2 | guild-level info |
| DMs | 2 | direct messages |
| Bot Presence | 2 | set the bot's status/activity |
| Templates | 4 | server templates |
| App Commands | 4 | slash-command management |
| Onboarding | 2 | server onboarding flow |
| Reactions | 1 | (also reachable from Messages) |

There are also three MCP resources — `discord://server/summary`, `discord://server/channels`, `discord://server/roles` — for a quick read of server state without a tool call.

## Destructive tools

Several tools are irreversible or affect many messages/members at once: `bulk_delete_messages`, `delete_channel`, `delete_message` (past Discord's edit window it cannot be recovered), ban/kick tools, role deletion, and anything under Server Admin that changes guild-wide settings. Confirm the target and scope with the user before calling one of these — the MCP layer has no undo.

## Troubleshooting

**Everything fails immediately after connecting.** Close code `4014` — a privileged intent is off. Fix it in the Discord Developer Portal (Bot → Privileged Gateway Intents → enable both), not in the tool call.

**A specific tool returns a permissions error.** Run `npx -y @quadslab.io/discord-mcp@2.1.1 check` from a terminal to see what the bot actually has in this guild, then have a server admin grant the missing permission or re-invite the bot with a wider scope.

**A local image didn't post as an inline image.** Check `fileName` has an image extension, and that the path passed to `fileUrl` is absolute, not relative to some assumed working directory.

**"Server not loaded" right after install.** Cold `npx` start downloading the package can exceed the host's handshake timeout. Reconnect the MCP server; it starts cleanly once npm has cached the package.
