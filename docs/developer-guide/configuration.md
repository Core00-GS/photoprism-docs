# Configuration

PhotoPrism’s runtime configuration is managed by the [`internal/config`](https://github.com/photoprism/photoprism/tree/develop/internal/config) package. Defaults are defined in [`options.go`](https://github.com/photoprism/photoprism/blob/develop/internal/config/options.go) and then merged with values from command-line flags, environment variables, and optional YAML files (`storage/config/*.yml`).

This page summarizes the tooling available to inspect those values and highlights the defaults that ship with the developer `compose.yaml` stack referenced in [Build Setup](setup.md).

## CLI Reference

- `photoprism help` (or `photoprism --help`) lists all subcommands and global flags.
- `photoprism show config` renders every active option along with its current value. Pass `--json`, `--md`, `--tsv`, or `--csv` to change the output format.
- `photoprism show config-options` prints the description and default value for each option. Use this when updating [Config Options](../getting-started/config-options.md).
- `photoprism show config-yaml` displays the configuration keys and their expected types in the same structure that the YAML files use. It is a read-only helper meant to guide you when editing files under `storage/config`.
- Additional `show` subcommands document search filters, metadata tags, and supported thumbnail sizes; see [`internal/commands/show.go`](https://github.com/photoprism/photoprism/blob/develop/internal/commands/show.go) for the complete list.

Commands can be executed on the host or inside any running container. With Docker Compose (development stack) prefix commands with `docker compose exec photoprism …`.

## Sources and Precedence

PhotoPrism loads configuration in the following order:

1. **Built-in defaults** defined in [`internal/config/options.go`](https://github.com/photoprism/photoprism/blob/develop/internal/config/options.go).
2. **`defaults.yml`** — optional system defaults (typically `/etc/photoprism/defaults.yml`). See [Global Defaults](../getting-started/config-files/defaults.md) if you package PhotoPrism for other environments and need to override the compiled defaults.
3. **Environment variables** prefixed with `PHOTOPRISM_…`. The list mirrors [Config Options](../getting-started/config-options.md). In Docker/Compose this is the primary override mechanism.
4. **`options.yml`** — user-level configuration stored under `storage/config/options.yml` (or another directory controlled by `PHOTOPRISM_CONFIG_PATH`). Values here override both defaults and environment variables, see [Config Files](../getting-started/config-files/index.md).
5. **Command-line flags** (for example `photoprism --cache-path=/tmp/cache`). Flags always win when a conflict exists.

The `PHOTOPRISM_CONFIG_PATH` variable controls where PhotoPrism looks for YAML files (defaults to `storage/config`). Keeping this path inside the repo makes it easy to track changes or share sample configs with teammates.

!!! note
    Any change to configuration (flags, env vars, YAML files) requires a restart. The Go process reads options during startup and does not watch for changes.

## Development Defaults (`compose.yaml`)

The root-level [`compose.yaml`](https://github.com/photoprism/photoprism/blob/develop/compose.yaml) provides a test and development environment that mirrors the services used in CI. The most important values are summarized below so you can quickly orient yourself or override them in a local `.env` file:

| Purpose     | Variable / Setting                                                                                                                                 | Default in `compose.yaml`                                                                                                                                                   |
|-------------|----------------------------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Admin login | `PHOTOPRISM_ADMIN_USER` / `PHOTOPRISM_ADMIN_PASSWORD`                                                                                              | `admin` / `photoprism`                                                                                                                                                      |
| URL & ports | `PHOTOPRISM_SITE_URL`, `PHOTOPRISM_HTTP_HOST`, `PHOTOPRISM_HTTP_PORT`, `PHOTOPRISM_DISABLE_TLS`                                                    | `https://app.localssl.dev/`, `0.0.0.0`, `2342`, TLS disabled (handled by Traefik)                                                                                           |
| Features    | `PHOTOPRISM_DEBUG`, `PHOTOPRISM_EXPERIMENTAL`, `PHOTOPRISM_READONLY`, `PHOTOPRISM_HTTP_MODE`                                                       | `true`, `true`, `false`, `debug`                                                                                                                                            |
| Paths       | `PHOTOPRISM_STORAGE_PATH`, `PHOTOPRISM_ORIGINALS_PATH`, `PHOTOPRISM_IMPORT_PATH`                                                                   | `/go/src/github.com/photoprism/photoprism/storage`, `/go/src/github.com/photoprism/photoprism/storage/originals`, `/go/src/github.com/photoprism/photoprism/storage/import` |
| Database    | `PHOTOPRISM_DATABASE_DRIVER`, `PHOTOPRISM_DATABASE_SERVER`, `PHOTOPRISM_DATABASE_NAME`, `PHOTOPRISM_DATABASE_USER`, `PHOTOPRISM_DATABASE_PASSWORD` | `mysql`, `mariadb:4001`, `photoprism`, `root`, `photoprism`                                                                                                                 |
| Services    | Dummy WebDAV (`dummy-webdav`), Keycloak (`dummy-oidc`), LDAP (`dummy-ldap`), Qdrant, Traefik                                                       | Pre-configured hostnames under `*.localssl.dev`                                                                                                                             |
| Auth demos  | `PHOTOPRISM_OIDC_*` values point to the bundled Keycloak realm; `PHOTOPRISM_LDAP_*` targets the dummy LDAP container.                              |

Additional knobs in the compose file enable AI features (`PHOTOPRISM_VISION_*`), tune media processing (`PHOTOPRISM_FFMPEG_*`), and configure hardware acceleration. For day-to-day frontend/backend work you rarely need to change these values; overriding `PHOTOPRISM_STORAGE_PATH` or `PHOTOPRISM_SITE_URL` in `.env` is usually sufficient.

The compose stack mounts the repository itself into `/go/src/github.com/photoprism/photoprism` and binds `./storage` from the host to `/photoprism`. Even though the default paths above point to the repository tree, the `/photoprism` mount ensures you can wipe the development database or originals by pruning the `storage/` folder on the host.

The `PHOTOPRISM_UID` / `PHOTOPRISM_GID` variables default to the host user (resolved via `${UID:-1000}`) so file ownership inside `storage/` matches your developer account. When running on macOS or Windows, set those values explicitly in `.env` if you encounter permission issues.

## Tips for Contributors

1. **Inspect before editing.** Run `photoprism show config --json | jq '.sections[0].items[] | select(.name=="site-url")'` to confirm the current value before changing environment variables or YAML files.
2. **Prefer `.env` for local overrides.** Docker Compose automatically loads `.env` from the repo root, allowing you to keep machine-specific tweaks (like custom ports) out of `compose.yaml`.
3. **Keep config files out of version control.** The `storage/` directory is ignored by default; if you need reproducible settings, commit sample `options.yml` files or document the required values.
4**Restart services after changes.** Some settings may require rebuilding or resetting volumes.

Following these guidelines keeps the developer environment predictable and minimizes the risk of drifting away from the settings used in CI and QA.
