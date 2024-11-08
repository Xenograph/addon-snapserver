ARG BUILD_FROM

FROM alpine:3.17 AS builder

RUN apk -U add \
        alsa-lib-dev \
        autoconf \
        automake \
        avahi-dev \
        build-base \
        dbus \
        ffmpeg-dev \
        git \
        libconfig-dev \
        libgcrypt-dev \
        libplist-dev \
        libressl-dev \
        libsndfile-dev \
        libsodium-dev \
        libtool \
        mosquitto-dev \
        popt-dev \
        soxr-dev \
        xxd

##### ALAC #####
RUN git clone https://github.com/mikebrady/alac
WORKDIR /alac
RUN autoreconf -i
RUN ./configure
RUN make
RUN make install
WORKDIR /
##### ALAC END #####

##### NQPTP #####
RUN git clone https://github.com/mikebrady/nqptp
WORKDIR /nqptp
RUN git checkout 1.2.4
RUN autoreconf -i
RUN ./configure
RUN make
WORKDIR /
##### NQPTP END #####

##### SPS #####
RUN git clone https://github.com/mikebrady/shairport-sync
WORKDIR /shairport-sync
RUN git checkout 4.3.4
WORKDIR /shairport-sync/build
RUN autoreconf -i ../
RUN CFLAGS="-O3" CXXFLAGS="-O3" ../configure --sysconfdir=/etc --with-soxr --with-avahi --with-ssl=openssl \
        --with-airplay-2 --with-metadata --with-dummy --with-pipe --with-dbus-interface \
        --with-stdout --with-mpris-interface --with-mqtt-client \
        --with-apple-alac --with-convolution
RUN make -j $(nproc)
RUN DESTDIR=install make install
WORKDIR /
##### SPS END #####

FROM $BUILD_FROM

# Set shell
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

# Copy build files.
COPY --from=builder /shairport-sync/build/install/usr/local/bin/shairport-sync /usr/local/bin/shairport-sync
COPY --from=builder /shairport-sync/build/install/usr/local/share/man/man1 /usr/share/man/man1
COPY --from=builder /nqptp/nqptp /usr/local/bin/nqptp
COPY --from=builder /usr/local/lib/libalac.* /usr/local/lib/
COPY --from=builder /shairport-sync/build/install/etc/shairport-sync.conf /etc/
COPY --from=builder /shairport-sync/build/install/etc/shairport-sync.conf.sample /etc/
COPY --from=builder /shairport-sync/build/install/etc/dbus-1/system.d/shairport-sync-dbus.conf /etc/dbus-1/system.d/
COPY --from=builder /shairport-sync/build/install/etc/dbus-1/system.d/shairport-sync-mpris.conf /etc/dbus-1/system.d/

COPY ./etc/s6-overlay/s6-rc.d /etc/s6-overlay/s6-rc.dRUN chmod +x /etc/s6-overlay/s6-rc.d/01-startup/script.sh
        
#Install Librespot
RUN apk add --no-cache --repository=http://dl-cdn.alpinelinux.org/alpine/edge/testing/ librespot=0.6.0-r0

#Install Snapcast
RUN apk add --no-cache --repository=http://dl-cdn.alpinelinux.org/alpine/edge/community/ snapcast-server=0.29.0-r0

#Install Snapweb
RUN \
    wget https://github.com/badaix/snapweb/releases/download/v0.8.0/snapweb.zip && \
    unzip -o snapweb.zip -d /usr/share/snapserver/snapweb/ && \
    rm snapweb.zip

# Copy data for add-on
COPY run.sh /
RUN chmod a+x /run.sh

CMD [ "/run.sh" ]
