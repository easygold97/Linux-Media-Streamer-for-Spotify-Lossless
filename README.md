# Linux-Media-Streamer-for-Spotify-Lossless
Headless Spotify streamer built on Ubuntu Server 24.04 LTS.  Runs the official Spotify client in a minimal, stable, headless environment with support for Spotify Lossless and any ALSA-compatible audio device.

🚀 Features

✅ Ubuntu Server 24.04 LTS

✅ Headless (no desktop environment)

✅ No display manager

✅ Automatic start at boot

✅ Spotify Connect compatible

✅ Spotify Lossless support (when available on account)

✅ Works with any ALSA-compatible DAC

✅ Automatic restart on crash

✅ Minimal system footprint

✅ Audio path optimized (no unnecessary resampling)

📦 Requirements

Ubuntu Server 24.04 LTS (clean install)

x86_64 mini PC

Spotify Premium account (Lossless enabled if available)

USB DAC / PCIe sound card / HDMI audio (ALSA compatible)

Internet connection (Wi-Fi or Ethernet)

OpenSSH enabled during installation

During Ubuntu installation:

Enable OpenSSH

Do NOT install a desktop environment

1️⃣ System Update
sudo apt update
sudo apt upgrade -y
2️⃣ Install Spotify Official Client

Add official repository:

curl -sS https://download.spotify.com/debian/pubkey_5384CE82BA52C83A.asc | \
sudo gpg --dearmor --yes -o /etc/apt/trusted.gpg.d/spotify.gpg

echo "deb https://repository.spotify.com stable non-free" | \
sudo tee /etc/apt/sources.list.d/spotify.list

Install:

sudo apt update
sudo apt install spotify-client
3️⃣ Install Minimal Graphical Dependencies

We only need a virtual X server:

sudo apt install xorg xvfb dbus-x11 pulseaudio

No desktop environment is required.

4️⃣ First Login (One-Time Setup)

Temporarily connect a monitor and keyboard.

Create:

nano ~/.xinitrc

Insert:

exec dbus-run-session spotify --disable-gpu --no-sandbox

Start:

startx

Log into Spotify.

Inside Spotify settings:

Settings → Audio Quality → Enable Lossless (if available)

Close X.

Credentials are stored in:

~/.config/spotify

The system can now operate fully headless.

5️⃣ Headless Launch Script

Create:

sudo nano /usr/local/bin/spotify-headless.sh

Insert:

#!/bin/bash

export DISPLAY=:99
export XDG_RUNTIME_DIR=/run/user/$(id -u)

Xvfb :99 -screen 0 1280x720x24 -nolisten tcp &
XVFB_PID=$!

sleep 2

exec dbus-run-session spotify --disable-gpu --no-sandbox

kill $XVFB_PID

Make executable:

sudo chmod +x /usr/local/bin/spotify-headless.sh
6️⃣ Create systemd USER Service (Important)

⚠️ Do NOT use a system-wide service.

mkdir -p ~/.config/systemd/user
nano ~/.config/systemd/user/spotify-headless.service

Insert:

[Unit]
Description=Spotify Headless Streamer
After=network.target sound.target

[Service]
ExecStart=/usr/local/bin/spotify-headless.sh
Restart=always
RestartSec=5

[Install]
WantedBy=default.target

Enable user lingering:

sudo loginctl enable-linger $USER

Enable service:

systemctl --user daemon-reload
systemctl --user enable spotify-headless
systemctl --user start spotify-headless
7️⃣ Audio Configuration (Generic for Any DAC)
Detect audio devices
aplay -l
List Pulse sinks
pactl list short sinks

Example:

alsa_output.usb-My_DAC-00.analog-stereo
alsa_output.pci-0000_00_1b.0.hdmi-stereo
Set preferred device
pactl set-default-sink SINK_NAME

Make permanent:

mkdir -p ~/.config/pulse
nano ~/.config/pulse/default.pa

Insert:

set-default-sink SINK_NAME
🔊 Audio Optimization

Edit:

sudo nano /etc/pulse/daemon.conf

For standard 44.1kHz DAC:

default-sample-rate = 44100
alternate-sample-rate = 48000
avoid-resampling = yes
resample-method = trivial

For hi-res DAC:

default-sample-rate = 48000
alternate-sample-rate = 44100
avoid-resampling = yes

Restart service:

systemctl --user restart spotify-headless
🔍 Verify Real Audio Path

During playback:

cat /proc/asound/cardX/stream0

Expected example:

Format: S16_LE
Momentary freq = 44100 Hz
🔊 Volume Recommendation

Set digital volume to 100%:

pactl set-sink-volume SINK_NAME 100%

Control volume from amplifier for maximum quality.

🧹 Optional System Cleanup

Disable unnecessary services:

sudo systemctl disable --now cups
sudo systemctl disable --now cups-browsed
sudo systemctl disable --now ModemManager
sudo systemctl disable --now multipathd
sudo systemctl disable --now packagekit
🏗 Architecture Overview

Ubuntu Server
→ systemd user service
→ Xvfb
→ dbus-run-session
→ Spotify official client (Lossless)
→ PulseAudio (user session)
→ ALSA
→ DAC
→ Amplifier

⚠️ Troubleshooting
Only "auto_null" sink appears

Use systemd user service, not system-wide service.

No audio output

Check:

pactl list short sinks
Verify Spotify running
systemctl --user status spotify-headless
Restart service
systemctl --user restart spotify-headless
🎯 Project Goals

Stable

Minimal

Headless

Lossless capable

Hardware agnostic

Easy to replicate
