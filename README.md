# Personal Notes — WordPress Playground

A Notion-like personal knowledge base running locally via [WordPress Playground](https://wordpress.github.io/wordpress-playground/). The infrastructure is public; your notes are stored in a separate **private** Git repository that you clone locally as `./data/`.

Currently, there is [a proposal](https://adamadam.blog/2025/01/08/wordpress-as-a-git-repo/) from Adam Zieliński to make it possible for WordPress Playground to have content stored as Git backed Markdown files.
You can [follow the discussion on GitHub](https://github.com/WordPress/wordpress-playground/discussions/1524) about this, but it is not yet officially supported.

Until then, this repo is a kludge to create a repeatable Playground environment which can be public, with content stored in a separate Git repository that can be private.

## Architecture

| Layer | Technology |
|---|---|
| Runtime | WordPress Playground CLI (`@wp-playground/cli`) |
| Editor | WordPress block editor (Gutenberg) |
| Code blocks | Code Syntax Block plugin |
| Tables | TablePress plugin |
| Theme | Twenty Twenty-Five |
| Note storage | Private Git repo cloned at `./data/` |

WordPress Playground runs WordPress entirely in a local Node.js process (via WebAssembly). No PHP installation, no database daemon, no web server configuration required.

WP Playground caches the site at a deterministic path derived from the project directory (SHA-256 of the absolute path):

```
~/.wordpress-playground/sites/<sha256-of-project-path>/
```

`script/start` copies any saved database and uploads from `./data` into the site cache before launching. After a writing session, `script/save` copies them back out and commits to the private repo.

## How public/private separation works

```
scratch-wordpress/          ← this repo (public)
└── data/                   ← your private repo, cloned here (gitignored)
    ├── database/
    │   └── .ht.sqlite      ← SQLite database with all your notes
    ├── uploads/            ← media library
    ├── plugins/            ← custom plugins (each subdirectory is one plugin)
    └── themes/             ← custom themes (each subdirectory is one theme)
```

The public repo contains only infrastructure (scripts, blueprint, config). The `data/` directory is gitignored — no note content is ever stored in the public repo. Each user brings their own private repo.

Custom plugins and themes placed in `data/plugins/` and `data/themes/` are mounted individually into the WP Playground instance at startup, leaving the shared plugins installed via `blueprint.json` untouched.

## Prerequisites

- Node.js 18+ and npm 9+
- Git

## First-time setup

### 1. Clone this repo and install dependencies

```bash
git clone <this-repo-url>
cd scratch-wordpress
./script/setup
```

### 2. Create a private notes repo and clone it as `./data/`

Create a new **private** repository (e.g. `notes-data`) on GitHub or any Git host, then clone it into the `data/` directory:

```bash
git clone git@github.com:you/notes-data.git data
```

Or use the helper script which does this and sets up the initial structure:

```bash
./script/init-data git@github.com:you/notes-data.git
```

### 3. Start and write notes

```bash
./script/start
```

The site opens at `http://127.0.0.1:9400`. Admin panel at `/wp-admin` — credentials `admin` / `password`.

### 4. Save your notes

After a writing session, commit the database to your private repo:

```bash
./script/save
```

## Restoring notes on a new machine

```bash
git clone <this-repo-url>
cd scratch-wordpress
./script/setup
git clone git@github.com:you/notes-data.git data
./script/start
```

## Scripts

| Script | Purpose |
|---|---|
| `setup` | Install Node.js dependencies |
| `start` | Start WordPress Playground, mounting `./data` as the database |
| `stop` | Stop all running WordPress Playground instances |
| `reset` | Wipe WP Playground's cached state and rebuild from `blueprint.json` |
| `save` | Commit database and media library to the private repo |
| `init-data <url>` | One-time: clone a private repo as `./data/` |

npm shortcuts: `npm run setup`, `npm start`, `npm run reset`, `npm run save`.

## Configuration

`blueprint.json` controls what WordPress installs on first boot (and after `reset`):

- WordPress and PHP versions
- Plugins to install (Code Syntax Block, TablePress)
- Theme (Twenty Twenty-Five)
- Site title and description

Edit this file then run `./script/reset` to apply changes. Note that `reset` wipes and rebuilds from the blueprint, but your notes are safe because the database lives in `./data/` (your private repo), not in WP Playground's cache.

### Per-user overrides

Create a `blueprint-local.json` in the project root to customise your local setup without affecting other users. This file is gitignored. At startup, `script/start` merges it with `blueprint.json`:

- Top-level keys in `blueprint-local.json` override the base (e.g. change PHP version)
- Steps in `blueprint-local.json` are appended after the base steps

Example — switch to PHP 8.2 and install an extra plugin:

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

## File layout

```
.
├── blueprint.json          # WordPress Playground configuration (shared)
├── blueprint-local.json    # Per-user overrides — gitignored, optional
├── data/                   # Your private repo — database, media, custom plugins/themes (gitignored)
├── package.json
├── script/
│   ├── init-data           # One-time: clone private repo as ./data/
│   ├── merge-blueprints    # Merge blueprint.json + blueprint-local.json
│   ├── reset               # Wipe and rebuild WP Playground site
│   ├── save                # Commit and push the database
│   ├── setup               # Install dependencies
│   └── start               # Start dev server
└── README.md
```
