# Lunarr Media Server Docker Compose

This repository contains a ready-to-use `docker-compose.yml` and instructions to deploy **[Lunarr](https://github.com/sayem314/lunarr)** – a modern, lightweight, self-hosted media server (alternative to Plex/Jellyfin/Emby) built with SvelteKit + Go.

## 📺 What is Lunarr?

Lunarr is a media streaming server designed for self-hosted enthusiasts who want:
- 🎬 Modern, responsive UI (SvelteKit)
- ⚡ Go-based backend – fast and low memory footprint
- 📚 Library management with TMDB metadata
- ▶️ Direct play & HLS transcoding via FFmpeg
- 🔐 Multi-user support with admin panel
- 🚫 Zero telemetry – your data stays private
- 🐳 Docker‑ready for easy deployment

## 🚀 Quick Start

1. **Clone this repo** (or just copy the files)

   ```bash
   git clone https://github.com/JLalib/docker-lunarr.git
   cd docker-lunarr
   ```

2. **Edit `docker-compose.yml`** – replace `your-secret-here` with a strong secret:

   ```bash
   openssl rand -hex 32   # generate a 32‑byte hex string
   ```

   Then set:

   ```yaml
   environment:
     - AUTH_SECRET=your-generated-secret-here
     - ORIGIN=http://localhost:3000   # change if using a domain
   ```

   Adjust the volume `/mnt/media:/media:ro` to point to your media directory (or use SFTP/WebDAV as needed).

3. **Start the stack**

   ```bash
   docker compose up -d
   ```

4. **Open your browser** → `http://<host-ip>:3000`  
   The first account you create becomes the admin.

## 📂 Volume Mapping

| Container path | Host path (example)          | Purpose                     |
|----------------|------------------------------|-----------------------------|
| `/data`        | `lunarr-data` (Docker volume)| Application data & DB       |
| `/media`       | `/mnt/media` (read‑only)     | Your media files            |

> **Tip:** If you keep media on a NAS, you can mount via SFTP or WebDAV instead of a direct bind‑mount.

## 🔧 Configuration

| Variable       | Example                       | Description                                   |
|----------------|------------------------------|-----------------------------------------------|
| `AUTH_SECRET`  | `a1b2c3d4e5f6...` (64 hex)   | Secret used for signing sessions (required)   |
| `ORIGIN`       | `http://localhost:3000`       | Base URL used for generating absolute links   |
| `TZ`           | `Europe/Madrid`              | Optional: set container timezone              |

## 🛠️ Maintenance

- **View logs**: `docker logs -f lunarr`
- **Update to latest image**:

  ```bash
  docker compose pull
  docker compose up -d
  ```

- **Backup data**:

  ```bash
  docker cp lunarr:/data ./lunarr-backup-$(date +%Y%m%d)
  ```

- **Restore** (stop container first, then copy back).

## 📄 License

This project is released under the MIT License – see the [LICENSE](LICENSE) file for details.

---

*Generated automatically from the blog post:*  
[Cómo instalar Lunarr en Docker - Servidor de streaming de medios autohospedado en Docker](https://genbyte.blogspot.com/2026/07/como-instalar-lunarr-en-docker-servidor.html)