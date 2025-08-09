FROM ubuntu:22.04

ENV DEBIAN_FRONTEND=noninteractive
ENV USER=root

# Install desktop environment, VNC, terminal, and clipboard support
RUN apt update && \
    apt install -y xfce4 xfce4-goodies tightvncserver xfce4-terminal autocutsel && \
    apt clean

# Create VNC startup script with clipboard manager
RUN mkdir -p /root/.vnc && \
    echo '#!/bin/bash\n'\
    'xrdb $HOME/.Xresources\n'\
    'autocutsel -fork &\n'\
    'startxfce4 &' > /root/.vnc/xstartup && \
    chmod +x /root/.vnc/xstartup
RUN apt purge -y firefox snapd && \
    apt update && \
    apt install -y software-properties-common && \
    add-apt-repository -y ppa:mozillateam/ppa && \
    echo 'Package: firefox*' > /etc/apt/preferences.d/mozillateamppa && \
    echo 'Pin: release o=LP-PPA-mozillateam' >> /etc/apt/preferences.d/mozillateamppa && \
    echo 'Pin-Priority: 1001' >> /etc/apt/preferences.d/mozillateamppa && \
    apt update && \
    apt install -y firefox && \
    apt clean

EXPOSE 5901

# Start VNC server with defined resolution
ENV VNC_GEOMETRY=1280x800

CMD ["sh", "-c", "vncserver :1 -geometry ${VNC_GEOMETRY} -depth 24 && tail -F /root/.vnc/*.log"]
