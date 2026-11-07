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
Tested 2026-11-07: media without key OK; with an invalid key gluetun stays unhealthy and qBittorrent refuses to start (fail-closed). 
In Proxmox LXC add `-dev0 /dev/net/tun,mode=0666` (and nesting=1,keyctl=1) for gluetun.

Stage 3 done: `hldeck status|update|backup|export` (installed to /usr/local/bin by install.sh). Site page: homelabdeck.dev/installer.
Stage 4 done: `hldeck wire` connects Prowlarr->Sonarr/Radarr, Sonarr/Radarr->qBittorrent (+root folders, qBittorrent password in credentials.txt, save path /data/torrents). Idempotent; runs automatically after install when a VPN key is set. Manual: Jellyfin wizard/libraries, indexers, Seerr.
Tested 2026-11-07 with qBittorrent started WITHOUT gluetun (host override HLDECK_QBIT_HOST=qbittorrent); the apps' own connection tests pass. Default host `gluetun` (real setup) untested.
Images are pinned to exact versions (variables V_* at the top of install.sh, tested together 2026-11-07). To upgrade: bump the V_* values, re-test in a clean LXC, release. Users re-run the installer to move to the new set (data is kept).


Tunnel tested 2026-11-07 (no paid VPN): self-hosted WireGuard server (linuxserver/wireguard) in the same LXC, gluetun with VPN_SERVICE_PROVIDER=custom. Results: gluetun healthy, qBittorrent traffic goes through the tunnel, `hldeck wire` with default host `gluetun` passes Sonarr/Radarr download-client tests, kill switch blocks internet AND LAN when the server is stopped and traffic resumes when it returns.
Custom (own server) vpn.env needs: VPN_SERVICE_PROVIDER=custom, WIREGUARD_PRIVATE_KEY, WIREGUARD_PUBLIC_KEY, WIREGUARD_ADDRESSES, WIREGUARD_ENDPOINT_IP, WIREGUARD_ENDPOINT_PORT (+ optional WIREGUARD_PRESHARED_KEY).
Not tested: a commercial provider (Mullvad etc.).
