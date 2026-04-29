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

## Ops Repository

The deployment source of truth is an existing local ops repository:

- Repository: `https://github.com/z0rgoyok/vps-vpn-ops`
- Local path: `/Users/deniszabozhanov/dev/tools/vps-vpn-ops`
- HeyForm service docs: `docs/heyform-operations.md`
- Host inventory: `docs/inventory.md`
- Operator runbook: `docs/runbook.md`
- Deploy script: `scripts/deploy-heyform.sh`
- Host env file: `hosts/msk1-vikunja.env`

Before changing production, use that local ops repository and read its HeyForm operations document. The ops repository owns `Docker Compose`, `Caddy`, host access, Cloudflare DNS state, and rollback instructions. This HeyForm fork owns application source code only.

## Deploy

Run deploy inside the local ops repository:

```bash
cd /Users/deniszabozhanov/dev/tools/vps-vpn-ops
HEYFORM_SOURCE_DIR=/Users/deniszabozhanov/dev/wasteland_w/heyform \
  make deploy-heyform HOST_FILE=hosts/msk1-vikunja.env
```

Equivalent single command from this repository:

```bash
HEYFORM_SOURCE_DIR=/Users/deniszabozhanov/dev/wasteland_w/heyform \
  make -C /Users/deniszabozhanov/dev/tools/vps-vpn-ops \
  deploy-heyform HOST_FILE=hosts/msk1-vikunja.env
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
