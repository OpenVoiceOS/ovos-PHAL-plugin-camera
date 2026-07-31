# Camera Plugin for OVOS PHAL

This plugin gives OVOS access to a camera. It uses OpenCV on most systems and libcamera on Raspberry Pi. The plugin can take snapshots, serve a video stream over HTTP, and respond to camera commands sent over the message bus.

## Features

- Detect and use a compatible camera system: libcamera on Raspberry Pi, or OpenCV on other systems.
- Open and close the camera on demand.
- Capture frames and save them to a file, or return them as base64-encoded strings.
- Serve a video stream as an MJPEG feed over HTTP.

---

## HiveMind Support

You can use this plugin in OVOS and with [HiveMind](https://github.com/JarbasHiveMind) satellites.

Allow `"ovos.phal.camera.pong"` in your HiveMind so your satellite can report camera support:

```bash
hivemind-core allow-msg "ovos.phal.camera.pong"
```

---

## Installation

1. Install the plugin:

   ```bash
   pip install ovos-phal-plugin-camera
   ```

2. Add the plugin to your OVOS PHAL configuration:

   ```json
   {
       "PHAL": {
           "ovos-phal-plugin-camera": {
               "video_source": 0,
               "start_open": false,
               "serve_mjpeg": false,
               "mjpeg_host": "0.0.0.0",
               "mjpeg_port": 5000
           }
       }
   }
   ```

### Additional Steps for Raspberry Pi Users

On Raspberry Pi, the Picamera2 library needs the `libcamera` package. Raspberry Pi OS installs `libcamera` system-wide, so a Python virtual environment (venv) needs extra setup to reach it.

These examples use the default venv location from ovos-installer, `~/.venvs/ovos`. Adjust the path if your setup differs.

#### Steps to Enable `libcamera` in Your Virtual Environment

1. Install the required system packages:

   ```bash
   sudo apt install -y python3-libcamera python3-kms++ libcap-dev
   ```

2. Open the `pyvenv.cfg` file in your virtual environment directory:

   ```bash
   nano ~/.venvs/ovos/pyvenv.cfg
   ```

   Add or update this line:

   ```plaintext
   include-system-site-packages = true
   ```

   Save the file and close the editor.

3. Activate your virtual environment:

   ```bash
   source ~/.venvs/ovos/bin/activate
   ```

   Check that `libcamera` is accessible:

   ```bash
   python3 -c "import libcamera; print('libcamera is accessible')"
   ```

#### Why These Steps Are Necessary

The `libcamera` package is not on PyPI. Raspberry Pi OS installs it system-wide. A virtual environment isolates itself from system-wide Python packages by default, so these steps let the plugin reach `libcamera` while the venv still isolates other packages.

#### Notes

- These steps apply only to Raspberry Pi users who use the Picamera2 library. On other platforms, the plugin uses OpenCV, which needs no extra configuration.
- Before you run these steps, confirm `libcamera` is installed on your Raspberry Pi:

  ```bash
  libcamera-still --version
  ```

---

## Configuration Options

| Option         | Type   | Default   | Description                                           |
| -------------- | ------ | --------- | ----------------------------------------------------- |
| `video_source` | `int`  | `0`       | Index of the video source to use for the camera.      |
| `start_open`   | `bool` | `false`   | Whether to open the camera at plugin startup.         |
| `serve_mjpeg`  | `bool` | `false`   | Whether to start an MJPEG server for video streaming. |
| `mjpeg_port`   | `int`  | `5000`    | Port for the MJPEG server.                            |

---

## Bus Events

### Handled Events

| Event Name               | Description                       | Payload                     |
| ------------------------ | --------------------------------- | ---------------------------- |
| `ovos.phal.camera.open`  | Opens the camera.                 | None                        |
| `ovos.phal.camera.close` | Closes the camera.                | None                        |
| `ovos.phal.camera.get`   | Captures a frame from the camera. | `{ "path": "<file_path>" }` |

### Emitted Events

| Event Name                      | Description                      | Payload                                                           |
| -------------------------------- | -------------------------------- | ------------------------------------------------------------------ |
| `ovos.phal.camera.get.response` | Response for the captured frame. | `{ "path": "<file_path>" }` or `{ "b64_frame": "<base64_data>" }` |

---

## Usage

### Open the Camera

Send this message to open the camera:

```python
bus.emit(Message("ovos.phal.camera.open"))
```

### Close the Camera

Send this message to close the camera:

```python
bus.emit(Message("ovos.phal.camera.close"))
```

### Capture a Frame

Send this message to capture a frame:

```python
bus.emit(Message("ovos.phal.camera.get", {"path": "/path/to/save/image.jpg"}))
```

If you do not provide a `path`, the plugin returns the frame as a base64-encoded string.

### MJPEG Server

If you enable `serve_mjpeg` in the configuration, the MJPEG feed is available at:

```
http://<mjpeg_host>:<mjpeg_port>/video_feed
```

You can use the MJPEG feed to integrate this camera [into Home Assistant](https://www.home-assistant.io/integrations/mjpeg/).

---

## Related Projects

- [OpenVoiceOS/ovos-skill-camera](https://github.com/OpenVoiceOS/ovos-skill-camera): the OVOS skill that uses this plugin.
- [JarbasHiveMind](https://github.com/JarbasHiveMind): the satellite system this plugin can report camera support to.

---

## License

This project is licensed under the [Apache 2.0 License](LICENSE).
