# Docker + Nginx + GitHub Actions deploy (solo-friendly)

Goal: `git push` → GitHub Actions → SSH → `docker compose up -d`

Keep it simple:
- one VPS
- one repo checkout
- docker compose as the “runtime contract”
- Nginx reverse proxy in front
- app binds to localhost only

## High-level flow

1) Push to `main`
2) CI builds/tests (optional)
3) Deploy job SSHes into the server
4) Server pulls latest code
5) `docker compose up -d --build`
6) prune old images occasionally

## Patterns worth copying (without copying your code)

- **Multi-stage Docker builds**: smaller images, faster deploys
- **Next.js standalone output**: fewer runtime deps
- **Prune**: don’t let old layers fill your disk

## Server layout (example)

- `/srv/myapp` (git checkout)
- `/srv/myapp/docker-compose.yml`
- `/srv/myapp/.env` (server-only, not committed)

## docker-compose.yml (bind app to 127.0.0.1)

```yaml
services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    env_file: .env
    restart: unless-stopped
    ports:
      - "127.0.0.1:3000:3000"
    # If you need persistent data, mount volumes explicitly.

  nginx:
    image: nginx:alpine
    restart: unless-stopped
    depends_on:
      - app
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./infra/nginx.conf:/etc/nginx/conf.d/default.conf:ro
      # Mount certs if you terminate TLS here (or terminate elsewhere).
```

Notes:
- The app is **not** reachable from the public internet directly.
- Only Nginx is public.

## Minimal Nginx reverse proxy

```nginx
server {
  listen 80;
  server_name example.com;

  location / {
    proxy_pass http://127.0.0.1:3000;
    proxy_http_version 1.1;

    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
  }
}
```

Keep TLS termination consistent with your edge setup.
If Cloudflare is in front, ensure your origin only accepts what you intend.

## GitHub Actions: SSH deploy (appleboy/ssh-action)

Store secrets in GitHub:
- `DEPLOY_HOST`
- `DEPLOY_USER`
- `DEPLOY_KEY` (private key)
- optional: `DEPLOY_PORT`

Minimal workflow:

```yaml
name: deploy

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: SSH deploy
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.DEPLOY_HOST }}
          username: ${{ secrets.DEPLOY_USER }}
          key: ${{ secrets.DEPLOY_KEY }}
          port: ${{ secrets.DEPLOY_PORT || 22 }}
          script: |
            set -euo pipefail
            cd /srv/myapp
            git fetch --all
            git reset --hard origin/main
            docker compose pull || true
            docker compose up -d --build
            docker image prune -f
```

Notes:
- `git reset --hard` is blunt. That’s the point: no snowflake servers.
- `docker compose pull || true` helps if you also publish images sometimes.
- prune keeps disks alive on small VPSes.

## Checklist

- [ ] Server has Docker + Compose v2 installed
- [ ] Repo checked out at a stable path (e.g., `/srv/myapp`)
- [ ] `.env` exists on server; not committed
- [ ] App binds to `127.0.0.1` only
- [ ] Nginx proxies to localhost app port
- [ ] GitHub secrets set (host/user/key[/port])
- [ ] Workflow deploys on push to `main`
- [ ] Old images pruned (manual or scheduled)
