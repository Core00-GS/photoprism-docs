PhotoPrism Docs CODEMAP

**Last Updated:** November 15, 2025

Purpose
- Provide a fast orientation for contributors touching docs.photoprism.app so they can locate Markdown sources, assets, templates, and build tooling without guesswork.
- Complement `AGENTS.md` by showing how MkDocs configuration, content folders, and deployment helpers relate to one another.

Quick Start
- Local workflow: run `make deps` (Debian/Ubuntu) or `make install` to create `venv/`, refresh with `make upgrade` when you need the latest dependencies, then `make watch` to serve http://localhost:8000/ with live reload. Use `make build` for a production render and `make deploy` only when explicitly asked to push via `mkdocs gh-deploy`.
- Branch flow: work in `develop`, review via pull requests, then merge into `deploy` (or run `make merge`) so the GitHub Actions pipeline rebuilds and publishes docs.photoprism.app. Keep `mkdocs.yml` and `mkdocs.deploy.yml` in sync before opening a PR.
- Container option: a `Dockerfile` exists (FROM `squidfunk/mkdocs-material:latest`), but it is currently unsupported because it previously broke; default to the Python virtualenv workflow unless the team revives container builds.

Tooling & Configuration
- `README.md` — contributor entry point with MkDocs overview, deployment expectations, and CLA reminder.
- `Makefile` — canonical command surface: environment setup (`deps`, `install`, `upgrade`), MkDocs lifecycle (`serve`, `build`, `deploy`), cleanup (`remove-venv`), Git helpers (`pull`, `push`, `merge`), and utilities such as `img-resize` (ImageMagick) for screenshot normalization.
- `requirements.txt` — Python dependencies (MkDocs ≥1.5, Material extensions, pymdown-extensions, social plugin requirements). Keep it aligned with the version installed by `make install`.
- `.env` (local) — optional environment overrides (proxies, custom paths, etc.); MkDocs Material Insiders now installs via PyPI so no token is needed.
- `Dockerfile` — legacy container definition not referenced by the Makefile; keep it in sync only if the team decides to reintroduce containerized builds.
- `mkdocs.yml` — main site configuration: navigation tree, metadata (`site_name`, `extra.social`), theme options (Material + custom palette), edit links (`edit_uri`), plugin set (`redirects`), and Markdown extensions (pymdownx suite, tooltips, task lists, mermaid, etc.).
- `mkdocs.deploy.yml` — inherits the base config and appends `search`, `privacy`, and the production redirect map plus asset mirrors. Use this when building or deploying the published site.
- `overrides/` — Material template overrides: `main.html` manages social/OG tags, favicon set, analytics (`a.photoprism.app`), and the banner, while `partials/copyright.html` replaces the footer.
- Generated output lives in `site/` (ignored). Running `mkdocs build` refreshes it; never edit the files manually.
- `venv/` contains the Python virtual environment; recreate it with `make install` if dependencies drift.

Content Map (docs/)
- Root landing pages: `index.md` (homepage content), `known-issues.md`, `release-notes.md`, and `credits.md` surface product-wide updates; update them whenever app releases ship or contributor acknowledgments change.
- Licenses: `docs/license/agpl.md`, `docs/license/apache.md`, and `docs/license/docs.md` provide legal text for software, libraries, and documentation terms.
- Shared media: `docs/img/` holds global hero images and badges, while section-specific `img/` folders (e.g., `docs/user-guide/img/`, `docs/developer-guide/img/`, and nested `img/` directories) store contextual screenshots. `docs/icons/` bundles favicons plus SVG/PNG logo variants alongside `docs/icons/LICENSE`.
- Styling: `docs/css/custom.css` contains site-wide overrides (typography tweaks, custom callouts, responsive helpers). Keep selectors scoped to avoid clashing with Material defaults.

