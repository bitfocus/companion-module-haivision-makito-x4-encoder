# Haivision Makito X4 Encoder

Control and monitor [Haivision Makito X4 Encoder](https://www.haivision.com/)
devices from Companion via the device's REST API.

## Configuration

| Field              | Description                                                                 |
| ------------------ | --------------------------------------------------------------------------- |
| **Device IP**      | IP address of the Makito X4 Encoder.                                        |
| **Port**           | HTTPS/HTTP port. Use `443` for HTTPS (default); any other value uses HTTP.  |
| **Username**       | Device login (default `admin`).                                             |
| **Password**       | Device password.                                                            |
| **Enable Polling** | When on, the module periodically refreshes status, variables and feedbacks. |
| **Poll Interval**  | How often to poll, in seconds (1–60, default 5).                            |

The module authenticates with the device using a session cookie. Self-signed
certificates are accepted. If the connection status stays red, double-check the
IP, port, and credentials, then look at the connection log for details.

## What you can do

**Encoders** (the X4 has four video encoders, shown as Encoder 1–4):

- Start, Stop, Toggle, and Restart each encoder.
- Set bitrate, resolution, framerate/GOP, and codec (H.264 / H.265).
- Show a live thumbnail of each encoder on a button.

**Streams** (UDP / RTP / SRT / RTSP):

- Create, Edit, and Delete streams.
- Start, Stop, Toggle, and Restart streams.

**System presets** (device `.cfg` configuration presets):

- Save, Load, Delete, Rename, Duplicate, set Startup preset, and toggle Autosave.

**Device:**

- Enable/disable the preview service, Reboot the device, and a Custom API Call
  action for any endpoint not covered above.

## Variables, feedbacks and presets

The module exposes variables for device info, each encoder's state/codec/
bitrate/resolution/framerate/input, and up to ten streams. Feedbacks can color
buttons by encoder state, bitrate comparison, resolution/framerate/codec match,
connection status, and show encoder thumbnails. A set of ready-made presets is
included under categories like _Encoder Control_, _Encoder Settings_,
_Encoder Thumbnails_, _System Presets_, and _Status_.

## Notes

- Destructive actions (Reboot, Delete Stream, Delete Preset) require ticking a
  **Confirm** checkbox before they run.
- Stream destinations are configured through the **Create Stream** / **Edit
  Stream** actions.
