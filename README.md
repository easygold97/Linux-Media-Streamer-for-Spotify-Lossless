
# 🎧 Linux Media Streamer for Spotify Lossless

A minimal, stable and headless Spotify streamer built on **Ubuntu Server 24.04 LTS**.

Runs the official Spotify Linux client in a lightweight environment with support for **Spotify Lossless**, automatic startup, and compatibility with any ALSA-supported audio device.

---

## 🚀 Features

- ✅ Ubuntu Server 24.04 LTS
- ✅ Fully headless (no desktop environment)
- ✅ No display manager
- ✅ Automatic start at boot
- ✅ Spotify Connect compatible
- ✅ Spotify Lossless support (if enabled on account)
- ✅ Works with any ALSA-compatible DAC
- ✅ Automatic restart on crash
- ✅ Minimal system footprint
- ✅ Optimized audio path (no unnecessary resampling)

---

## 📦 Requirements

- Ubuntu Server 24.04 LTS (clean installation)
- x86_64 mini PC
- Spotify Premium account (Lossless enabled if available)
- USB DAC / PCIe sound card / HDMI audio (ALSA compatible)
- Internet connection (Wi-Fi or Ethernet)
- OpenSSH enabled during Ubuntu installation

During installation:
- Enable **OpenSSH**
- Do NOT install a desktop environment

---

# 1️⃣ System Update

```bash
sudo apt update
sudo apt upgrade -y
```

---

# 2️⃣ Install Official Spotify Client

```bash
curl -sS https://download.spotify.com/debian/pubkey_5384CE82BA52C83A.asc | sudo gpg --dearmor --yes -o /etc/apt/trusted.gpg.d/spotify.gpg

echo "deb https://repository.spotify.com stable non-free" | sudo tee /etc/apt/sources.list.d/spotify.list

sudo apt update
sudo apt install spotify-client
```

---

# 3️⃣ Install Minimal Graphical Dependencies

```bash
sudo apt install xorg xvfb dbus-x11 pulseaudio
```

---

# 4️⃣ First Login (One-Time Setup)

```bash
nano ~/.xinitrc
```

Insert:

```
exec dbus-run-session spotify --disable-gpu --no-sandbox
```

Start:

```bash
startx
```

Log into Spotify and enable Lossless if available:

Settings → Audio Quality → Enable Lossless

Close X when finished.

---

# 5️⃣ Headless Launch Script

```bash
sudo nano /usr/local/bin/spotify-headless.sh
```

Insert:

```bash
#!/bin/bash

export DISPLAY=:99
export XDG_RUNTIME_DIR=/run/user/$(id -u)

Xvfb :99 -screen 0 1280x720x24 -nolisten tcp &
XVFB_PID=$!

sleep 2

exec dbus-run-session spotify --disable-gpu --no-sandbox

kill $XVFB_PID
```

Make executable:

```bash
sudo chmod +x /usr/local/bin/spotify-headless.sh
```

---

# 6️⃣ Create systemd USER Service

```bash
mkdir -p ~/.config/systemd/user
nano ~/.config/systemd/user/spotify-headless.service
```

Insert:

```ini
[Unit]
Description=Spotify Headless Streamer
After=network.target sound.target

[Service]
ExecStart=/usr/local/bin/spotify-headless.sh
Restart=always
RestartSec=5

[Install]
WantedBy=default.target
```

Enable lingering:

```bash
sudo loginctl enable-linger $USER
```

Enable service:

```bash
systemctl --user daemon-reload
systemctl --user enable spotify-headless
systemctl --user start spotify-headless
```

---

# 7️⃣ Audio Configuration

List audio devices:

```bash
aplay -l
```

List Pulse sinks:

```bash
pactl list short sinks
```

Set preferred device:

```bash
pactl set-default-sink SINK_NAME
```

Make permanent:

```bash
mkdir -p ~/.config/pulse
nano ~/.config/pulse/default.pa
```

Insert:

```
set-default-sink SINK_NAME
```

---

# 🔊 Audio Optimization

Edit:

```bash
sudo nano /etc/pulse/daemon.conf
```

Recommended:

```
default-sample-rate = 44100
alternate-sample-rate = 48000
avoid-resampling = yes
resample-method = trivial
```

Restart:

```bash
systemctl --user restart spotify-headless
```

---

# 🔍 Verify Audio Path

During playback:

```bash
cat /proc/asound/cardX/stream0
```

Expected:

```
Format: S16_LE
Momentary freq = 44100 Hz
```

---

# 🧹 Optional Cleanup

```bash
sudo systemctl disable --now cups
sudo systemctl disable --now cups-browsed
sudo systemctl disable --now ModemManager
sudo systemctl disable --now multipathd
sudo systemctl disable --now packagekit
```

---

# 🏗 Architecture

Ubuntu Server  
→ systemd user service  
→ Xvfb  
→ dbus-run-session  
→ Spotify official client (Lossless)  
→ PulseAudio  
→ ALSA  
→ DAC  
→ Amplifier  

---

Enjoy your Linux-based Spotify Lossless streamer 🎶
