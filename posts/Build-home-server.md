# Build Home Server
Device: EQ12
    - CPU: n100
    - Memory: 16g * 1
    - Disk: 1T SSD * 1
    - OS: Debian12

Softwares:
    - Docker
    - SSH

Services:

| service name | address | desc |
| ------------ | ------- | ---- |
| jellyfin | http://192.168.31.114:8096 | home media service |
| file browser | http://192.168.31.114:8080| file browser |
| samba | smb://192.168.31.114:445 | file share service |
| transmission | http://192.168.31.114:9091 | PT download service |
| hexo | http://192.168.31.114:4000 | | hexo blog service, TODO: use docker later |
| code server | http://192.168.31.114:8888 | vscode web server |
| Apple HA | - | todo |

<!--more-->

Command:

```bash
# start server when start
sudo update-rc.d {service name} enable/disable

# jellyfin - https://jellyfin.org/downloads/docker
docker pull jellyfin/jellyfin:latest
mkdir -p /srv/jellyfin/{config,cache}
docker run -d --name jellyfin --restart=always -v /home/zeki/services/jellyfin/config:/config -v /home/zeki/services/jellyfin/cache:/cache -v /home/zeki:/media --net=host --device=/dev/dri/renderD128 jellyfin/jellyfin:latest
root/nixsyw-xyzfaW-cuqmo1


# install Portainer - https://docs.portainer.io/start/install-ce/server/docker/linux
sudo docker volume create portainer_data
docker run -d -p 8000:8000 -p 9443:9443 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:latest

# instal filebrowser
touch /home/zeki/services/filebrowser/filebrowser.db
docker run -d \
    --name filebrowser \
    --restart=always \
    -v /home/zeki/services/filebrowser/srv:/srv \
    -v /home/zeki/services/filebrowser/filebrowser.db:/database/filebrowser.db \
    -v /home/zeki/services/filebrowser/settings.json:/config/settings.json \
    -e PUID=$(id -u) \
    -e PGID=$(id -g) \
    -p 8080:80 \
    filebrowser/filebrowser:s6

# install transmission
sudo docker run -d \
  --name=transmission \
  -e PUID=1000 \
  -e PGID=1000 \
  -e TZ=Etc/UTC \
  -e TRANSMISSION_WEB_HOME= `#optional` \
  -e USER= `#optional` \
  -e PASS= `#optional` \
  -e WHITELIST= `#optional` \
  -e PEERPORT= `#optional` \
  -e HOST_WHITELIST= `#optional` \
  -p 9091:9091 \
  -p 51413:51413 \
  -p 51413:51413/udp \
  -v /home/zeki/services/transmission/config:/config \
  -v /home/zeki/services/transmission/downloads:/downloads \
  -v /home/zeki/services/transmission/watch:/watch \
  --restart unless-stopped \
  lscr.io/linuxserver/transmission:latest

# install smb docker
sudo docker run  -it --name samba -p 139:139 -p 445:445             -v /home/zeki Documents/share:/mount                        -d dperson/samba     -u "admin;123567890"             -s "share;/mount/;yes;no;no;all;admin" -S


Need Merge with Windows!!!

# code server


# hexo 
    - nodejs 

    - docker

```

date: 2023-12-03 03:25:20