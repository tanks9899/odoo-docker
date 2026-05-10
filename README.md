# Odoo Docker

Custom Odoo 19.0 Docker build contexts for two scenarios:

- **Nightly build** — install from Odoo's nightly `.deb` packages when the [official Docker Hub image](https://github.com/odoo/docker) lags behind.
- **Source build** — run from a local Odoo source checkout, useful for development or patching.

---

## Nightly build

### When to use

Check the [official Odoo Docker repository](https://github.com/odoo/docker) to see if the image for your target version is up to date. If the release date is behind the [nightly packages](https://nightly.odoo.com/19.0/nightly/deb/), build your own.

### Getting the latest release info

1. Go to [https://nightly.odoo.com/19.0/nightly/deb/](https://nightly.odoo.com/19.0/nightly/deb/).
2. Download the `_amd64.changes` file for the latest build — it contains the release date and SHA1 checksum.
3. Note down:
   - **Release date** — used as `ODOO_RELEASE` (format: `YYYYMMDD`, e.g. `20260502`)
   - **SHA1 checksum** of the `.deb` file — used as `ODOO_SHA`

### Updating the Dockerfile

Edit `build-odoo-nightly/Dockerfile` and update these two lines:

```dockerfile
ARG ODOO_RELEASE=20260421
ARG ODOO_SHA=f9e219b07a6abf5b272cd016f828a8f40ec82eb6
```

### Building the image

```bash
docker build -t odoo-19.0:<ODOO_RELEASE> build-odoo-nightly/
```

### Running

Update the `web.image` field in `compose.build-odoo-nightly.yml` to your new tag:

```yaml
services:
  web:
    image: odoo-19.0:20260502
```

Then start:

```bash
docker compose -f compose.build-odoo-nightly.yml up -d
```

### Prerequisites

Create a file named `odoo_pg_pass` in the repo root containing the PostgreSQL password. It is consumed as a Docker secret by both the `web` and `db` services.

```bash
echo "your-password" > odoo_pg_pass
```

---

## Source build

### When to use

Use this when you want to run Odoo directly from a local source tree — for example, to test a patch or develop a core module.

### Setup

Place your Odoo source tree (containing `odoo-bin` and `requirements.txt`) inside `build-odoo-source/`.

### Running

```bash
docker compose -f compose.build-odoo-source.yml up -d
```

The entire `build-odoo-source/` directory is bind-mounted to `/opt/odoo` inside the container, so source changes are reflected without rebuilding the image.

The PostgreSQL connection uses hardcoded credentials (`odoo`/`odoo`) defined in `build-odoo-source/odoo.conf` and the compose file — no Docker secret required.

---

## Custom modules

Place custom addon modules in the `addons/` directory. Both stacks mount it to `/mnt/extra-addons` inside the container.

## Runtime config override

- **Nightly build** — `config/odoo.conf` is bind-mounted into the container at `/etc/odoo`, overriding the config baked into the image. Edit this file to customise options without rebuilding.
- **Source build** — `build-odoo-source/odoo.conf` is bind-mounted directly to `/etc/odoo/odoo.conf` inside the container. Edit this file to customise options without rebuilding.
