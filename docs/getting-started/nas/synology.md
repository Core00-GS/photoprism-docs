# Running PhotoPrism on a Synology NAS

Before setting up PhotoPrism on your NAS, we recommend that you check the [Synology Knowledge Base](https://kb.synology.com/en-us/DSM/tutorial/What_kind_of_CPU_does_my_NAS_have) for the CPU and memory configuration of your device.

For a good user experience, it should be a 64-bit system with [at least 2 cores and 3 GB of RAM](../index.md#system-requirements). Indexing large photo and video collections also benefits greatly from [using SSD storage](../troubleshooting/performance.md#storage), especially for the database and cache files.

!!! tldr ""
    Should you experience problems with the installation, we recommend that you ask the Synology community for advice, as we cannot provide support for third-party software and services.
    Also note that [RAW image conversion and TensorFlow are disabled](../../user-guide/settings/advanced.md) on devices with 1 GB or less memory, and that high-resolution panoramic images may require [additional swap space](../troubleshooting/docker.md#adding-swap) and/or physical memory above the [recommended minimum](../index.md#system-requirements).

### Will my device be fast enough?

This largely depends on your expectations and the number of files you have. Most users report that PhotoPrism runs
well on their Synology NAS. However, you should keep in mind:

- initial indexing may take longer than on standard desktop computers
- the hardware has no video transcoding support and software transcoding is generally slow

## Setup ##

### Setup using Portainer ###

A step-by-step guide to install PhotoPrism with Portainer can be found [here](../portainer/index.md).

### Setup using Synology Container Manager ###

Follow the steps below if you prefer Synology's built-in [Container Manager](https://www.synology.com/en-global/releaseNote/ContainerManager) (DSM 7.2+). The workflow mirrors Docker Compose, so you keep the entire configuration in a single YAML file and can redeploy updates with a few clicks.

#### 1. Prerequisites

- Install **[Container Manager](https://www.synology.com/en-global/releaseNote/ContainerManager)** from *Package Center ▸ Search ▸ “Container”*. DSM replaces the legacy Docker app with this package starting in DSM 7.2.
- Confirm your NAS has a 64‑bit CPU and at least 4 GB RAM; LinuxLinks found PhotoPrism unstable below that threshold when indexing on DSM hardware.
- Decide where originals live. Fast SSD volumes for the `storage` directory (cache, thumbnails, database dumps) significantly improve indexing performance.

#### 2. Create shared folders

1. Open **Control Panel ▸ Shared Folder ▸ Create**.
2. Create a share such as `/volume1/docker/photoprism`, then add subfolders:
   - `storage` – holds config, cache, sidecars.
   - `database` – persistent MariaDB data.
   - `import` – optional staging folder for uploads.
   - `originals` – use this only if you plan to copy photos into a new folder. Most users should mount the share that already contains their pictures (for example `/volume1/photo`).
3. Grant read/write access to the Container Manager system account (and your admin user) so Compose can mount these paths. Keeping assets inside `/volume1/docker/<project>` simplifies snapshots and backups.

#### 3. Download the PhotoPrism image

1. Launch **Container Manager** and switch to the **Registry** tab.
2. Search for `photoprism/photoprism`, select it, and choose the `latest` tag (use an architecture-specific tag only if your NAS requires it).
3. Click **Download**; the image appears under the **Image** tab once the pull completes.

#### 4. Create a Compose project

1. Go to the **Project** tab and click **Create**.
2. Set a project name such as `photoprism`.
3. Select **Create with compose**, then paste an adapted version of our standard Compose file into the editor:

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
          # persistent database files on the NAS
          - /volume1/docker/photoprism/database:/var/lib/mysql

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
          # canonical URL used to generate share links
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
        volumes:
          # config, cache, and backups (place on SSD storage if available)
          - /volume1/docker/photoprism/storage:/photoprism/storage
          # existing media; replace /volume1/photo with your actual path
          - /volume1/photo:/photoprism/originals
          # optional import staging folder
          - /volume1/docker/photoprism/import:/photoprism/import
    ```

     By default, our Docker images use the volume mount paths `/photoprism/storage` and `/photoprism/originals`, so no [additional variables](../config-options.md#storage) are required to configure them.

4. Update the placeholders (passwords, timezone, `/volume1/docker/photoprism`, and your actual originals share such as `/volume1/photo`) before clicking **Next ▸ Create**. Container Manager stores the Compose file with the project so you can edit it later without retyping.

#### 5. Deploy and verify

1. Highlight your new project and click **Start**. Container Manager creates both services; the **Container** tab should show green status dots for `photoprism` and `mariadb`.
2. Visit `http://<NAS-IP>:2342/` from your browser, complete the welcome wizard, and begin indexing. Expect the first scan to take a while on Atom-class CPUs; keep an eye on RAM usage and let the process finish without interruptions.
3. Revisit the Compose file any time you need to adjust paths, environment variables, or add hardware acceleration flags documented in [Config Options](../config-options.md).
4. Our [First Steps 👣](../../user-guide/first-steps.md) tutorial guides you through the user interface and settings to ensure your library is indexed according to your individual preferences.

## Troubleshooting ##

If your device runs out of memory or other system resources:

- [ ] Try [reducing the number of workers](../config-options.md#indexing) by setting `PHOTOPRISM_WORKERS` to a reasonably small value in your `compose.yaml` file, depending on the performance of your device
- [ ] Make sure [your device has at least 4 GB of swap space](../troubleshooting/docker.md#adding-swap) so that indexing doesn't cause restarts when memory usage spikes; RAW image conversion and video transcoding are especially demanding
- [ ] If you are using SQLite, switch to MariaDB, which is [better optimized for high concurrency](../faq.md#should-i-use-sqlite-mariadb-or-mysql)
- [ ] As a last measure, you can [disable image classification and facial recognition](../config-options.md#feature-flags) 

Other issues? Our [troubleshooting checklists](../troubleshooting/index.md) help you quickly diagnose and resolve them.
