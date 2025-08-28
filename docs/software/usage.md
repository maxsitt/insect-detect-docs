# Usage

If you run into any problems, please create a
[GitHub issue](https://github.com/maxsitt/insect-detect/issues){target=_blank}.

??? bug "RuntimeError: X_LINK_DEVICE_ALREADY_IN_USE"

    ``` bash
    RuntimeError: Failed to connect to device, error message: X_LINK_DEVICE_ALREADY_IN_USE
    ```

    If you run into the error shown above, the most common cause is that a
    previously started script is already communicating with the OAK camera
    (e.g. cron job at boot). You can show all currently running processes by
    using the process manager [htop](https://en.wikipedia.org/wiki/Htop){target=_blank}.

    Start htop by running:

    ``` bash
    htop
    ```

    If you see one of the Python scripts in the list of processes, you can hit
    ++f9++ with the script selected. This will open the `SIGTERM` option and by
    confirming with ++enter++ the process will be stopped. Close htop by pressing
    ++q++ and you should now be able to connect to the OAK camera again.

---

## Web App

You can use the Insect Detect
[web app](https://github.com/maxsitt/insect-detect/blob/main/webapp.py){target=_blank}
to stream frames from the OAK camera via HTTP to the browser-based interface
(based on the [NiceGUI](https://github.com/zauberzeug/nicegui/){target=_blank} framework).
It includes real-time camera control (e.g. manual focus) and customization of all
[configuration](https://github.com/maxsitt/insect-detect/blob/main/configs/config_custom.yaml){target=_blank}
parameters, as well as selecting the
[active config](https://github.com/maxsitt/insect-detect/blob/main/configs/config_selector.yaml){target=_blank}
that is used by both the web app and the recording script.

Run the [`webapp.py`](https://github.com/maxsitt/insect-detect/blob/main/webapp.py){target=_blank}
script with the Python interpreter from the virtual environment where you
installed the required packages (e.g. `env_insdet`):

``` bash
env_insdet/bin/python3 insect-detect/webapp.py
```

Use the link that is printed to the Terminal to open the web app in your browser.
You can use any device that is connected to the same network as the Raspberry Pi
(e.g. phone or tablet) to open the web app. If the link based on your hostname
does not work, use the RPi's IP address instead.

If the Raspberry Pi is connected to a hotspot from your phone, you can use the
[Network Analyzer](https://play.google.com/store/apps/details?id=net.techet.netanalyzerlite.an&hl=en){target=_blank}
app to find its IP address.

Stop the web app by either using the button in your browser window
or by pressing ++ctrl+c++ in the Terminal.

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

For fully autonomous monitoring in the field, enable `auto_run` in the `startup`
settings of your custom config file and add `trigger_capture.py` as
`primary` script. This will run the recording script automatically after each boot
(triggered by the power management board).

**Processing pipeline:**

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
