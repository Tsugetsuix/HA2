# ha2 - setup

---

## Folder Structure

create following with owner set to 1000:1000

/etc/ha2/config

create following without owner set or root:root

/mnt/data/

 /media

    /tv

        /serie1

            /season1

    /movies

        /movie1

/torrents

    /tv

    /movies   

---

## Mounted drive

create file in /root/.sambacred

```
username=
password=
```

paste following in /etc/fstab

```
//<username>.your-storagebox.de/backup /mnt/data cifs vers=3.0,credentials=/root/.sambacred,noperm 0 0
```

---

## Docker install

```
 curl -fsSL https://get.docker.com -o get-docker.sh
 sudo sh get-docker.sh
 sudo apt install docker-compose -y
```

---

## Run Docker-Compose.yml

```docker
version: '3'
services:
  heimdall:
    image: lscr.io/linuxserver/heimdall:latest
    container_name: heimdall
    environment:
      - PUID=1000
      - PGID=1000
      - UMASK=002
      - TZ=Europe/Amsterdam
    volumes:
      - /etc/ha2/config/heimdall:/config
    ports:
      - 80:80
      - 443:443
    restart: unless-stopped
  jellyfin:
    image: lscr.io/linuxserver/jellyfin:latest
    container_name: jellyfin
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/Amsterdam
      - JELLYFIN_PublishedServerUrl=romkeb.com
    volumes:
      - /etc/ha2/config/jellyfin:/config
      - /mnt/data/media:/data/media
    ports:
      - 8096:8096
      - 8920:8920 
      - 7359:7359/udp 
      - 1900:1900/udp 
    restart: unless-stopped
  jellyseerr:
    image: fallenbagel/jellyseerr:latest
    container_name: jellyseerr
    environment:
      - LOG_LEVEL=debug
      - TZ=Europe/Amsterdam
      - UMASK=002
    ports:
      - 5055:5055
    volumes:
      - /etc/ha2/config/jellyseerr:/app/config
    restart: unless-stopped
  qbittorrent:
    container_name: qbittorrent
    image: cr.hotio.dev/hotio/qbittorrent
    ports:
      - "8080:8080"
    environment:
      - PUID=1000
      - PGID=1000
      - UMASK=002
      - TZ=Etc/UTC
    volumes:
      - /etc/ha2/config/qbittorrent:/config
      - /mnt/data/torrents:/data/torrents
    restart: unless-stopped
  bazarr:
    container_name: bazarr
    image: cr.hotio.dev/hotio/bazarr
    ports:
      - "6767:6767"
    environment:
      - PUID=1000
      - PGID=1000
      - UMASK=002
      - TZ=Etc/UTC
    volumes:
      - /etc/ha2/config/bazarr:/config
      - /mnt/data/:/data
    restart: unless-stopped
  radarr:
    container_name: radarr
    image: cr.hotio.dev/hotio/radarr
    ports:
      - "7878:7878"
    environment:
      - PUID=1000
      - PGID=1000
      - UMASK=002
      - TZ=Etc/UTC
    volumes:
      - /etc/ha2/config/radarr:/config
      - /mnt/data/:/data
    restart: unless-stopped
  sonarr:
    container_name: sonarr
    image: cr.hotio.dev/hotio/sonarr
    ports:
      - "8989:8989"
    environment:
      - PUID=1000
      - PGID=1000
      - UMASK=002
      - TZ=Etc/UTC
    volumes:
      - /etc/ha2/config/radarr:/config
      - /mnt/data/:/data
    restart: unless-stopped
  prowlarr:
    container_name: prowlarr
    image: cr.hotio.dev/hotio/prowlarr
    ports:
      - "9696:9696"
    environment:
      - PUID=1000
      - PGID=1000
      - UMASK=002
      - TZ=Etc/UTC
    volumes:
      - /etc/ha2/config/prowlarr:/config
    restart: unless-stopped
```
