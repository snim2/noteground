# Personal Notes — WordPress Playground

A Notion-like personal knowledge base running locally via [WordPress Playground](https://wordpress.github.io/wordpress-playground/). The infrastructure is public; your notes are stored in a separate **private** Git repository linked as a submodule.

## Architecture

| Layer | Technology |
|---|---|
| Runtime | WordPress Playground CLI (`@wp-playground/cli`) |
| Editor | WordPress block editor (Gutenberg) |
| Code blocks | Code Syntax Block plugin |
| Tables | TablePress plugin |
| Theme | Twenty Twenty-Five |
| Note storage | Private Git repo (submodule at `./data/`) |

WordPress Playground runs WordPress entirely in a local Node.js process (via WebAssembly). No PHP installation, no database daemon, no web server configuration required.

WP Playground caches the site at a deterministic path derived from the project directory (SHA-256 of the absolute path):

```
~/.wordpress-playground/sites/<sha256-of-project-path>/
```

`script/start` copies any saved database and uploads from `./data` into the site cache before launching. After a writing session, `script/save` copies them back out and commits to the private repo.

## How public/private separation works

```
scratch-wordpress/          ← this repo (public)
└── data/                   ← git submodule → your-notes-data (private)
    ├── database/
    │   └── .ht.sqlite      ← SQLite database with all your notes
    └── uploads/            ← media library
```

The public repo contains only infrastructure (scripts, blueprint, config). The `data/` submodule records a single commit hash pointing into your private repo — no note content is ever stored in the public repo.

## Prerequisites

- Node.js 18+ and npm 9+
- Git

## First-time setup

### 1. Install dependencies

```bash
git clone <this-repo-url>
cd scratch-wordpress
./script/setup
```

### 2. Create a private notes repo and link it

Create a new **private** repository (e.g. `notes-data`) on GitHub or any Git host, then:

```bash
./script/init-data git@github.com:you/notes-data.git
```

This adds `data/` as a submodule and commits `.gitmodules` to the public repo.

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

Then push the public repo to record the updated submodule pointer:

```bash
git push
```

## Restoring notes on a new machine

```bash
git clone <this-repo-url>
cd scratch-wordpress
./script/setup        # clones the data submodule automatically
./script/start
```

## Scripts

| Script | Purpose |
|---|---|
| `setup` | Install Node.js dependencies; init data submodule if configured |
| `start` | Start WordPress Playground, mounting `./data` as the database |
| `reset` | Wipe WP Playground's cached state and rebuild from `blueprint.json` |
| `save` | Commit database and media library to the private repo; update submodule pointer |
| `init-data <url>` | One-time: link a private repo as the `./data` submodule |

npm shortcuts: `npm run setup`, `npm start`, `npm run reset`, `npm run save`.

## Configuration

`blueprint.json` controls what WordPress installs on first boot (and after `reset`):

- WordPress and PHP versions
- Plugins to install (Code Syntax Block, TablePress)
- Theme (Twenty Twenty-Five)
- Site title and description

Edit this file then run `./script/reset` to apply changes. Note that `reset` wipes and rebuilds from the blueprint, but your notes are safe because the database lives in `./data/` (the private submodule), not in WP Playground's cache.

## File layout

```
.
├── blueprint.json          # WordPress Playground configuration
├── data/                   # Private submodule — database + media library
├── package.json
├── script/
│   ├── init-data           # One-time submodule setup
│   ├── reset               # Wipe and rebuild WP Playground site
│   ├── save                # Commit and push the database
│   ├── setup               # Install dependencies
│   └── start               # Start dev server
└── README.md
```
