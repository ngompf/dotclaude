---
name: upload
description: Build and upload firmware to the ESP32 over WiFi (OTA). Use when the user asks to upload, flash, or deploy firmware.
disable-model-invocation: true
allowed-tools: [Bash]
---

Build and flash the firmware to the ESP32 over WiFi:

```bash
pio run -e rymcu-esp32-devkitc-ota -t upload
```

Run the command above and report whether the upload succeeded or failed.

## Post-upload validation

After a successful upload, validate all HTTP endpoints are responding:

```bash
IP=$(grep 'upload_port' ota_config.generated.ini | grep -oP '= \K.+')
echo "=== GET /led ===" && curl -s --connect-timeout 5 "http://$IP/led?id=0"
echo ""
echo "=== Flash red ===" && curl -s --connect-timeout 5 -X POST "http://$IP/led?id=0&state=on&r=255&g=0&b=0" -w "HTTP %{http_code}"
sleep 1
echo ""
echo "=== Flash green ===" && curl -s --connect-timeout 5 -X POST "http://$IP/led?id=0&state=on&r=0&g=255&b=0" -w "HTTP %{http_code}"
sleep 1
echo ""
echo "=== LED off ===" && curl -s --connect-timeout 5 -X POST "http://$IP/led?id=0&state=off" -w "HTTP %{http_code}"
```

Report the response from each endpoint. `GET /led` should return JSON, the LED POST steps should each return HTTP 200.

## Troubleshooting

### OTA fails with Error 7 (can't reach ESP32)
The ESP32 must have been previously flashed via USB with firmware that connects to WiFi. If OTA has never worked, do a USB flash first (see below).

### USB device not found (`/dev/ttyUSB0` missing)
The ESP32 isn't being passed through to the container. On WSL2, attach it from Windows PowerShell (as admin):
```powershell
usbipd list
usbipd bind --busid <busid>
usbipd attach --wsl --busid <busid>
```

### Permission denied on `/dev/ttyUSB0`
```bash
sudo chmod a+rw /dev/ttyUSB0
```
For a permanent fix: `sudo usermod -a -G dialout $USER` (requires container restart to take effect).

### First-time USB flash (before OTA is available)
```bash
sudo chmod a+rw /dev/ttyUSB0
pio run -e rymcu-esp32-devkitc -t upload
```
Once the device boots and connects to WiFi, OTA will work for subsequent uploads.
