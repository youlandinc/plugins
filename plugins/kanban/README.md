# Corepass Work

The team board at [work.corepass.com](https://work.corepass.com), as tools an
agent can call: the roster, the cards, the epics.

One hosted MCP server, no local install. On first use the agent opens a browser
to Corepass Work, you sign in with your ordinary board password and allow it, and
the token it gets back acts as **you** — cards it creates are authored by you and
show your name in their history, not a bot's.

## Tools

| | |
|---|---|
| `list_users` | Everyone who can be assigned a card |
| `board_summary` | How much work sits in each column, optionally for one person |
| `list_cards` | Cards, filtered by column, assignee, type, priority, epic or text |
| `get_card` | One card in full, with its recent history |
| `create_card` | File a card; only a title is required |
| `update_card` | Move, reassign, retitle, reprioritise, or restore from the archive |
| `archive_done` | Clear the Done column — hidden, never deleted |
| `delete_card` | Permanent, and refuses to run without an explicit confirmation |
| `list_epics` / `create_epic` | Epics, which here are time boxes with a start and an end date |

Assignees are named the way people name them: a full name, a unique fragment of
one, `me`, or `unassigned`. A fragment matching two teammates is an error listing
both rather than a guess.

## What it can reach

What your own account can reach, and nothing more. The team board, your private
board, and your epics — a private card stays visible only to its owner, because
the server applies the same rules to an agent that it applies to a person.

Access is revocable: the token appears with the client holding it, and revoking
it stops the agent mid-session.
