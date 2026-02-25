
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

---

# 🛡 Bonus Point – OS-Level Watchdog (Optional)

For increased reliability, you can enable a kernel-level watchdog to automatically reboot the system in case of freeze, prolonged network failure, or critical memory conditions.

This is especially useful for unattended installations.

---

## Why OS-Level Watchdog?

This adds protection against:

- Kernel freeze
- System hang
- Network loss for extended periods
- Severe memory exhaustion

It does NOT rely on CPU load (important for low-power systems).

---

## 1️⃣ Enable systemd Runtime Watchdog (Recommended)

Edit:

    sudo nano /etc/systemd/system.conf

Add or uncomment:

    RuntimeWatchdogSec=30

Apply changes:

    sudo systemctl daemon-reexec

This activates the kernel watchdog via systemd.
If the system becomes unresponsive for 30 seconds, it will reboot automatically.

This method is lightweight and safe for low-CPU hardware.

---

## 2️⃣ (Optional) Install Linux Watchdog Service

Install:

    sudo apt install watchdog

Load software watchdog module:

    sudo modprobe softdog
    echo softdog | sudo tee -a /etc/modules

Edit configuration:

    sudo nano /etc/watchdog.conf

Minimal safe configuration:

    watchdog-device = /dev/watchdog
    watchdog-timeout = 20
    interval = 10

    # Network check
    ping = 8.8.8.8
    interface = wlan0

    # Root filesystem activity
    file = /var/log/syslog

    # Memory safeguard (very low threshold)
    min-memory = 20

    # Do NOT enable max-load on low-power CPUs

Enable service:

    sudo systemctl enable watchdog
    sudo systemctl start watchdog

---

## Recommended Setup for Low-Power Hardware

For small CPUs (Atom, Celeron, etc.):

- Enable RuntimeWatchdogSec
- Enable network ping check
- Enable minimal memory threshold
- Do NOT enable max-load
- Do NOT use aggressive thresholds

This avoids false positives while maintaining automatic recovery capability.

---

## Recovery Strategy Overview

1. Spotify crash → systemd restarts service  
2. Service fails repeatedly → watchdog still active  
3. System freeze → kernel watchdog triggers reboot  
4. Long network outage → watchdog triggers reboot  

This creates a self-healing system suitable for unattended use.


---

# 🚀 Faster Boot Optimization

## 1️⃣ Ensure Multi-User Target (No GUI)

Set default boot target:

    sudo systemctl set-default multi-user.target

---

## 2️⃣ Reduce Network Wait Time

If boot is delayed by network wait services:

    sudo systemctl disable systemd-networkd-wait-online.service

For NetworkManager systems:

    sudo systemctl disable NetworkManager-wait-online.service

---

## 3️⃣ Analyze Boot Performance

Measure boot time:

    systemd-analyze
    systemd-analyze blame

This helps identify slow services.

---

# ⚙️ CPU Governor Optimization

For consistent performance and lower audio jitter, set CPU governor to performance mode.

## Install cpufreq utilities

    sudo apt install cpufrequtils

## Set performance governor immediately

    sudo cpufreq-set -g performance

## Make it persistent

Edit:

    sudo nano /etc/default/cpufrequtils

Insert:

    GOVERNOR="performance"

Disable ondemand service:

    sudo systemctl disable ondemand

Reboot to apply permanently.

---

# 🎯 Recommended Configuration for Low-Power Hardware

For small CPUs:

- Use performance governor
- Disable unnecessary services
- Keep watchdog active
- Avoid aggressive memory or CPU watchdog thresholds

This ensures:

- Faster boot
- Stable playback
- Reduced frequency scaling latency
- Predictable system behavior

---

# 🏁 Expected Result

With these optimizations:

- Faster boot time
- Lower service overhead
- More consistent CPU frequency
- Improved long-term stability

Your system becomes closer to a dedicated commercial media streamer appliance.

---
