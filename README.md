# arr-docker-easy

One-command Docker Compose setup for the *arr stack: Sonarr, Radarr, Prowlarr, Jellyfin, Jellyseerr, and Transmission.

No hand-editing compose files, no hunting for the right image tags. Clone, configure your ports and paths in `.env`, and run `docker compose up -d`.

## What's Included

- Prowlarr (port 9696): Indexer manager. Connects Sonarr/Radarr to torrent and Usenet indexers.
- Sonarr (port 8989): TV show manager. Tracks episodes, grabs releases, sends to download client.
- Radarr (port 7878): Movie manager. Same idea as Sonarr but for films.
- Jellyfin (port 8096): Media server. Plays your library on any device, no subscription needed.
- Jellyseerr (port 5055): Request manager. Lets household members request movies/shows through a web UI.
- Transmission (port 9091): BitTorrent client. Optional but included for a complete working setup out of the box.

## Requirements

- Docker 20.10+ with Compose v2
- At least 2 GB RAM (4 GB if you want hardware transcoding in Jellyfin)
- Enough disk for your media library

## Quick Start

```bash
# 1. Clone
git clone https://github.com/cappy-dev/arr-docker-easy.git
cd arr-docker-easy

# 2. Configure
cp .env.example .env
# Edit .env with your user ID, timezone, and media paths
nano .env

# 3. Launch
docker compose up -d
```

That's it. Open `http://your-server:8989` for Sonarr, `:7878` for Radarr, `:8096` for Jellyfin.

## Directory Structure

After first launch, `data/` will look like this:

```
data/
  prowlarr/      # Prowlarr config + database
  sonarr/        # Sonarr config + database
  radarr/        # Radarr config + database
  jellyfin/      # Jellyfin config + metadata
  jellyseerr/    # Jellyseerr config + database
  transmission/  # Transmission config + blocklists
media/
  tv/            # TV shows (set this in Sonarr)
  movies/        # Movies (set this in Radarr)
  downloads/     # Incomplete + completed downloads
```

## Configuration Walkthrough

### 1. Prowlarr (do this first)

Open `:9696`, complete the setup wizard, then add indexers under Settings > Indexers. Prowlarr will sync them to Sonarr and Radarr automatically once you connect them.

### 2. Connect Sonarr and Radarr to Prowlarr

In Prowlarr: Settings > Apps > Add. Point it at your Sonarr and Radarr instances (use the container names: `http://sonarr:8989` and `http://radarr:7878`).

### 3. Connect Transmission

In Sonarr and Radarr: Settings > Download Client > Add > Transmission. Host is `transmission`, port `9091`. No auth by default.

### 4. Jellyfin

Open `:8096`, set up your admin account, and add media libraries pointing to `/media/tv` and `/media/movies`.

### 5. Jellyseerr

Open `:5055`, sign in with your Jellyfin account, and configure which libraries to track for requests.

## Environment Variables

All config lives in `.env`. Key variables:

- `PUID` (default 1000): User ID for file permissions
- `PGID` (default 1000): Group ID for file permissions
- `TZ` (default America/Chicago): Timezone for scheduling and timestamps
- `DATA_ROOT` (default ./data): Where container configs live
- `MEDIA_ROOT` (default ./media): Where your media and downloads live

Run `id` on your host to find your PUID and PGID. If they are not 1000, change them in `.env` or file permissions inside containers will be wrong.

## Common Tasks

```bash
# Stop everything
docker compose down

# Update all images
docker compose pull
docker compose up -d

# Check logs for a specific service
docker compose logs -f sonarr

# Reset a service (deletes its config!)
rm -rf data/sonarr
docker compose up -d sonarr
```

## Hardware Transcoding (Jellyfin)

Jellyfin supports Intel Quick Sync, NVIDIA NVENC, and AMD AMF. To enable it:

1. Make sure your host has the right drivers installed
2. Pass the device through in `docker-compose.yml` by adding to the jellyfin service:
   ```yaml
   devices:
     - /dev/dri:/dev/dri          # Intel/AMD
     # - /dev/nvidia0:/dev/nvidia0  # NVIDIA
   ```
3. In Jellyfin: Dashboard > Playback > Transcoding > select your hardware encoder

## Security Notes

- These services expose HTTP ports with no built-in auth on first run. Put them behind a reverse proxy (Caddy, Nginx) with auth if exposed to the internet.
- Transmission has no auth by default. Set a password in its settings after first launch.
- Keep `.env` out of version control. It is already in `.gitignore`.

## License

MIT. Use it however you want.
