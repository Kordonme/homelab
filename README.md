# 🧱 New VM Setup Guide (Proxmox + Docker + Portainer + ZFS)

This guide walks you through setting up a **new VM from scratch** with consistent naming, storage, and Docker setup.

---

# 🎯 Goal

- All persistent data lives on `/tank` (Proxmox host)
- VM disk is NOT used for application data
- Docker runs inside the VM
- Portainer manages applications
- Storage is mounted using VirtIO-FS

---

# 🧠 Naming Convention (IMPORTANT)

Choose a clear, purpose-based VM name.

### Examples:
- `apps`
- `media`
- `db`
- `worker`

Use the same name consistently:
- VM name
- Portainer agent folder
- Compose project naming (where relevant)

---

# 🚀 Step 1 — Prepare storage on Proxmox host

Run on Proxmox:

```bash
mkdir -p /tank/docker/<vm-name>
```

---

# 🚀 Step 2 — Create Directory Mapping

In Proxmox UI:

**Datacenter → Directory Mappings → Add**

Example mappings:

| ID              | Path                    |
|-----------------|-------------------------|
| docker-apps     | /tank/docker/apps       |
| docker-media    | /tank/docker/media      |
| docker-db       | /tank/docker/db         |
| docker-portainer| /tank/docker/portainer  |

---

# 🚀 Step 3 — Attach storage to VM

Go to:

**VM → Hardware → Add → VirtIOFS**

Select the appropriate mapping (example for apps VM):
- `docker-apps`

Restart the VM.

---

# 🚀 Step 4 — Mount storage inside VM

Example for `apps`:

```bash
sudo mkdir -p /tank/docker/apps
sudo mount -t virtiofs docker-apps /tank/docker/apps
```

---

# 💾 Persist mount (important)

Edit fstab:

```bash
sudo nano /etc/fstab
```

Add:

```bash
docker-apps /tank/docker/apps virtiofs defaults 0 0
```

---

# ✅ Verify mount

```bash
touch /tank/docker/apps/test
```

Check on Proxmox:

```bash
ls /tank/docker/apps
```

---

# 🚀 Step 5 — Fix permissions

```bash
sudo chown -R 1000:1000 /tank/docker/apps
```

---

# 🚀 Step 6 — Install Docker

```bash
sudo apt update
sudo apt install docker.io docker-compose-plugin
sudo usermod -aG docker $USER
```

Log out and back in.

---

# 🚀 Step 7 — Setup Portainer Agent

Create folder using VM name:

```bash
mkdir -p /tank/docker/portainer/agent-apps
```

Create file:

```bash
nano ~/portainer-agent.yml
```

Paste:

```yaml
services:
  agent:
    image: portainer/agent:latest
    restart: unless-stopped
    ports:
      - "9001:9001"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /var/lib/docker/volumes:/var/lib/docker/volumes
      - /:/host
      - /tank/docker/portainer/agent-apps:/data
```

Run:

```bash
docker compose -f ~/portainer-agent.yml -p infra up -d
```

---

# 🚀 Step 8 — Deploy applications (via Portainer)

Example stack:

```yaml
services:
  app:
    image: your-image
    volumes:
      - /tank/docker/apps/my-app:/data
```

---

# ⚠️ CRITICAL RULES

## Never use relative paths

❌
```bash
./data
```

✅
```bash
/tank/docker/...
```

---

## Do not use Docker volumes for persistent data

Avoid:

```bash
docker volume ls
```

---

## Always store data in /tank/docker

---

## ZFS datasets are not normal folders

Delete with:

```bash
zfs destroy tank/docker/<name>
```

---

# 🔥 Final Result

```
Docker → VM → VirtIO-FS → /tank/docker → ZFS
```

---

# 🧠 Recommended structure

```
/tank/docker/
  apps/
    my-app/
  media/
  db/
  portainer/
    agent-apps/
```

---

# ✅ Final checklist

- [ ] Storage mounted
- [ ] fstab configured
- [ ] Write test works
- [ ] Docker installed
- [ ] Portainer agent running
- [ ] Data stored in /tank/docker

---

# 🚀 Done

Your VM is now:
- Persistent
- Structured
- Easy to maintain
- Ready for Portainer-managed workloads
