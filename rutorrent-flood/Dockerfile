From lsiobase/alpine:3.6
Maintainer p0rtal@flixxr.net

# Mediainfo version
Arg MEDIAINFO_VER="0.7.99"

# build dependencies to be removed after
Run \
apk add --no-cache --virtual=build-dependencies \
        autoconf \
        automake \
        cppunit-dev \
        curl-dev \
        file \
        g++ \
        gcc \
        git \
        libressl-dev \
        libtool \
        make \
        ncurses-dev && \

# runtime dependencies
apk add --no-cache \
         ca-certificates \
         curl \
         fcgi \
         ffmpeg \
         geoip \
         gzip \
         logrotate \
         nginx \
         nodejs-npm \
         php7 \
         php7-cgi \
         php7-fpm \
         php7-json \
         php7-mbstring \
         php7-pear \
         rtorrent \
         screen \
         tar \
         unrar \
         unzip \
         wget \
         zip && \

#Install Flood WebUI first
mkdir -p \
          /app/flood && \
flood_tag=$(curl -sX GET "https://api.github.com/repos/jfurrow/flood/releases/latest" \
          | awk '/tag_name/{print $4;exit}' FS='[""]') && \
curl -o \
/tmp/flood.tar.gz -L \
          "https://github.com/jfurrow/flood/archive/${flood_tag}.tar.gz" && \
tar xf \
/tmp/flood.tar.gz -C \
            /app/flood --strip-components=1 && \
cd /app/flood && \
npm install --production && \

# Set up flood's config.js file
cp /app/flood/config.template.js /app/flood/config.js && \
sed -i \
        -e "s#dbPath: '.*',#dbPath: '/config/flood/db/',#" \
        -e "s#socket: '.*'#socket: 'true',#" \
        -e "s#socketPath: '.*'#socketPath: '/run/php/.rtorrent.sock'#" \
/app/flood/config.js && \

# Install ruTorrent and dependencies next
mkdir -p \
        /usr/share/webapps/rutorrent \
        /defaults/rutorrent-conf && \
curl -o \
/tmp/rutorrent.tar.gz -L \
        "https://github.com/Novik/ruTorrent/archive/master.tar.gz" && \
tar xf \
/tmp/rutorrent.tar.gz -C \
          /usr/share/webapps/rutorrent --strip-components=1 && \
mv /usr/share/webapps/rutorrent/conf/* \
          /defaults/rutorrent-conf/ && \
rm -rf \
          /defaults/rutorrent-conf/users && \

          # compile mediainfo
curl -o \
          /tmp/libmediainfo.tar.gz -L \
          "http://mediaarea.net/download/binary/libmediainfo0/${MEDIAINFO_VER}/MediaInfo_DLL_${MEDIAINFO_VER}_GNU_FromSource.tar.gz" && \
curl -o \
          /tmp/mediainfo.tar.gz -L \
          	"https://mediaarea.net/download/binary/mediainfo/${MEDIAINFO_VER}/MediaInfo_CLI_${MEDIAINFO_VER}_GNU_FromSource.tar.gz" && \
mkdir -p \
          /tmp/libmediainfo \
          /tmp/mediainfo && \
tar xf /tmp/libmediainfo.tar.gz -C \
          /tmp/libmediainfo --strip-components=1 && \
tar xf /tmp/mediainfo.tar.gz -C \
          /tmp/mediainfo --strip-components=1 && \

cd /tmp/libmediainfo && \
          ./SO_Compile.sh && \
cd /tmp/libmediainfo/ZenLib/Project/GNU/Library && \
          	make install && \
cd /tmp/libmediainfo/MediaInfoLib/Project/GNU/Library && \
          	make install && \
cd /tmp/mediainfo && \
          ./CLI_Compile.sh && \
cd /tmp/mediainfo/MediaInfo/Project/GNU/CLI && \
          	make install && \

# remove build-dependencies
apk del --purge \
            build-dependencies && \
rm -rf \
            /root \
            /tmp/* \
            /etc/nginx/conf.d/default.conf && \
mkdir -p \
            /root && \

#fix logrotate
sed -i "s#/var/log/message {}.*# #g" /etc/logrotate.conf

# Copy in files
Copy root/ /

#Declare Ports and Volumes
EXPOSE 80 3000
VOLUME /config /downloads
