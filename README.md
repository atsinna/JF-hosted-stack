# JF-hosted-stack

Jellyfin + Sonarr + Radarr + Prowlarr + qBittorrent (Docker)(VM)(Ubuntu)

Self-hosted media stack with Docker.
Flow: Prowlarr → Sonarr/Radarr → qBittorrent → /downloads → import → /media → Jellyfin

[Search] Prowlarr 
        │
        ├──> Sonarr (TV) ─┐
        │                 ├──> qBittorrent → /downloads
        └──> Radarr (Movies) ┘
                               └──> Sonarr/Radarr import → /media → Jellyfin

What’s included

Jellyfin (8096): media server

qBittorrent (8080/6881): downloader

Sonarr (8989): TV automation

Radarr (7878): Movies automation

Prowlarr (9696): Indexer manager

Requirements

Docker Engine + Docker Compose plugin
Linux user/group UID:GID = 1000:1000 (default Ubuntu user fits)

Disk paths you control (below)

/home/atsinna/jfh/
├─ configs/
│  ├─ jellyfin
│  ├─ prowlarr
│  ├─ qbt
│  ├─ radarr
│  └─ sonarr
├─ downloads/
└─ media/
   ├─ movies/
   └─ tv/


Create + fix permissions:

mkdir -p /home/atsinna/jfh/{configs/{jellyfin,prowlarr,qbt,radarr,sonarr},downloads,media/{movies,tv}}
			
			sudo chown/chmod permissions -- sudo separates files from root and user making sonarr/radarr root folder not appear
			docker should also be systemctl enabled

App URLs

qBittorrent → http://<your-ip>:8080

Prowlarr → http://<your-ip>:9696

Sonarr → http://<your-ip>:8989

Radarr → http://<your-ip>:7878

Jellyfin → http://<your-ip>:8096

.env variables (what they do)

PUID,PGID	run containers as your user	1000, 1000

TZ		timezone			America/New_York

UMASK		file perms mask			002

BASE		base host path for volumes	/home/atsinna/jfh
BIND_IP		where to bind ports		0.0.0.0 or 127.0.0.1
QBT_WEB_PORT	qBittorrent UI port		8080
QBT_BTP_PORT	BT port (TCP/UDP)		6881
PROWLARR_PORT	Prowlarr port			9696
SONARR_PORT	Sonarr port			8989
RADARR_PORT	Radarr port			7878
JELLYFIN_PORT	Jellyfin port			8096 -- may need to 8096/web

Service Configurations:

qBittorrent
Default save path: /downloads
Categories: movies → /downloads/movies, tv → /downloads/tv -- not necessary

Prowlarr
Add indexers (public or private)
Settings → Apps → Add Sonarr (http://sonarr:8989) and Radarr (http://radarr:7878) with their API keys

Sonarr/Radarr
Root folders: /media/tv and /media/movies
Download client: qBittorrent (qbt:8080, category tv or movies)

Jellyfin

Libraries: Movies → /media/movies, TV → /media/tv
Plugins (Dashboard → Plugins): TMDb, TVDB, OMDb, OpenSubtitles (optional)
Edit each library → Metadata settings → enable those providers

Common commands
docker compose ps
docker compose logs -f jellyfin
docker compose pull && docker compose up -d
docker compose down

Troubleshooting quickies

Apps can’t talk (connection refused)
Use container names on the same network: http://sonarr:8989, http://radarr:7878, http://qbt:8080
(or use your host IP if you exposed ports)

Jellyfin doesn’t see new items
They’re probably still in /downloads (seeding). Use hardlinks or wait for seeding/stop ratio.

Security notes
Don’t expose ARR/qBit UIs publicly. Bind to 127.0.0.1 or use Tailscale/Cloudflare Tunnel.

