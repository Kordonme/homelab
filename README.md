# 🧱 Homelab VM Setup Guide (Proxmox + Docker + Portainer + ZFS)

This guide describes a **generic, reusable setup** for any VM in your homelab.

Goal:
- All persistent data lives on `/tank` (ZFS on Proxmox host)
- Each VM has a clear responsibility
- Docker is managed via Portainer
- No data is stored inside VM disks

---

# 🧠 Architecture Overview

Proxmox host:
  /tank/docker/
    apps/
    media/
    db/
    portainer/

VMs:
  - Apps VM → APIs, web services
  - Media VM → streaming, downloads
  - DB VM → databases
  - Portainer VM → control plane

Each VM only mounts what it needs.

---

# 🚀 Step 1 — Prepare storage (on Proxmox host)

Create base structure:

/tank/docker/
  apps/
  media/
  db/
  portainer/

---

# 🚀 Step 2 — Create Directory Mapping

In Proxmox UI:

Datacenter → Directory Mappings → Add

Example:

docker-apps   → /tank/docker/apps  
docker-media  → /tank/docker/media  
docker-db     → /tank/docker/db  
docker-portainer → /tank/docker/portainer  

---

# 🚀 Step 3 — Attach storage to VM

VM → Hardware → Add → VirtIOFS

Select the relevant mapping (e.g. docker-apps)

Restart VM

---

# 🚀 Step 4 — Mount inside VM

Create mount point:

mkdir -p /tank/docker/apps

Mount:

mount -t virtiofs docker-apps /tank/docker/apps

Persist in /etc/fstab:

docker-apps /tank/docker/apps virtiofs defaults 0 0

---

# ✅ Verify setup

touch /tank/docker/apps/test

Check on Proxmox host:

ls /tank/docker/apps

If file exists → setup is correct

---

# 🚀 Step 5 — Permissions

General usage:

chown -R 1000:1000 /tank/docker/apps

Adjust per service if needed

---

# 🚀 Step 6 — Install Docker

apt update  
apt install docker.io docker-compose-plugin  

Add user:

usermod -aG docker $USER  

Log out and back in

---

# 🚀 Step 7 — Portainer Agent

Each VM should run an agent.

Recommended structure:

/tank/docker/portainer/agent-<vm-name>

Example compose:

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
      - /tank/docker/portainer/agent-<vm-name>:/data

---

# 🚀 Step 8 — Application stacks

General rule:

- Always use absolute paths
- Always store data in /tank/docker/...

Example:

services:
  app:
    image: your-image
    volumes:
      - /tank/docker/apps/your-app:/data

---

# ⚠️ CRITICAL WARNINGS

## 1. Never use relative paths

❌ ./data  
✅ /tank/docker/...

---

## 2. Do not mix storage types

Avoid:
- Docker volumes (hidden)
- VM disk storage

Use:
- /tank/docker only

---

## 3. ZFS datasets are not normal folders

Delete with:

zfs destroy tank/docker/<name>

NOT:

rm -rf

---

## 4. Keep structure by purpose, not VM

Good:

/tank/docker/apps  
/tank/docker/media  

Bad:

/tank/docker/vm-103/

---

## 5. Keep stacks consistent

Use:
- Portainer stacks for apps
- CLI only for bootstrap (agent)

---

# 🧠 Naming conventions

Folders:

/tank/docker/apps/<service>  
/tank/docker/media/<service>  
/tank/docker/db/<service>  

Agent:

/tank/docker/portainer/agent-<vm>

Stacks:

apps  
media  
db  

---

# 🔥 Final checklist

- [ ] VirtIO-FS attached
- [ ] Mounted inside VM
- [ ] Write test works
- [ ] Docker installed
- [ ] Portainer agent running
- [ ] Data stored under /tank/docker
- [ ] Data visible from host

---

# 🚀 Result

Docker → VM → VirtIO-FS → /tank/docker → ZFS

- Persistent
- Fast
- Structured
- Backup-ready

---

# 🧠 Future improvements

- Remove exposed ports (use internal networking)
- Add ZFS snapshots
- Move stacks to Git
- Centralize Portainer control
