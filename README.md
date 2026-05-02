# Odoo Docker

Custom Odoo Docker image build, for cases when the [official Odoo Docker image](https://github.com/odoo/docker) is not yet updated to the latest nightly release.

## When to build a custom image

Check the [official Odoo Docker repository](https://github.com/odoo/docker) to see if the image for your target version is up to date. If the release date is behind, you can build your own image from the latest nightly package.

## Getting the latest release info

1. Go to [https://nightly.odoo.com/](https://nightly.odoo.com/) and navigate to your version (e.g. `19.0/nightly/deb/`).
2. Download the `_amd64.changes` file for the latest build — it contains the release date and SHA1 checksum.
3. Note down:
   - **Release date** — used as `ODOO_RELEASE` (format: `YYYYMMDD`, e.g. `20260502`)
   - **SHA1 checksum** of the `.deb` file — used as `ODOO_SHA`

## Updating the Dockerfile

Edit `build-odoo-image/Dockerfile` and update these two lines:

```dockerfile
ARG ODOO_RELEASE=20260421
ARG ODOO_SHA=f9e219b07a6abf5b272cd016f828a8f40ec82eb6
```

Replace with the values from the `.changes` file you downloaded.

## Building the image

Run from the `build-odoo-image/` directory, tagging with the version and release date:

```bash
cd build-odoo-image
docker build -t odoo-19.0:20260502 .
```

## Updating compose.yml

After the build, update the `image` field in `compose.yml` to point to your custom image:

```yaml
services:
  web:
    image: odoo-19.0:20260502
```

## Running

```bash
docker compose up -d
```
