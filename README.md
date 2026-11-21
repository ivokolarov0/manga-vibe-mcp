# Remote control (MCP) for MangaVibe

**Remote control** lets an assistant use [MangaVibe](https://github.com/ivokolarov0/manga-vibe) for
you: Claude, or any app that speaks **MCP** (the Model Context Protocol). It can search and filter your
library, open a series, start or continue reading, turn pages, change settings, and press any button
on a screen where MangaVibe is open, and it can say what is on that screen right now. It is the base
for **voice control**: the [Manga Accessibility](https://github.com/ivokolarov0/manga-vibe-ext-accessibility)
plugin can build on it so someone can run the whole app by voice.

It is **off** until you switch it on.

## What you need

- **A MangaVibe release newer than 2.17.0** (Plugin API 13). MangaVibe 2.17.0 still has Remote control
  built in, under the same id, so it keeps its own and this one isn't loaded (Settings → Plugins says
  why). Until the next release, use a checkout of MangaVibe's `main`.

## Install

The plugin is the `mcp/` folder. Copy it into MangaVibe's data folder as `plugins/mcp/`:

1. Download this repo (**Code → Download ZIP**, or the newest release's source zip) and unpack it.
2. Copy the `mcp` folder into the `plugins` folder of your data folder: `docker/mangavibe/plugins/mcp`
   on a Synology, or `plugins/mcp` inside the folder you mounted as `/data` (create `plugins` if it
   isn't there). For a local checkout: `<DATA_DIR>/plugins/mcp/` (for `preview.py`,
   `dev-data/plugins/mcp/`).
3. In MangaVibe: **Settings → Plugins → Reload plugins** (or restart the app).

For development on Windows, a directory junction saves copying after each change:

```
mklink /J D:\projects\manga-vibe\dev-data\plugins\mcp D:\projects\manga-vibe-mcp\mcp
```

## Updates

**Settings → Plugins → Remote control (MCP) → Updates** (Check for updates, Update to vX). It installs
the `mcp/` folder from the newest release's source zip. Your keys are kept.

- **To publish a version:** raise `version` in `mcp/__init__.py`, push it, then create a release
  tagged `v<version>` (e.g. `gh release create v1.1.0 --generate-notes`). Nothing is offered before
  the release.
- An installed copy that is a link to a checkout (the junction above) isn't updated this way: update
  the checkout with git.

## Switch it on

1. **Settings → Plugins → Remote control (MCP)**: switch it on.
2. **Settings → Remote control** (every account has its own page): type a name, e.g. *Claude on my
   laptop*, and press **Make a key**. Copy the key: it is shown only this once.
3. Give your assistant the **address** shown there (`http://<MangaVibe's address>/mcp`) and the key.

Make one key per assistant, so you can delete one without the others. **The key acts as you**: it sees
your libraries and your reading, and can do only what you can. With Accounts on, a key made by
someone who isn't an admin can't change the app's settings, folders or plugins, and deleting an
account deletes its keys. A locked library stays locked to the assistant.

## Connect an assistant

**Claude Code** (on the computer, in a terminal):

```bash
claude mcp add --transport http mangavibe http://192.168.1.215:8080/mcp --header "Authorization: Bearer mvk_1_..."
```

**Apps that read an `mcpServers` file** (Cursor, VS Code, many others) take what the settings page
shows after you make a key:

```json
{ "mcpServers": { "mangavibe": { "type": "http", "url": "http://192.168.1.215:8080/mcp",
  "headers": { "Authorization": "Bearer mvk_1_..." } } } }
```

**Claude Desktop** starts local servers only, so use the small `mcp-remote` bridge (needs Node.js):

```json
{ "mcpServers": { "mangavibe": { "command": "npx", "args": ["-y", "mcp-remote",
  "http://192.168.1.215:8080/mcp", "--allow-http", "--header", "Authorization:Bearer mvk_1_..."] } } }
```

An app that can't send headers can put the key in the address instead: `.../mcp?key=mvk_1_...`. Use
the header when you can: an address is easily logged in full. MangaVibe keeps the key out of its own
log, but a reverse proxy in front of it, or the assistant app's own logs and settings, may write it
down as it is.

**From a phone or another computer**, use the address you open MangaVibe with on that device: the
NAS's address on your home network, or over your VPN (see MangaVibe's
[Reading away from home](https://github.com/ivokolarov0/manga-vibe/blob/main/docs/away-from-home.md)).
Claude's website and phone app connect from Anthropic's servers, which can't reach a server at home,
so they can't use it. Don't open MangaVibe to the internet for this.

## What it can do

**Without a screen** (answers from the server):

| Tool | |
|---|---|
| `library_filters` | the filters and what each can be set to, with the genres and tags in your library |
| `library_search` | search and filter the library (status, genres, tags, format, length, letter, sort...) |
| `continue_reading` | what you're in the middle of, with the book and page to go on with |
| `series_get` | a series' details and its books in order, with how far each is read |
| `series_update`, `mark_read` | reading status, favorite, reading mode; mark books read or unread |
| `settings_get`, `settings_change` | the settings, what each can be, which you may change |
| `api_request` | anything else, through MangaVibe's own API, as you |

**On a screen** where MangaVibe is open (a browser tab, the Android app) and logged in as you:

| Tool | |
|---|---|
| `screens_list` | your open screens; tools use the one you used last unless told which |
| `screen_describe` | what is on the screen: the page, what is open over it, the reader's state, and every button, link, field and switch with a number (`ref`) |
| `screen_press`, `screen_fill`, `screen_key`, `screen_scroll` | press a button by its ref, type into a field or pick an option, press a key (Escape, arrows), scroll |
| `screen_navigate`, `screen_library` | go to a page (library, a series, Continue, Settings, reading a book); show the library with filters |
| `reader_state`, `reader_turn`, `reader_go_to`, `reader_mode`, `reader_options`, `reader_plugin`, `reader_close` | the reader: where you are, turn pages, go to a page or the next book, pages or vertical, the reading options, plugins' buttons and overlays, leave |

`screen_describe` and `screen_press` reach **every** part of the app, including what plugins add, so
anything you can do with a tap can be done by voice too. Plugins can also add tools of their own; they
are named after the plugin, e.g. `saved_searches__show` and `libraries__pick` (see MangaVibe's
[PLUGINS.md](https://github.com/ivokolarov0/manga-vibe/blob/main/docs/PLUGINS.md), *Assistants and voice
control*).

Screens ask MangaVibe for commands every few seconds while they are open; a closed tab or a phone
that is off simply stops being listed.

## Security

- A key is as good as your login. Keep it like a password; delete it in **Settings → Remote
  control** when an assistant no longer needs it. Only a fingerprint of it is stored.
- An assistant does what it is told, and it can be told things by what it reads (a web page, a
  document). If it has other tools too, give it a key of an account that isn't an admin.
- Web pages from other sites can't use `/mcp` from your browser: such calls are refused.
- Switched off, `/mcp` doesn't answer at all.

## Tests

The plugin brings its own tests (`tests/`). They start the MangaVibe app with the plugin installed in
a temporary data folder, next to the app's small generated test library, exactly as an owner installs
it.

The app is looked for next to this repo (`../manga-vibe`), or set `MANGAVIBE_DIR` to its checkout.
Run them with the app's Python, which has its packages:

```
..\manga-vibe\.venv\Scripts\python -m pytest        # Windows
../manga-vibe/.venv/bin/python -m pytest            # Linux/macOS
```

Run them after changing the plugin, and after updating MangaVibe. `test_tool_lists_match_the_page`
fails when MangaVibe gets a new setting, or its built-in library filters or sorts change: add it to
`SETTINGS`, `FILTERS` or `SORTS` in `mcp/tools.py`.

**The library's filters and sorts** come from MangaVibe itself (`GET /api/library/schema`, Plugin API
16): they're a plugin's, which the owner can replace or extend, and `library_filters`,
`library_search` and `screen_library` follow whichever plugin does them. A plugin's own filters are
listed under `more_filters` and given in `more`. On a MangaVibe from before that, the tools use the
copies of the built-in ones in `mcp/tools.py` (`FILTERS`, `SORTS`).

## Layout

| Path | What it is |
|------|------------|
| `mcp/__init__.py` | the plugin (id `mcp`), its version and repo |
| `mcp/keys.py` | the keys, and the middleware in front of `/mcp` |
| `mcp/protocol.py` | MCP's JSON-RPC over HTTP |
| `mcp/tools.py` | the tools: data ones through the app's own API, screen ones through `screens.py` |
| `mcp/screens.py` | the open pages that wait for commands (long polling) |
| `mcp/api.py` | its API at `/api/plugins/mcp/api/...` |
| `mcp/mcp.js`, `settings.html`, `mcp.css` | the page's side (a "screen"), Settings → Remote control |
| `tests/` | the tests (see above) |

Why it works the way it does (keys, long polling, data tools through the app's API) is in MangaVibe's
[DECISIONS.md](https://github.com/ivokolarov0/manga-vibe/blob/main/DECISIONS.md), *Remote control*.

## License

MIT, see [LICENSE](LICENSE).
