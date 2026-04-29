# Manual Deploy

This fork is deployed manually to `quiz.wastelandw.ru`.

## Production Model

- Source repository: `https://github.com/z0rgoyok/heyform`
- Local source path: `/Users/deniszabozhanov/dev/wasteland_w/heyform`
- Ops repository: `/Users/deniszabozhanov/dev/tools/vps-vpn-ops`
- Production host: `msk1-vikunja`
- Production app path: `/opt/apps/heyform`
- Public domain: `quiz.wastelandw.ru`
- Public ingress: shared `Caddy` from `/opt/apps/vikunja`
- Runtime secrets: `/opt/apps/heyform/.env` on the server
- Persistent data: `/opt/apps/heyform/mongodb`, `/opt/apps/heyform/redis`, `/opt/apps/heyform/assets`

## GitHub Actions

GitHub Actions must stay disabled for this fork. Production deploys are manual and are run from the ops repository.

The fork contains no workflow files under `.github/workflows`.

## Deploy

From the ops repository:

```bash
cd /Users/deniszabozhanov/dev/tools/vps-vpn-ops
HEYFORM_SOURCE_DIR=/Users/deniszabozhanov/dev/wasteland_w/heyform \
  make deploy-heyform HOST_FILE=hosts/msk1-vikunja.env
```

The deploy script:

1. uploads this source tree to `/opt/apps/heyform/source`;
2. builds the Docker image on the production host;
3. keeps the existing server `.env`;
4. keeps MongoDB, Redis, and uploaded assets directories;
5. restarts `HeyForm` and shared `Caddy`.

## Verify

```bash
curl -I https://quiz.wastelandw.ru
curl -sS https://quiz.wastelandw.ru/health/ready
```

Expected health response:

```json
{"status":"ok","service":"heyform-server","checks":{"mongo":"up","redis":"up"}}
```

## Rollback

Rollback is manual:

1. checkout the previous good commit in this repository;
2. run the same deploy command;
3. verify `/health/ready`;
4. if the shared ingress was affected, restore the previous `/opt/backups/heyform-deploy/Caddyfile.*` and restart `Caddy`.
