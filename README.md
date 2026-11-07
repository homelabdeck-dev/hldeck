# hldeck - HomelabDeck mini installer

Profiles:
- `starter` - Homepage, Uptime Kuma, Portainer, Nginx Proxy Manager (dir /opt/hldeck)
- `media` - Jellyfin, Seerr, Sonarr, Radarr, Prowlarr, qBittorrent behind gluetun (WireGuard, kill switch) (dir /opt/hldeck-media)

    sudo bash install.sh            # install
    sudo bash install.sh media --vpn-provider <name> --vpn-key <wireguard-private-key>
    bash install.sh --dry-run       # generate files in a temp dir, change nothing

Debian/Ubuntu, x86_64/aarch64, >= 5 GB free. Output is plain Docker Compose in /opt/hldeck.
Tested 2026-11-07 in a clean Debian 13 LXC (install + idempotent rerun, all 4 services answer).

Media notes: without a VPN key, qBittorrent and gluetun are NOT started (writes vpn.env with CHANGE_ME). One /data mount shared by all apps so hardlinks work. Some providers (e.g. Mullvad) also need WIREGUARD_ADDRESSES in vpn.env.
Tested 2026-11-07: media without key OK; with an invalid key gluetun stays unhealthy and qBittorrent refuses to start (fail-closed). NOT yet tested: a real working tunnel.
In Proxmox LXC add `-dev0 /dev/net/tun,mode=0666` (and nesting=1,keyctl=1) for gluetun.

Roadmap: stage 3 status/backup/export, stage 4 *arr auto-wiring.
Known: images use upstream tags from the HomelabDeck catalog (some `latest`); pin versions before public release.
