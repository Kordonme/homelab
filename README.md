# 🧱 Homelab VM Setup Guide (Proxmox + Docker + Portainer + ZFS)

## 🚀 Step 1 — Prepare storage (on Proxmox host)

```bash
mkdir -p /tank/docker/{apps,media,db,portainer}
```

---

## 🚀 Step 2 — Mount inside VM

```bash
mkdir -p /tank/docker/apps
mount -t virtiofs docker-apps /tank/docker/apps
```

Persist:

```bash
nano /etc/fstab
```

Add:

```bash
docker-apps /tank/docker/apps virtiofs defaults 0 0
```

---

## ✅ Verify

```bash
touch /tank/docker/apps/test
```

---

## 🚀 Step 3 — Permissions

```bash
chown -R 1000:1000 /tank/docker/apps
```

---

## 🚀 Step 4 — Install Docker

```bash
apt update
apt install docker.io docker-compose-plugin
usermod -aG docker $USER
```

---

## 🚀 Step 5 — Portainer Agent

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
      - /tank/docker/portainer/agent-<vm-name>:/data
```

---

## 🚀 Step 6 — App example

```yaml
services:
  app:
    image: your-image
    volumes:
      - /tank/docker/apps/your-app:/data
```

---

## ⚠️ Rules

- Always use `/tank/docker/...`
- Never use relative paths
- Do not mix Docker volumes and host paths
- Use `zfs destroy` for datasets

---

## ✅ Result

```
Docker → VM → VirtIO-FS → /tank/docker → ZFS
```
