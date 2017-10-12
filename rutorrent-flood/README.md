This is a combination of ruTorrent and Flood, just two of an amazing set of images provided by the great Linuxserver.io.

I much prefer the look and simplicity of Flood's Web UI but the immense toolkit that ruTorrent brings to the table makes it almost a requirement when using rTorrent.

I combined these to get the best of both worlds while hopefully maintaining a lightweight image that I was unable to find amongst other images similar to this.  

Usage:

      docker create --name=rutorrent-flood \
      -v <path to data>:/config \
      -v <path to downloads>:/downloads \
      -e PGID=<gid> -e PUID=<uid> \
      -e TZ=<timezone> \
      -p 80:80 \
      -p 3000:3000 \
      -p 5000:5000 \
      -p 50000:50000 \
      -p 6881:6881/udp \
      p0rtals/rutorrent-flood

A docker-compose example might look like so:

        version: '2'
        services:
          rtorrent:
            image: p0rtals/rutorrent-flood
            container_name: rutorrent-flood
            restart: always
            ports:
              - 80:80
              - 3000:3000
              - 5000:5000
              - 50000:50000
              - 6881:6881/udp
            volumes:
              - /my/downloads:/downloads
              - /my/config/files:/config
              - /etc/localtime:/etc/localtime:ro
            environment:
              PUID: 1000
              PGID: 1000
              TZ: America/Denver

Parameters:

The parameters are split into two halves, separated by a colon, the left hand side representing the host and the right the container side. For example with a port -p external:internal - what this shows is the port mapping from internal to external of the container. So -p 8080:80 would expose port 80 from inside the container to be accessible from the host's IP on port 8080 http://192.168.x.x:8080 would show you what's running INSIDE the container on port 80.

        -p 80 - the port utilized by rutorrent
        -p 3000 - the port utilized by flood
        -p 5000 - rtorrent scgi port
        -p 50000 - the port rtorrent listens on
        -p 6881/udp - rtorrent DHT port
        -v /config - combined config file location
        -v /downloads - path to your downloads folder
        -e PGID for GroupID - see below for explanation
        -e PUID for UserID - see below for explanation
        -e TZ for timezone information, eg Europe/London

It is based on alpine linux with s6 overlay, for shell access whilst the container is running, do docker exec -it rutorrent-flood /bin/bash.

User / Group Identifiers:

Sometimes when using data volumes (-v flags) permissions issues can arise between the host OS and the container. We avoid this issue by allowing you to specify the user PUID and group PGID. Ensure the data volume directory on the host is owned by the same user you specify and it will "just work".

In this instance PUID=1001 and PGID=1001. To find yours use id user as below:

          $ id <dockeruser>
            uid=1001(dockeruser) gid=1001(dockergroup) groups=1001(dockergroup)

Setting up the application:

rTorrent configuration files for are in /config/rtorrent and the configuration for php in /config/php.

ruTorrent can be found at <your-ip>:80 and configuration options can be found in /config/rutorrent/settings.

Flood can be found at <your-ip>:3000 and configuration options can be found in the config.js file located in /config/flood.

Settings, changed by the user through the "Settings" panel in ruTorrent, are valid until rtorrent restart. After which all settings will be set according to the rtorrent config file (/config/rtorrent/rtorrent.rc),this is a limitation of the actual apps themselves.

Important note for unraid users or those running services such as a webserver on port 80, change port 80 assignment

** It should also be noted that this container when run will create subfolders ,completed, incoming and watched in the /downloads volume.**

The Port Assignments and configuration folder structure has been changed from the previous ubuntu based versions of this container and we recommend a clean install

Umask can be set in the /config/rtorrent/rtorrent.rc file by changing value in system.umask.set

If you are seeing this error Caught internal_error: 'DhtRouter::get_tracker did not actually insert tracker.'. , a possible fix is to disable dht in /config/rtorrent/rtorrent.rc by changing the following values:

        dht = disable
        peer_exchange = no
