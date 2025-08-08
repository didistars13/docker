# Run Docker Container with VNC# Run Docker Container with VNC

## 1. Generate VNC Password File
We first set up a minimal container to **generate a VNC password file**.  
This file will be used by the VNC server to authenticate users.

```bash
docker run --rm -it -v "${PWD}:/output" ubuntu:22.04 bash -c "\
  apt update && apt install -y tightvncserver > /dev/null && \
  echo 'Enter your VNC password below:' && \
  vncpasswd /output/vnc-passwd"
```

After running this command:
- You’ll be prompted to enter a VNC password.
- The password will be stored in the file `vnc-passwd` in your current directory.
- Keep it safe — it will be used to connect to the VNC server running in the container.

---

## 2. Create the Dockerfile
Now we create the Dockerfile to build the VNC server image.

```bash
cat <<EOF > Dockerfile
FROM ubuntu:22.04

ENV DEBIAN_FRONTEND=noninteractive
ENV USER=root

# Install desktop environment and VNC server
RUN apt update && \
    apt install -y xfce4 xfce4-goodies tightvncserver xfce4-terminal && \
    apt clean

# Create VNC startup script
RUN mkdir -p /root/.vnc && \
    echo '#!/bin/bash\n'\
    'xrdb $HOME/.Xresources\n'\
    'startxfce4 &' > /root/.vnc/xstartup && \
    chmod +x /root/.vnc/xstartup

EXPOSE 5901

# Start VNC server when container runs
CMD ["sh", "-c", "vncserver :1 -geometry 1280x800 -depth 24 && tail -F /root/.vnc/*.log"]
EOF
```

---

## 3. Build the Image
```bash
docker build -t my-vnc-image .
```

---

## 4. Run the VNC Server Container
```bash
docker run -d -p 5901:5901 \
  -v ${PWD}/vnc-passwd:/root/.vnc/passwd \
  --name my-vnc my-vnc-image
```

---

## 5. Notes
- `vnc-passwd` should belong to the `root` user and have `0600` permissions:
```bash
sudo chown root:root vnc-passwd
sudo chmod 600 vnc-passwd
```
- To **scale resolution**, update the Dockerfile `CMD` line:
```bash
CMD ["sh", "-c", "vncserver :1 -geometry 1920x1080 -depth 24 && tail -F /root/.vnc/*.log"]
```
- Press **F8** inside TigerVNC to check settings.

---

## 6. Enable Clipboard Support
If you want **Ctrl+C / Ctrl+V** to work between your PC and the container, install `autocutsel`:

```dockerfile
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

EXPOSE 5901

# Start VNC server with defined resolution
ENV VNC_GEOMETRY=1280x800

CMD ["sh", "-c", "vncserver :1 -geometry ${VNC_GEOMETRY} -depth 24 && tail -F /root/.vnc/*.log"]
```

---

## 7. Install a Browser (Example: Firefox)
To install Firefox in Ubuntu without Snap:

**Manual install inside container:**
```bash
apt install -y wget gnupg software-properties-common && \
    add-apt-repository -y ppa:mozillateam/ppa && \
    apt update && \
    apt install -y firefox && \
    apt clean
```

**Add to Dockerfile:**
```dockerfile
RUN apt remove -y firefox snapd && \
    apt update && \
    apt install -y wget gnupg software-properties-common && \
    add-apt-repository -y ppa:mozillateam/ppa && \
    apt update && \
    apt install -y firefox && \
    apt clean
```
