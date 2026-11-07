# hldeck - HomelabDeck mini installer

Stage 1: `starter` profile (Homepage, Uptime Kuma, Portainer, Nginx Proxy Manager).

    sudo bash install.sh            # install
    bash install.sh --dry-run       # generate files in a temp dir, change nothing

Debian/Ubuntu, x86_64/aarch64, >= 5 GB free. Output is plain Docker Compose in /opt/hldeck.
Tested 2026-11-07 in a clean Debian 13 LXC (install + idempotent rerun, all 4 services answer).

Roadmap: stage 2 `media` profile (+gluetun), stage 3 status/backup/export, stage 4 *arr auto-wiring.
Known: images use upstream tags from the HomelabDeck catalog (some `latest`); pin versions before public release.
