# Deploying — memechat.dream-team.works

The MemeChat landing page is served from **https://memechat.dream-team.works**.

Deploys are automatic: **push to `master`** triggers `.github/workflows/deploy.yml`,
which stages the site and rsyncs it to the server over key-only SSH.

```
GitHub Actions runner
  │  push to master
  ├─ stage: memechat-landing.html -> _site/index.html, assets/ -> _site/assets/
  └─ rsync ──► ssh deploy@139.99.88.23 ──► /var/www/memechat.dream-team.works
```

CI reaches the server at its public IPv4 (`139.99.88.23`) as the unprivileged
`deploy` user with a dedicated SSH key. The `admin.dream-team.works` Cloudflare
Access tunnel remains available for interactive admin use, but CI no longer
depends on it (and no longer needs a Cloudflare Access service token).

## Required repository secrets

Create in GitHub: **Settings → Secrets and variables → Actions → New repository secret**.

| Secret | Value |
| --- | --- |
| `DEPLOY_SSH_KEY` | Full contents of `~/.ssh/memechat_deploy_ed25519` (private key) |

The former `CF_ACCESS_CLIENT_ID` / `CF_ACCESS_CLIENT_SECRET` secrets are no longer
used by the workflow and can be deleted if nothing else needs them.

### Deploy SSH key

- Local private key: `~/.ssh/memechat_deploy_ed25519` (comment `github-actions-memechat`)
- Its public half is in `/home/deploy/.ssh/authorized_keys` on the server.
- Server user `deploy` owns only the web root and has no sudo.

**Rotation:** generate a new pair, append its public key to
`/home/deploy/.ssh/authorized_keys`, update the secret, then remove the old line.

## Manual deploy (fallback)

Same result as CI, run from the repo root:

```bash
mkdir -p _site
cp memechat-landing.html _site/index.html
cp -r assets _site/assets

rsync -az --delete --chmod=D755,F644 \
  -e "ssh -i ~/.ssh/memechat_deploy_ed25519 -o User=deploy" \
  ./_site/ 139.99.88.23:/var/www/memechat.dream-team.works/
```

## Server layout

| What | Where |
| --- | --- |
| Web root | `/var/www/memechat.dream-team.works` (owned by `deploy:deploy`) |
| nginx vhost | `/etc/nginx/sites-available/memechat.dream-team.works.conf` |
| Tunnel ingress | `memechat.dream-team.works` → `http://localhost:80` (tunnel `ovh-server`) |
| DNS | CNAME `memechat` → `bd0efb4d-a596-4c6f-9997-7276f4980f0b.cfargotunnel.com` (proxied) |
| SSH (CI) | `deploy@139.99.88.23:22`, key-only |
| SSH (admin) | `admin.dream-team.works` behind Cloudflare Access (browser login) |

Only `index.html` and `assets/` are deployed; repo docs and recon files stay out of the web root.

## Hardening notes

- `PasswordAuthentication no` is enforced via `/etc/ssh/sshd_config.d/00-hardening.conf`
  (required because `50-cloud-init.conf` sets `yes` and sshd uses the first value found).
  The `deploy` and `root` accounts are password-locked.
- Optional: install `fail2ban`; not strictly needed with key-only auth.
- Optional: pin the runner's SSH host key by storing a `known_hosts` secret and
  switching to `StrictHostKeyChecking yes` (currently TOFU via `accept-new`).
- If the server IP ever changes (`139.99.88.23`), update `DEPLOY_HOST` in the workflow.
