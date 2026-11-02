# doop-docker

Prebuilt Docker images for [doop](https://github.com/kgoedecke/doop). Upstream
does not publish an image, so this repo builds one from each upstream release.

This repo holds no app source. The workflow builds the upstream Git repo at the
release tag with the upstream `Dockerfile`, and pushes the image to GHCR.

## Image

```
ghcr.io/<owner>/doop
```

| Tag | Points to |
| --- | --- |
| `latest` | newest upstream release |
| `0.12` | newest `0.12.x` build |
| `0.12.0` | upstream tag `v0.12.0` |

Platforms: `linux/amd64`, `linux/arm64`.

The image is built without PostHog build args, so client analytics are off.

## How it works

[`.github/workflows/build.yml`](.github/workflows/build.yml):

1. Every 6 hours, it finds the newest upstream `vX.Y.Z` release
   (`desktop-v*` tags are ignored).
2. If that version is already in GHCR, it stops.
3. Otherwise it builds each platform on a native runner, then merges them into
   one multi-arch tag set.

Run it by hand from the Actions tab to build a given tag (for example
`v0.11.0`) or to force a rebuild.

## Setup

1. Push this repo to GitHub.
2. Run the workflow once (Actions -> Build and push image -> Run workflow).
3. In the package settings on GitHub, set the `doop` package visibility to
   public if you want to pull without login.

Note: GitHub turns off scheduled workflows in public repos after 60 days with
no repo activity. Turn it on again from the Actions tab if that happens.

## Run

```yaml
services:
  doop:
    image: ghcr.io/<owner>/doop:latest
    ports:
      - '4400:4400'
    environment:
      DATABASE_URL: postgres://doop:doop@db:5432/doop
      BETTER_AUTH_SECRET: ${BETTER_AUTH_SECRET:?set a long random string}
      BETTER_AUTH_URL: ${BETTER_AUTH_URL:-http://localhost:4400}
    depends_on:
      db:
        condition: service_healthy
    restart: unless-stopped

  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: doop
      POSTGRES_PASSWORD: doop
      POSTGRES_DB: doop
    volumes:
      - doop-pg:/var/lib/postgresql/data
    healthcheck:
      test: ['CMD-SHELL', 'pg_isready -U doop']
      interval: 5s
      timeout: 3s
      retries: 10
    restart: unless-stopped

volumes:
  doop-pg:
```

For all other settings (SMTP, `ANTHROPIC_API_KEY`, and more), see upstream
[`.env.example`](https://github.com/kgoedecke/doop/blob/main/.env.example).

## License

doop is licensed under AGPL-3.0 by its authors. This repo only holds build
automation.
