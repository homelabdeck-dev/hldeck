# HLDeck - HomelabDeck mini installer (command: `hldeck`)

Part of [HomelabDeck](https://homelabdeck.dev/installer). MIT licensed.

## Quick start (beginners)

You need a Debian or Ubuntu machine (a VM, a Proxmox LXC container, a mini PC or a Raspberry Pi 64-bit) with internet access and at least 5 GB free. You run the commands in its terminal as root, or put `sudo` in front.

**1. Pick ONE command and paste it into the terminal:**

Starter stack (dashboard, monitoring, Docker manager, reverse proxy):

    curl -fsSL https://homelabdeck.dev/install.sh | sudo bash -s -- starter

Media Lite stack (Jellyfin, Seerr, Sonarr, Radarr, Prowlarr, qBittorrent; no VPN, for your own and legally obtained media):

    curl -fsSL https://homelabdeck.dev/install.sh | sudo bash -s -- media-lite

Media stack with VPN kill switch (needs a WireGuard key from your VPN provider):

    curl -fsSL https://homelabdeck.dev/install.sh | sudo bash -s -- media --vpn-provider PROVIDER_NAME --vpn-key YOUR_WIREGUARD_KEY

**2. Wait a few minutes.** The installer installs Docker if it is missing, downloads the apps and starts them. At the end it prints the address of every app, for example `http://192.168.1.50:8096`. Open them in your browser.

**3. Check it any time:** `hldeck status`

Want to read the script first? Download it, read it, then run it:

    curl -fsSLO https://homelabdeck.dev/install.sh
    less install.sh
    sudo bash install.sh media-lite

Try without installing anything: `bash install.sh media-lite --dry-run`

Running in a Proxmox LXC? Enable `nesting` and `keyctl` in the container's Options > Features first (the `media` stack with VPN also needs `/dev/net/tun`, see below).

What stays manual after the media stacks: create the Jellyfin user and libraries, add your indexers in Prowlarr, sign in to Seerr. The installer prints these steps at the end.

## Profiles

- `starter` - Homepage, Uptime Kuma, Portainer, Nginx Proxy Manager (dir /opt/hldeck)
- `media` - Jellyfin, Seerr, Sonarr, Radarr, Prowlarr, qBittorrent behind gluetun (WireGuard, kill switch) (dir /opt/hldeck-media)
- `media-lite` - the same media stack but qBittorrent runs directly, WITHOUT a VPN (dir /opt/hldeck-media-lite). For your own and legally obtained media; `hldeck wire` runs automatically.

    sudo bash install.sh            # install (starter)
    sudo bash install.sh media-lite
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

media-lite tested 2026-11-08 in LXC 199 (Debian 13): clean install, all 6 services answer, `hldeck wire` connects Sonarr/Radarr to qBittorrent (host `qbittorrent`) and Prowlarr to both, idempotent rerun OK, `media` profile compose unchanged (byte-identical dry-run) and still skips qBittorrent without a key.