Getting Started Section (`docs/getting-started/`)
- Entry page: `index.md` introduces installation paths; `config-options.md` and the `config-files/` folder document `options.yml`, `defaults.yml`, and `settings.yml`.
- Deployment guides: `docker-compose.md`, `docker.md`, and subdirectories for `cloud/` (PikaPods, DigitalOcean), `nas/` (Asustor, Synology, QNAP, Unraid, OpenMediaVault), `ports/` (FreeBSD), and `raspberry-pi/` (requirements + microSD image).
- Operations: `advanced/` covers OIDC, transcoding, Kubernetes, volumes, backups, DB migrations, caching, proxying, and scaling. `proxies/` details Traefik, Caddy, NGINX, Apache, SWAG, HAProxy. `vpn/` contains Tailscale guidance.
- Troubleshooting: `troubleshooting/` hosts logs, Docker, MariaDB, SQLite, Windows, browser, metadata, performance, firewall, and Raspberry Pi checklists. `faq.md`, `using-https.md`, `using-a-cdn.md`, and `updates.md` round out the setup flow.

User Guide Section (`docs/user-guide/`)
- Core navigation: `index.md`, `first-steps.md`, and `navigate.md` explain the UI and onboarding experience.
- Feature deep dives live under `organize/` (albums, people, stacks, labels, archive, download, etc.), `search/` (index, filters, result views), `library/` (originals, imports, duplicates, metadata, upload, WebDAV, files), and `settings/` (general, library, advanced, services, account).
- Specialized areas: `sync/` (WebDAV, mobile, Dropbox integrations), `backups/` (create, restore, external storage, directory overview), `share/`, `users/` (admin UI, CLI, roles, guests, multi-library, 2FA, client credentials), `use-cases/` (Google, Apple, Flickr migrations), `pwa.md`, and `native-apps/` describe remote clients.

Developer Guide Section (`docs/developer-guide/`)
- Orientation: `index.md`, `setup.md`, `directories.md`, `issues.md`, `pull-requests.md`, `reviewing-pull-requests.md`, `code-quality.md`, `tests.md`, `configuration.md`, `documentation.md`, and `faq.md` establish workflow norms.
- Localization: `translations.md` and `translations-weblate.md` document string workflows.
- Media stack: `media/` (formats, raw, HEIC, video, live photos, thumbnails, storage), `metadata/` (XMP, EXIF extraction/editing, geocoding, orientation, perceptual hashes, color detection, camera models).
- Vision & AI: `vision/` (face recognition, label/caption generation, model comparison, CLI, TensorFlow adapters, `vision/service/` playground + setup).
- API & integrations: `api/` (intro, docs, auth, OIDC, OAuth2, search, thumbnails, Go client), `native-apps/` (platform guides + device resolutions), `ui/` (introduction, screenshots, components, design/colors, browsers, maps, infinite scrolling, focus management).
- Data layer: `database/` (index, migrations, ER diagram) and `technologies/` (YAML, Go, TensorFlow, Docker, Broadway, external APIs) describe dependencies; `security/` covers testing, Go secure coding, and policy.

About & Miscellaneous
- `docs/credits.md` thanks contributors and sponsors.
- `docs/license/` feeds the Licenses navigation tab configured under "About" in `mkdocs.yml`.
- `todo/` stores drafts not yet linked (currently `developer-guide/setup-fedora.md`). Move completed drafts into `docs/` and delete the placeholder.

Assets & Overrides
- `docs/icons/` contains favicon sizes referenced from `overrides/main.html`; ensure new icons follow the same naming scheme before linking them in the template.
- `docs/img/` and nested folders include `LICENSE` files for creative assets. Keep alt text synchronized with filenames to aid accessibility.
- `overrides/main.html` adjusts `<head>` metadata, Open Graph/Twitter cards, favicon declarations, and injects the membership CTA banner plus outbound link analytics script. When editing, verify both light/dark themes and ensure new domains are approved.
- `overrides/partials/copyright.html` renders the footer. Update it when legal wording or attribution changes.

Drafting & Maintenance Tips
- Every new page must be registered in `mkdocs.yml`; if you relocate or rename a file, add redirect entries to both `mkdocs.yml` and `mkdocs.deploy.yml`.
- Keep filenames lowercase with hyphens (`snake-case.md`) to match existing URLs; directories mirror navigation labels.
- Before committing images, run `make img-resize` where applicable and store assets alongside their Markdown.
- Use admonitions, tabs, and tooltips from the configured Markdown extensions instead of custom shortcodes; this keeps the content portable to Read the Docs and GitHub previews.
