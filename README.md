# Personal Notes — WordPress Playground

A Notion-like personal knowledge base running locally via [WordPress Playground](https://wordpress.github.io/wordpress-playground/). No PHP installation, no database daemon, no web server configuration required — WordPress runs entirely in your browser.

This repo is public, but your notes live in a separate **private** Git repository that you clone locally as `./data/`.

Note that there is [a proposal](https://adamadam.blog/2025/01/08/wordpress-as-a-git-repo/) from Adam Zieliński to make it possible for WordPress Playground to have content stored as Markdown files.
You can [follow the discussion on GitHub](https://github.com/WordPress/wordpress-playground/discussions/1524) about this, but it is not yet officially supported.

## Quickstart

### Prerequisites

- Node.js 18+ and npm 9+
- Git

### Clone this repo and install dependencies

```bash
git clone <this-repo-url>
cd scratch-wordpress
./script/setup
```

### Create a private notes repo

Create a new **private** repository on GitHub or any Git host, then clone it:

```bash
./script/init-data git@github.com:you/notes-data.git
```

### Start writing

```bash
./script/start
```

The site opens at `http://127.0.0.1:9400` by default. Admin panel at `/wp-admin` — credentials `admin` / `password`.

### Save your notes

After a writing session, press Ctrl+C. You'll be asked whether to save — or run at any time:

```bash
./script/save
```

This commits the database and media library to your private repo and pushes.

## Restoring notes on a new machine

```bash
git clone <this-repo-url>
cd scratch-wordpress
./script/setup
git clone git@github.com:you/notes-data.git data
./script/start
```

## Running multiple notebooks simultaneously

You can run as many independent WordPress instances as you like from a single checkout using the `NOTES_PROFILE` environment variable. Each profile gets its own data directory, WP Playground site cache, and port (auto-detected from 9400 upward).

```bash
# Set up a named profile
NOTES_PROFILE=work ./script/init-data git@github.com:you/notes-work.git

# Start it (open another terminal for a second instance)
NOTES_PROFILE=work ./script/start

# Save after a session
NOTES_PROFILE=work ./script/save
```

Profile names must contain only letters, numbers, and hyphens (e.g. `work`, `research-2`).

Each profile stores data in its own gitignored directory (`./data-work/`, `./data-research/`, etc.), each backed by its own private repo. No note content ever touches this public repo.

## Configuration

`blueprint.json` controls what WordPress installs on first boot (plugins, theme, PHP version, site title). Edit it then run `./script/reset` to rebuild. Your notes are safe — the database lives in `./data/`, not in WP Playground's cache.

### Per-user overrides

Create a gitignored `blueprint-local.json` to customise your local setup without affecting others. Top-level keys override the base; steps are appended.

```json
{
  "preferredVersions": { "php": "8.2" },
  "steps": [
    {
      "step": "installPlugin",
      "pluginData": { "resource": "wordpress.org/plugins", "slug": "classic-editor" }
    }
  ]
}
```

## Scripts

| Script | Purpose |
|---|---|
| `setup` | Install dependencies and check prerequisites |
| `start` | Start WordPress Playground |
| `save` | Commit database and media library to the private repo |
| `reset` | Wipe WP Playground's cached state and rebuild from `blueprint.json` |
| `stop` | Stop all running WordPress Playground instances |
| `init-data <url>` | One-time: clone a private repo as `./data/` |
| `test` | Run lints and schema validation |

`npm` shortcuts are available for all scripts: `npm start`, `npm run save`, etc.

## How privacy works

```plaintext
scratch-wordpress/          ← this repo (public)
└── data/                   ← your private repo, cloned here (gitignored)
    ├── database/
    │   └── .ht.sqlite      ← SQLite database with all your notes
    ├── uploads/            ← media library
    ├── plugins/            ← custom plugins
    └── themes/             ← custom themes
```

The public repo contains only infrastructure. The `data/` directory (and any `data-*/` profile directories) are gitignored.

## File layout

```plaintext
.
├── blueprint.json          # WordPress Playground configuration (shared)
├── blueprint-local.json    # Per-user overrides — gitignored, optional
├── data-*/                 # Your private repo(s) — gitignored
├── package.json
├── schemas/
│   └── blueprint.json      # JSON schema for blueprint files
└── script/
    ├── lib/
    │   └── profile         # Shared profile resolution for multi-notebook support
    ├── bootstrap            # Install system dependencies (macOS/Homebrew)
    ├── init-data            # One-time: clone private repo as ./data/
    ├── merge-blueprints     # Merge blueprint.json + blueprint-local.json
    ├── reset                # Wipe and rebuild WP Playground site
    ├── save                 # Commit and push database and media
    ├── setup                # Install dependencies
    ├── start                # Start dev server
    ├── stop                 # Stop all running Playground instances
    └── test                 # Run linters and tests
```
