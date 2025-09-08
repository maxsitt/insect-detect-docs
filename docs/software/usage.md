# Usage

## Wi-Fi vs RPi Hotspot

To connect to the Raspberry Pi via SSH or open the web app in a browser,
both your device and the RPi need to be connected to the same network.
You have three main options:

1. Wi-Fi network that both your device and the RPi are connected to.
2. Hotspot from your device that the RPi is connected to.
3. Hotspot from the RPi that your device is connected to.

The RPi can usually only be used *either* as wireless client (connects to
network with internet access) *or* as access point (creates own hotspot without
internet access). For an **unsupported** experimental workaround, check the
[RaspAP Docs](https://docs.raspap.com/features-experimental/ap-sta/){target=_blank}.

If you don't need internet access on your RPi (e.g. during fieldwork), there
are some advantages of letting the RPi create a hotspot: Any device can connect
to it and open the web app, which can be useful if multiple people want to set
up camera traps in the field. Also this option is more reliable than letting
the RPi connect to a hotspot from your device (every device is different!).

With the default `startup` configuration, the RPi will automatically start a
hotspot as fallback mechanism when no configured Wi-Fi network is available
(SSID and password = hostname).

---

## Web App

With the [web app](https://github.com/maxsitt/insect-detect/blob/main/webapp.py){target=_blank},
you can view the OAK camera live stream including detected and tracked insects
in the browser-based user interface
(based on [NiceGUI](https://github.com/zauberzeug/nicegui/){target=_blank}).
It also allows real-time camera control (e.g. setting manual focus) and customization of all
[configuration parameters](https://github.com/maxsitt/insect-detect/blob/main/configs/config_custom.yaml){target=_blank},
as well as selecting the
[active config](https://github.com/maxsitt/insect-detect/blob/main/configs/config_selector.yaml){target=_blank}
that is used by both the web app and the recording script.
When using the web app in the field while setting up your camera trap, you
can also save deployment metadata, such as location and background setting.
The `Advanced` section includes current system info (e.g. temperature,
CPU/RAM usage) and viewing of log files.

Run the [`webapp.py`](https://github.com/maxsitt/insect-detect/blob/main/webapp.py){target=_blank}
script with the Python interpreter from the virtual environment where you
installed the required packages (e.g. `env_insdet`):

``` bash
env_insdet/bin/python3 insect-detect/webapp.py
```

Use one of the links that is shown in the Terminal to open the web app in your
browser. If the hostname-based link does not work, use the IP address instead.
Always use the link with the IP address if your device is connected to the RPi
hotspot, this should be `http://10.42.0.1:5000` if not using https, or
`https://10.42.0.1:8443` if https is enabled.

Stop the web app with the `Stop App` button or by pressing ++ctrl+c++ in the Terminal.

---

## Recording Script

The [recording script](https://github.com/maxsitt/insect-detect/blob/main/trigger_capture.py){target=_blank}
can be used for fully automated insect monitoring in autonomous camera trap deployments.
All configuration parameters can be customized in the web app or by directly modifying the
[`config_custom.yaml`](https://github.com/maxsitt/insect-detect/tree/main/configs/config_custom.yaml){target=_blank}
file. You can generate multiple custom configuration files and select the active
config either in the web app or by modifying the
[`config_selector.yaml`](https://github.com/maxsitt/insect-detect/blob/main/configs/config_selector.yaml){target=_blank}.

Run the
[`trigger_capture.py`](https://github.com/maxsitt/insect-detect/blob/main/trigger_capture.py){target=_blank}
script with the Python interpreter from the virtual environment where you
installed the required packages (e.g. `env_insdet`):

``` bash
env_insdet/bin/python3 insect-detect/trigger_capture.py
```

Stop the script by pressing ++ctrl+c++ in the Terminal.

### Processing Pipeline

- A custom **YOLO insect detection model** is run in real time on device (OAK)
  and uses a continuous stream of downscaled LQ frames as input.
- An **object tracker** uses the bounding box coordinates of detected insects
  to assign a unique tracking ID to each individual present in the frame and
  track its movement through time.
- The tracker + model output from inference on LQ frames is synchronized with
  **MJPEG-encoded HQ frames** (default: 3840x2160 px) on device (OAK).
- The HQ frames are saved to the microSD card at the configured
  **capture intervals** while an insect is detected (triggered capture)
  and independent of detections (time-lapse capture).
- Corresponding **metadata** from the detection model and tracker output
  is saved to a metadata .csv file for each detected and tracked insect
  (including timestamp, label, confidence score, tracking ID, tracking status
  and bounding box coordinates).
- The bounding box coordinates can be used to **crop detected insects** from
  the corresponding HQ frames and save them as individual .jpg images.
  Depending on the post-processing configuration, the original HQ frames are
  optionally deleted to save storage space.
- If a power management board (Witty Pi 4 L3V7 or PiJuice Zero) is connected and
  enabled in the configuration, **intelligent power management** is activated which
  includes battery charge level monitoring with conditional recording durations.
- With the default configuration, running the recording consumes **~3.8 W** of power.

---

Depending on the specific requirements of your field deployment, different
behaviours can be configured in the `startup` settings that are run after
each boot:

### Use Case 1: Interactive

If you deploy the camera trap for only one recording session, e.g. while doing
other field work in parallel, you might want to just run the web app automatically
after boot to save deployment metadata and/or modify configuration parameters.
Click the `Start Rec` button in the web app after everything is set up to start
the recording script manually.

``` yaml hl_lines="6-8"
startup:
  hotspot_setup:
    enabled: true  # Create RPi Wi-Fi hotspot if it doesn't exist (uses hostname for SSID and password)
  network_setup:
    enabled: true  # Create/update all configured Wi-Fi networks in NetworkManager (including hotspot)
  auto_run:
    enabled: true       # Automatically run configured Python script(s) after boot
    primary: webapp.py  # Primary Python script in "insect-detect" directory that is run first
    fallback:           # Fallback Python script in "insect-detect" directory (can be empty)
    delay: 180          # Wait time (seconds) before stopping primary script and running fallback script
```

### Use Case 2: Autonomous

For long-term field deployment, you want to run the recording script
`trigger_capture.py` automatically after each boot. Please make sure to
modify and save all relevant configuration parameters either beforehand or
when setting up the camera trap.

``` yaml hl_lines="6-8"
startup:
  hotspot_setup:
    enabled: true  # Create RPi Wi-Fi hotspot if it doesn't exist (uses hostname for SSID and password)
  network_setup:
    enabled: true  # Create/update all configured Wi-Fi networks in NetworkManager (including hotspot)
  auto_run:
    enabled: true                # Automatically run configured Python script(s) after boot
    primary: trigger_capture.py  # Primary Python script in "insect-detect" directory that is run first
    fallback:                    # Fallback Python script in "insect-detect" directory (can be empty)
    delay: 180                   # Wait time (seconds) before stopping primary script and running fallback script
```

### Use Case 3: Hybrid

There might be scenarios where you want to optionally connect to the web app
after a scheduled wake-up time, e.g. to modify settings or check log files.
For this use case, you can set `webapp.py` as `primary` auto-run script and
`trigger_capture.py` as `fallback` script. If no user interaction (opening the
web app in your browser) is detected for the configured `delay` time, the
web app will be terminated and the recording script will be started automatically.

``` yaml hl_lines="6-10"
startup:
  hotspot_setup:
    enabled: true  # Create RPi Wi-Fi hotspot if it doesn't exist (uses hostname for SSID and password)
  network_setup:
    enabled: true  # Create/update all configured Wi-Fi networks in NetworkManager (including hotspot)
  auto_run:
    enabled: true                 # Automatically run configured Python script(s) after boot
    primary: webapp.py            # Primary Python script in "insect-detect" directory that is run first
    fallback: trigger_capture.py  # Fallback Python script in "insect-detect" directory (can be empty)
    delay: 180                    # Wait time (seconds) before stopping primary script and running fallback script
```

---

## Troubleshooting

If you run into any problems while using the
[`insect-detect`](https://github.com/maxsitt/insect-detect){target=_blank}
software, please create an
[issue](https://github.com/maxsitt/insect-detect/issues){target=_blank}.

??? bug "RuntimeError: X_LINK_DEVICE_ALREADY_IN_USE"

    ``` bash
    RuntimeError: Failed to connect to device, error message: X_LINK_DEVICE_ALREADY_IN_USE
    ```

    If you run into the error shown above, the most common cause is that a
    previously started script is already communicating with the OAK camera.
    Show all running processes by starting the process manager
    [htop](https://en.wikipedia.org/wiki/Htop){target=_blank}:

    ``` bash
    htop
    ```

    If you see one of the Python scripts in the list of processes, you can hit
    ++f9++ with the script selected. This will open the `SIGTERM` option and by
    confirming with ++enter++ the process will be stopped. Close htop by pressing
    ++q++ and you should now be able to connect to the OAK camera again.

??? bug "Couldn't read data from stream: 'frame' (X_LINK_ERROR)"

    ``` bash
    RuntimeError: Communication exception - possible device error/misconfiguration. Original message 'Couldnt read data from stream: 'frame' (X_LINK_ERROR)'
    ```

    If you run into the error shown above, the most common cause is a loose
    USB cable between OAK camera and Raspberry Pi. Other reasons can be a
    damaged cable or Micro-USB adapter. Please make sure to exclude these
    reasons, e.g. by testing a different cable and/or adapter.
