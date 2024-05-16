############################################################
# Dockerfile that contains SteamCMD and WineHQ
############################################################
FROM ubuntu:noble as build_stage

LABEL maintainer="github@diyagi.dev"
ARG PUID=1001

ENV USER steam
ENV HOMEDIR "/home/${USER}"
ENV STEAMCMDDIR "${HOMEDIR}/steamcmd"

RUN set -x \
	# Install, update & upgrade packages
	&& apt-get update \
	&& apt-get install -y --no-install-recommends --no-install-suggests \
		lib32stdc++6=14-20240412-0ubuntu1 \
		lib32gcc-s1=14-20240412-0ubuntu1 \
		ca-certificates=20240203 \
		nano=7.2-2build1 \
		curl=8.5.0-2ubuntu10.1 \
		locales=2.39-0ubuntu8.1 \
	&& sed -i -e 's/# en_US.UTF-8 UTF-8/en_US.UTF-8 UTF-8/' /etc/locale.gen \
	&& dpkg-reconfigure --frontend=noninteractive locales \
	# Create unprivileged user
	&& useradd -u "${PUID}" -m "${USER}" \
	# Download SteamCMD, execute as user
	&& su "${USER}" -c \
		"mkdir -p \"${STEAMCMDDIR}\" \
                && curl -fsSL 'https://steamcdn-a.akamaihd.net/client/installer/steamcmd_linux.tar.gz' | tar xvzf - -C \"${STEAMCMDDIR}\" \
                && \"./${STEAMCMDDIR}/steamcmd.sh\" +quit \
                && ln -s \"${STEAMCMDDIR}/linux32/steamclient.so\" \"${STEAMCMDDIR}/steamservice.so\" \
                && mkdir -p \"${HOMEDIR}/.steam/sdk32\" \
                && ln -s \"${STEAMCMDDIR}/linux32/steamclient.so\" \"${HOMEDIR}/.steam/sdk32/steamclient.so\" \
                && ln -s \"${STEAMCMDDIR}/linux32/steamcmd\" \"${STEAMCMDDIR}/linux32/steam\" \
                && mkdir -p \"${HOMEDIR}/.steam/sdk64\" \
                && ln -s \"${STEAMCMDDIR}/linux64/steamclient.so\" \"${HOMEDIR}/.steam/sdk64/steamclient.so\" \
                && ln -s \"${STEAMCMDDIR}/linux64/steamcmd\" \"${STEAMCMDDIR}/linux64/steam\" \
                && ln -s \"${STEAMCMDDIR}/steamcmd.sh\" \"${STEAMCMDDIR}/steam.sh\"" \
	# Symlink steamclient.so; So misconfigured dedicated servers can find it
 	&& ln -s "${STEAMCMDDIR}/linux64/steamclient.so" "/usr/lib/x86_64-linux-gnu/steamclient.so" \
	&& rm -rf /var/lib/apt/lists/*


FROM build_stage as winehq

ARG WINE_BRANCH=devel
ARG WINE_VERSION=9.8~noble-1
# https://wiki.winehq.org/Mono#Versions
ARG WINE_MONO_VERSION=9.1.0

# Install WineHQ
RUN dpkg --add-architecture i386 \
	&& mkdir -pm755 /etc/apt/keyrings \
	&& curl -fsSL -o /etc/apt/keyrings/winehq-archive.key https://dl.winehq.org/wine-builds/winehq.key \
	&& curl -fsSL -o /etc/apt/sources.list.d/winehq-noble.sources https://dl.winehq.org/wine-builds/ubuntu/dists/noble/winehq-noble.sources \
    && apt-get update \
    && apt-get install -y \
		winbind=2:4.19.5+dfsg-4ubuntu9 \
        xvfb=2:21.1.12-1ubuntu1 \
		xz-utils \
	&& apt-get install -y --no-install-recommends --no-install-suggests \
		winehq-${WINE_BRANCH}=${WINE_VERSION} \
    && rm -rf /var/lib/apt/lists/*

# Install wine-mono
# Stupid wine, i took days figuring out why it was timing out on boot...
# turns out wine need mono, and it does not say anything in the terminal,
# but shows an dialog box inside the dummy screen...
# i had to take an screenshot of the video buffer to figure this shit out
RUN mkdir -p /opt/wine/mono \
	&& curl -fsSL -o wine-mono-${WINE_MONO_VERSION}-x86.tar.xz https://dl.winehq.org/wine/wine-mono/${WINE_MONO_VERSION}/wine-mono-${WINE_MONO_VERSION}-x86.tar.xz \
	&& tar -xf wine-mono-${WINE_MONO_VERSION}-x86.tar.xz -C /opt/wine/mono \
	&& rm wine-mono-${WINE_MONO_VERSION}-x86.tar.xz


FROM winehq AS noble-root
WORKDIR ${STEAMCMDDIR}

FROM winehq AS noble
# Switch to user
USER ${USER}
