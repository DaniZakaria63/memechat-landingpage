# Deploying — memechat.dream-team.works

The MemeChat landing page is served from **https://memechat.dream-team.works**.

Deploys are automatic: **push to `master`** triggers `.github/workflows/deploy.yml`,
which stages the site and rsyncs it to the server through the Cloudflare Tunnel.

```
GitHub Actions runner
  │  push to master
  ├─ stage: memechat-landing.html -> _site/index.html, assets/ -> _site/assets/
  └─ rsync ──► cloudflared access ssh ──► admin.dream-team.works (Cloudflare Access)
                                              └─► sshd ─► deploy user
                                                          └─► /var/www/memechat.dream-team.works
```

## Required repository secrets

Create these in the GitHub repo: **Settings → Secrets and variables → Actions → New repository secret**.

| Secret | Value |
| --- | --- |
| `DEPLOY_SSH_KEY` | Full contents of `~/.ssh/memechat_deploy_ed25519` (the private key generated for this repo) |
| `CF_ACCESS_CLIENT_ID` | Cloudflare Access service token **Client ID** |
| `CF_ACCESS_CLIENT_SECRET` | Cloudflare Access service token **Client Secret** |

### 1. Deploy SSH key

A dedicated key pair named `github-actions-memechat` was generated. Its public half is
already in `/home/deploy/.ssh/authorized_keys` on the server; only the private half goes
into the `DEPLOY_SSH_KEY` secret.

- Local private key: `~/.ssh/memechat_deploy_ed25519`
- Server user: `deploy` (no sudo, owns only the web root)

**Rotation:** generate a new pair, append its public key to
`/home/deploy/.ssh/authorized_keys`, update the secret, then remove the old line.

### 2. Cloudflare Access service token

`admin.dream-team.works` (the SSH entry point) is protected by Cloudflare Access.
GitHub runners can't do the interactive browser login, so they authenticate with a
**service token**:

1. Cloudflare dashboard → **Zero Trust → Access → Service Auth → Service Tokens → Create Service Token**.
   - Name: `github-actions-memechat`
   - Duration: e.g. 1 year (set a calendar reminder to rotate)
2. Copy the **Client ID** into `CF_ACCESS_CLIENT_ID` and the **Client Secret** into
   `CF_ACCESS_CLIENT_SECRET` (the secret is shown only once).
3. Zero Trust → **Access → Applications** → open the application protecting
   `admin.dream-team.works` → add a policy:
   - Action: **Service Auth**
   - Include: **Service Token** → `github-actions-memechat`

   (If you already have a service token for the portfolio deploy, you can reuse it instead.)
4. First run of the workflow will fail with a friendly message until all three secrets
   exist — after creating them, use **Re-run all jobs**.

## Manual deploy (fallback)

Same result as CI, run from the repo root:

```bash
mkdir -p _site
cp memechat-landing.html _site/index.html
cp -r assets _site/assets

# Requires the tunnel auth (browser login or service token env vars) and the deploy key.
rsync -az --delete -e "ssh -i ~/.ssh/memechat_deploy_ed25519 -o User=deploy" \
  ./_site/ admin.dream-team.works:/var/www/memechat.dream-team.works/
```

## Server layout

| What | Where |
| --- | --- |
| Web root | `/var/www/memechat.dream-team.works` (owned by `deploy:deploy`) |
| nginx vhost | `/etc/nginx/sites-available/memechat.dream-team.works.conf` |
| Tunnel ingress | `memechat.dream-team.works` → `http://localhost:80` (tunnel `ovh-server`) |
| DNS | CNAME `memechat` → `bd0efb4d-a596-4c6f-9997-7276f4980f0b.cfargotunnel.com` (proxied) |

Only `index.html` and `assets/` are deployed; repo docs and recon files stay out of the web root.

## Hardening notes (optional)

- Pin the runner's SSH host key by adding a `known_hosts` secret and using
  `StrictHostKeyChecking yes` in the workflow, instead of `accept-new` (TOFU).
- Rotate the service token and the deploy key periodically; both are revocable
  independently of any other site on the server.
