# Camera Plugin for OVOS PHAL

This plugin allows users to interact with cameras using OpenCV or libcamera, take snapshots, and serve video streams over HTTP. It also provides methods for handling camera operations via message bus events.

## Features

- Detect and use compatible camera systems (libcamera on Raspberry Pi or OpenCV on other systems).
- Open and close the camera dynamically.
- Capture frames and save them to a file or return them as base64-encoded strings.
- Serve video streams as an MJPEG feed over HTTP.

---

## Installation

1. Install required dependencies:

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

---

## Configuration Options

| Option         | Type   | Default   | Description                                           |
| -------------- | ------ | --------- | ----------------------------------------------------- |
| `video_source` | `int`  | `0`       | Index of the video source to use for the camera.      |
| `start_open`   | `bool` | `false`   | Whether to open the camera at plugin startup.         |
| `serve_mjpeg`  | `bool` | `false`   | Whether to start an MJPEG server for video streaming. |
| `mjpeg_host`   | `str`  | `0.0.0.0` | Host address for the MJPEG server.                    |
| `mjpeg_port`   | `int`  | `5000`    | Port for the MJPEG server.                            |

---

## Bus Events

### Handled Events

| Event Name               | Description                       | Payload                     |
| ------------------------ | --------------------------------- | --------------------------- |
| `ovos.phal.camera.open`  | Opens the camera.                 | None                        |
| `ovos.phal.camera.close` | Closes the camera.                | None                        |
| `ovos.phal.camera.get`   | Captures a frame from the camera. | `{ "path": "<file_path>" }` |

### Emitted Events

| Event Name                      | Description                      | Payload                                                           |
| ------------------------------- | -------------------------------- | ----------------------------------------------------------------- |
| `ovos.phal.camera.get.response` | Response for the captured frame. | `{ "path": "<file_path>" }` or `{ "b64_frame": "<base64_data>" }` |

---

## Usage

### Open the Camera

Send the following message to open the camera:

```python
bus.emit(Message("ovos.phal.camera.open"))
```

### Close the Camera

Send the following message to close the camera:

```python
bus.emit(Message("ovos.phal.camera.close"))
```

### Capture a Frame

Send the following message to capture a frame:

```python
bus.emit(Message("ovos.phal.camera.get", {"path": "/path/to/save/image.jpg"}))
```

If the `path` is not provided, the frame will be returned as a base64-encoded string.

### MJPEG Server

If the `serve_mjpeg` option is enabled in the configuration, the MJPEG feed will be accessible at:

```
http://<mjpeg_host>:<mjpeg_port>/video_feed
```

You can use the MJPEG feed to integrate this camera [into Home Assistant](https://www.home-assistant.io/integrations/mjpeg/)


---

## License

This project is licensed under the [Apache 2.0 License](LICENSE).
