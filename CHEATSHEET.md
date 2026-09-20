# Yuvomi Workflow Cheat Sheet

## SSH into the Pi

```bash
ssh family-manager
```
(Config saved at `~/.ssh/config` on this laptop — `mcthree@family-manager` via key `~/.ssh/id_ed25519_yuvomi`, no password needed.)

## Local dev environment (inside WSL)

```bash
wsl                          # enter WSL from PowerShell/cmd
cd ~/projects/yuvomi
export NVM_DIR="$HOME/.nvm"; . "$NVM_DIR/nvm.sh"   # load Node (once per WSL shell)
npm run dev                  # http://localhost:3000, hot-reloads on save
```

Run detached (survives closing the terminal):
```bash
setsid nohup npm run dev > /tmp/yuvomi-dev.log 2>&1 < /dev/null & disown
```
Check it: `curl http://localhost:3000/` · Stop it: `pkill -f "server/index.js"`

**Note:** the dev server uses its own empty test database — it is NOT the same data as the Pi. Never treat it as the real household instance.

## Push code changes to GitHub

```bash
cd ~/projects/yuvomi
git add -A
git commit -m "fix(tasks): ..."
git push origin main
```
Triggers CI, which builds and publishes `ghcr.io/tmcmullin/yuvomi:latest`.
Check build progress: `gh run list --repo tmcmullin/yuvomi --limit 5`

## Deploy the new image to the Pi

```bash
ssh family-manager
cd ~/yuvomi
docker compose stop
tar czf ~/yuvomi-backups/backup-$(date +%Y%m%d-%H%M%S).tar.gz data backups documents .env docker-compose.yml
docker compose pull
docker compose up -d
```
Verify: `docker ps` (should show `healthy`) · Watch logs: `docker logs yuvomi -f`

## Restore from a backup (if something breaks)

```bash
cd ~/yuvomi
docker compose down
tar xzf ~/yuvomi-backups/<filename>.tar.gz
docker compose up -d
```

## Using the app day to day

- Real household instance (add real tasks, etc.): **http://family-manager:3000**
- Dev/test instance (safe to break, not real data): **http://localhost:3000** (only reachable while `npm run dev` is running in WSL)
