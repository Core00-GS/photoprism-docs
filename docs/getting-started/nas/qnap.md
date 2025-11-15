# Setting Up PhotoPrism on QNAP

Before setting up PhotoPrism on your NAS, we recommend that you check the [QNAP product database](https://www.qnap.com/en/product) for the CPU and memory configuration of your device.

For a good user experience, it should be a 64-bit system with [at least 2 cores and 3 GB of RAM](../index.md#system-requirements). Indexing large photo and video collections also benefits greatly from [using SSD storage](../troubleshooting/performance.md#storage), especially for the database and cache files.

!!! tldr ""
    Should you experience problems with the installation, we recommend that you ask the QNAP community for advice, as we cannot provide support for third-party software and services.
    Also note that [RAW image conversion and TensorFlow are disabled](../../user-guide/settings/advanced.md) on devices with 1 GB or less memory, and that high-resolution panoramic images may require [additional swap space](../troubleshooting/docker.md#adding-swap) and/or physical memory above the [recommended minimum](../index.md#system-requirements).

## Setup

### Setup using QNAP Container Station (GUI)

Follow these steps if you prefer Container Station’s graphical workflow on QTS 5 / QuTS hero. The interface treats PhotoPrism and its MariaDB dependency as a single “Application” defined by a Docker Compose file.

#### 1. Prepare your NAS

1. Install **Container Station** from the App Center, then launch it and accept the first-launch prompts (downloads the base images it requires).
2. Update QTS/QuTS hero to the latest release, then reboot so Kernel patches and Docker updates are applied before you deploy storage-heavy apps.
3. Decide where the PhotoPrism workspace should live. Most users keep everything under `/share/Container/photoprism`, while the actual originals may stay in `/share/Multimedia` or another existing volume.
4. Optional but recommended: enable SSH (Control Panel ▸ Network & File Services ▸ Telnet/SSH) so you can run `docker compose` commands for troubleshooting later.

#### 2. Create persistent folders

Use File Station or SSH to create the directories Container Station will mount:

```sh
mkdir -p /share/Container/photoprism/{storage,import,database}
```

- `storage` holds the application cache, sidecars, and backups (SSD recommended).
- `database` stores MariaDB data files.
- `originals` should point to the share that already contains your photo library (for example `/share/Multimedia`). Create `/share/Container/photoprism/originals` only if you plan to copy photos into a new folder later.
- `import` is optional but useful if you upload archives that should be reorganized by PhotoPrism before landing in Originals.

#### 3. Create the Compose application

1. Open **Container Station ▸ Applications ▸ Create**.
2. Enter an application name such as `photoprism`.
3. Paste the Compose configuration below into the YAML editor and adjust passwords, paths, and timezone values before clicking **Validate** and **Create**:

    ```yaml
    services:
      mariadb:
        image: mariadb:11
        restart: unless-stopped
        environment:
          MARIADB_AUTO_UPGRADE: "1"
          MARIADB_INITDB_SKIP_TZINFO: "1"
          MARIADB_ROOT_PASSWORD: "supersecret"
          MARIADB_DATABASE: photoprism
          MARIADB_USER: photoprism
          MARIADB_PASSWORD: "change-me"
        volumes:
          # persistent MariaDB data on the NAS
          - /share/Container/photoprism/database:/var/lib/mysql

      photoprism:
        image: photoprism/photoprism:latest
        depends_on:
          - mariadb
        restart: unless-stopped
        ports:
          - "2342:2342"
        environment:
          # initial admin password (8-72 characters)
          PHOTOPRISM_ADMIN_PASSWORD: "choose-a-strong-password"
          # public URL so share links and redirects include the right origin
          PHOTOPRISM_SITE_URL: "http://YOUR_NAS_IP:2342/"
          # force HTTP even if HTTPS is configured
          PHOTOPRISM_DISABLE_TLS: "false"
          # create a self-signed certificate as fallback
          PHOTOPRISM_DEFAULT_TLS: "true"
          # default UI language (e.g., en, de, fr)
          PHOTOPRISM_DEFAULT_LOCALE: "en"
          # location language (local, en, de, …)
          PHOTOPRISM_PLACES_LOCALE: "local"
          # write YAML sidecars with asset metadata
          PHOTOPRISM_SIDECAR_YAML: "true"
          # back up album metadata periodically
          PHOTOPRISM_BACKUP_ALBUMS: "true"
          # enable automatic database backups
          PHOTOPRISM_BACKUP_DATABASE: "true"
          # cron entry or shortcut (daily, weekly) for backups
          PHOTOPRISM_BACKUP_SCHEDULE: "daily"
          # cron syntax or "" to disable scheduled indexing
          PHOTOPRISM_INDEX_SCHEDULE: ""
          # delay (seconds) before indexing WebDAV uploads
          PHOTOPRISM_AUTO_INDEX: 300
          # delay (seconds) before importing WebDAV uploads
          PHOTOPRISM_AUTO_IMPORT: -1
          # auto-flag potentially offensive content (TensorFlow required)
          PHOTOPRISM_DETECT_NSFW: "false"
          # allow uploads that might be offensive
          PHOTOPRISM_UPLOAD_NSFW: "true"
          # restrict uploads to listed extensions (leave blank to allow all)
          PHOTOPRISM_UPLOAD_ALLOW: ""
          # allow zip uploads (extracted before import)
          PHOTOPRISM_UPLOAD_ARCHIVES: "true"
          # max upload size in MB
          PHOTOPRISM_UPLOAD_LIMIT: 5000
          # max originals size in MB (larger files are skipped)
          PHOTOPRISM_ORIGINALS_LIMIT: 5000
          # enable gzip to reduce bandwidth
          PHOTOPRISM_HTTP_COMPRESSION: "gzip"
          # database driver / connection settings
          PHOTOPRISM_DATABASE_DRIVER: "mysql"
          PHOTOPRISM_DATABASE_SERVER: "mariadb:3306"
          PHOTOPRISM_DATABASE_NAME: "photoprism"
          PHOTOPRISM_DATABASE_USER: "photoprism"
          PHOTOPRISM_DATABASE_PASSWORD: "change-me"
          # container timezone so cron schedules run at local time
          TZ: "UTC"
        volumes:
          # config, cache, and backups (use SSD-backed storage if available)
          - /share/Container/photoprism/storage:/photoprism/storage
          # existing media; replace /share/Multimedia with your actual path
          - /share/Multimedia:/photoprism/originals
          # optional staging folder for the Import tool
          - /share/Container/photoprism/import:/photoprism/import
    ```

     Our Docker images are configured to use the default mounts `/photoprism/storage` and `/photoprism/originals`, so no [additional environment variables](../config-options.md#storage) are needed to configure the storage and originals paths.

4. Double-check that each host path points to the correct absolute `/share/...` location so you don't lose data when updating or redeploying the stack. After the application is created, open **Applications ▸ photoprism ▸ Settings** and set a default web port shortcut (Service `photoprism`, Port `2342`) so the “Open” button launches the UI.

#### 4. Start and verify

1. Select the new application and click **Start**. Container Station will pull both images and create the containers.
2. Watch the application logs for `Starting PhotoPrism...` and `database system is ready to accept connections`.
3. Visit `http://<NAS-IP>:2342/`, sign in with the admin password you set, and follow the welcome wizard to point PhotoPrism at your originals folder if you mounted a parent directory.
4. Trigger **Library ▸ Index ▸ Start** once to create previews and metadata. Keep the browser tab open until the queue drains.

#### 5. Keep it updated

- When new releases ship, open **Applications ▸ photoprism**, click **Stop**, then **Update Images** to pull the latest tags, and start the application again.
- Regularly download the MariaDB backups from `/share/Container/photoprism/storage/backups` or replicate the entire folder to another disk.
- If Container Station reports YAML errors, click **Applications ▸ photoprism ▸ Edit YAML** to fix indentation or update environment variables without recreating the project.

### Setup using Docker Compose (CLI)

Prefer the terminal? The community tutorial below walks through the same deployment via SSH:

↪ <https://safjan.com/install-photoprism-on-qnap-nas-using-docker-compose/>

!!! example ""
    **Help improve these docs!** You can contribute by clicking :material-file-edit-outline: to send a pull request with your changes.
