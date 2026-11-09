<p align="center">
  <img width="64" src="./assets/icon-64x64.png" alt="Send logo" />
</p>

# Send (fork)

Simple, private file sharing: files are encrypted end to end in the browser, and the share link expires automatically after a number of downloads or a delay.

This is a fork of [timvisee/send](https://gitlab.com/timvisee/send), the community continuation of Mozilla's Firefox Send. It is maintained at [PabloVTM/send](https://github.com/PabloVTM/send) with the PabloSystems theme shared with the other PabloVTM tools: Deep Sage Green `#4B5A50`, Warm Ivory `#F1EEE3` and Vibrant Vermillion Red `#F20B0B` for details, system monospace fonts and a `pablo@systems:~/send` prompt next to the logo, in light and dark modes. It stays compatible with the [`ffsend`](https://github.com/timvisee/ffsend) CLI.

**Docs:** [Docker](docs/docker.md), [FAQ](docs/faq.md), [Encryption](docs/encryption.md), [Build](docs/build.md), [Deployment](docs/deployment.md), [Localization](docs/localization.md)

## Image

Images are published to GitHub Container Registry for `linux/amd64` and `linux/arm64`:

| Tag | Content |
| --- | --- |
| `ghcr.io/pablovtm/send:latest` | Latest build of `master`. |
| `ghcr.io/pablovtm/send:YYYY-MM-DD` | Dated version, e.g. `2026-11-09`. Each build upgrades the Alpine packages, so a new dated version also brings the latest Alpine security fixes. |

## Quick start

Send needs Redis for its metadata. Files are stored on the local disk by default, or on S3 or Google Cloud Storage.

```yaml
services:
  send:
    image: ghcr.io/pablovtm/send:latest
    restart: unless-stopped
    ports:
      - 1443:1443
    environment:
      - BASE_URL=http://localhost:1443
      - REDIS_HOST=redis
      - FILE_DIR=/uploads
    volumes:
      - ./uploads:/uploads
    depends_on:
      - redis
  redis:
    image: redis:alpine
    restart: unless-stopped
    command: redis-server --appendonly yes
    volumes:
      - ./redis:/data
```

The uploads directory must be writable by the container user (uid `1000`):

```bash
mkdir -p uploads && sudo chown 1000:1000 uploads
```

Then open http://localhost:1443. The container reports its health on `/__heartbeat__` (storage and Redis) and `/__lbheartbeat__` (process only).

To share files with others, set `BASE_URL` to the public HTTPS URL and put a reverse proxy with TLS termination in front of Send: the browser crypto APIs it relies on only work over HTTPS (or on `localhost`).

## Configuration

The server is configured with environment variables. The most common ones:

| Variable | Purpose |
| --- | --- |
| `BASE_URL` | Public URL of the instance, e.g. `https://send.example.com`. |
| `REDIS_HOST`, `REDIS_PORT`, `REDIS_PASSWORD` | Redis connection. |
| `FILE_DIR` | Local storage directory (when not using S3 or GCS). |
| `S3_BUCKET`, `S3_ENDPOINT`, `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY` | S3 or S3-compatible storage. |
| `GCS_BUCKET` | Google Cloud Storage bucket. |
| `MAX_FILE_SIZE` | Maximum upload size in bytes (default 2.5 GiB). |
| `MAX_EXPIRE_SECONDS`, `MAX_DOWNLOADS` | Upper limits for the expiry delay (default 7 days) and the download count (default 100). |
| `SEND_FOOTER_DMCA_URL` | Contact page for takedown requests, shown in the footer. |
| `UI_COLOR_PRIMARY`, `UI_COLOR_ACCENT` | Override the theme colors (defaults `#4B5A50` and `#3E4B43`). |

All options and their defaults are in [server/config.js](server/config.js), with examples in [docs/docker.md](docs/docker.md). Long expiry delays on a public instance invite abuse: keep it on a LAN, behind authentication, or set a takedown contact.

## Releasing

Push a date tag to publish a dated version (and `latest`):

```bash
git tag 2026-11-09
git push origin 2026-11-09
```

The [build-test-publish](.github/workflows/build-test-publish.yml) workflow lints and tests the code, builds the image from the latest Alpine base with no cache, smoke tests it with Redis, then pushes the multi-arch image. Pushes to `master` publish `latest`.

## Building

```bash
docker build -t send .
```

## Development

Requires Node.js 26 and, optionally, a Redis server.

```bash
npm install
npm start
```

Then browse to http://localhost:8080.

| Command | Description |
| --- | --- |
| `npm run lint` | Lints the CSS and JavaScript code. |
| `npm test` | Runs the backend and frontend tests (the frontend tests need Chrome, point `PUPPETEER_EXECUTABLE_PATH` at it). |
| `npm run build` | Builds the production assets. |
| `npm run prod` | Runs the server in production configuration. |
| `npm run format` | Formats the code with prettier. |

## Credits

Send was created by [Mozilla](https://mozilla.org) as Firefox Send and kept alive by [Tim Visee](https://timvisee.com) and the [contributors](CONTRIBUTORS) of [timvisee/send](https://gitlab.com/timvisee/send). If you find it useful, consider [donating to Tim](https://timvisee.com/donate). This fork is maintained by [PabloVTM](https://github.com/PabloVTM).

## License

[Mozilla Public License 2.0](LICENSE). [qrcode.js](https://github.com/kazuhikoarase/qrcode-generator) is licensed under MIT.
