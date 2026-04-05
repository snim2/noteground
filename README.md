# Personal Notes — WordPress Playground

A Notion-like personal knowledge base running locally via [WordPress Playground](https://wordpress.github.io/wordpress-playground/). Site state is ephemeral in-browser; documents are backed by this Git repository.

## Architecture

| Layer | Technology |
|---|---|
| Runtime | WordPress Playground CLI (`@wp-playground/cli`) |
| Editor | WordPress block editor (Gutenberg) |
| Code blocks | Code Syntax Block plugin |
| Tables | TablePress plugin |
| Theme | Twenty Twenty-Four |
| Document store | Git (this repo) |

WordPress Playground runs WordPress entirely in a local Node.js process (via WebAssembly). No PHP installation, no database daemon, no web server configuration required.

## Prerequisites

- Node.js 18+ and npm 9+

## Quick start

```bash
# 1. Clone
git clone <repo-url>
cd scratch-wordpress

# 2. Install dependencies
./script/setup

# 3. Start the site
./script/start
```

The site opens automatically at `http://127.0.0.1:9400`.
Admin panel: `http://127.0.0.1:9400/wp-admin` — credentials `admin` / `password`.

Press `Ctrl+C` to stop.

Alternatively, use npm:

```bash
npm run setup   # install dependencies
npm start       # start the site
```

## Scripts

All scripts live in `script/` following the [Scripts to Rule Them All](https://github.com/github/script) convention.

| Script | Purpose |
|---|---|
| `setup` | Install Node.js dependencies |
| `start` | Start the WordPress Playground server |
| `reset` | Wipe stored site data and rebuild from `blueprint.json` |

## Configuration

`blueprint.json` controls what WordPress installs on first boot (and after `reset`):

- WordPress version and PHP version
- Site title and description
- Plugins to install and activate
- Theme to activate

Edit this file to change the site configuration, then run `./script/reset` to apply changes.

## Document storage (Git backing)

WordPress Playground persists site state in `.wordpress-playground/` (gitignored). To keep documents in Git:

1. Export posts/pages via **Tools → Export** in wp-admin and commit the `.xml` file under `content/`.
2. On a fresh clone, run `reset` then re-import via **Tools → Import**.

A future step will automate this with a WP-CLI blueprint step or a custom plugin.

## File layout

```
.
├── blueprint.json                 # WordPress Playground configuration
├── content/                       # Exported WordPress content (XML)
├── package.json
├── script/
│   ├── setup                      # Install dependencies
│   ├── start                      # Start dev server
│   └── reset                      # Wipe and rebuild site
└── README.md
```
