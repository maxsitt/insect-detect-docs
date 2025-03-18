# Programming

!!! warning "Outdated Script Versions"

    The versions of the Python scripts shown in this subsection are currently
    **not up-to-date** with the versions in the
    [`insect-detect`](https://github.com/maxsitt/insect-detect){target=_blank}
    GitHub repo. Will be updated as soon as possible.

!!! tip "Adapt the software to your use case"

    You will find all Python scripts to deploy the **Insect Detect** DIY camera
    trap for automated insect monitoring in this section, together with
    suggestions on possible modifications. Click on the :material-plus-circle:
    symbol to open the code annotations for more information. More details
    about the API that is used can be found at the
    [Luxonis Docs](https://docs.luxonis.com/software/){target=_blank}.

If you run into any problems, find a bug or something that could be optimized, please
create a [GitHub issue](https://github.com/maxsitt/insect-detect/issues){target=_blank}.

??? bug "RuntimeError: X_LINK_DEVICE_ALREADY_IN_USE"

    ``` bash
    RuntimeError: Failed to connect to device, error message: X_LINK_DEVICE_ALREADY_IN_USE
    ```

    If you try to run one of the following Python scripts and the error above
    occurs, the most common cause is that a previously started script is already
    running and communicating with the OAK camera (e.g. cron job at boot). You
    can see all currently running processes by using Raspberry Pi's task manager
    [htop](https://en.wikipedia.org/wiki/Htop){target=_blank}. Start the task
    manager by running:

    ``` bash
    htop
    ```

    If you see one of the Python scripts in the list of processes, you can hit
    ++f9++ with the script selected. This will open the `SIGTERM` option and by
    confirming with ++enter++ the process will be stopped. Close htop by pressing
    ++q++ and you should now be able to successfully run the script.

---

## OAK camera preview

The following Python script will create and configure the
[ColorCamera](https://docs.luxonis.com/software/depthai-components/nodes/color_camera/){target=_blank}
node to send downscaled LQ frames (e.g. 320x320 px) to the host (Raspberry Pi)
and show them in a new window. If you are connected to the RPi via SSH,
[X11 forwarding](pisetup.md#set-up-x11-forwarding){target=_blank}
has to be set up, together with an started and active
[X server](localsetup.md#vcxsrv-windows-x-server){target=_blank}
to show the frames in a window on your local PC.

Run the script with the Python interpreter from the virtual environment where
you installed the required packages (e.g. `env_insdet`):

``` bash
env_insdet/bin/python3 insect-detect/cam_preview.py
```

??? info "Optional arguments"

    Add after `env_insdet/bin/python3 insect-detect/cam_preview.py`, separated by space:

    - `-af CM_MIN CM_MAX` set auto focus range in cm (min distance, max distance)
    - `-big` show a bigger preview window with 640x640 px size

Stop the script by pressing ++ctrl+c++ in the Terminal
or by hitting ++q++ with the preview window selected.

``` py title="cam_preview.py" linenums="1" hl_lines="12-15 41 44 47 50 63 88-89"
#!/usr/bin/env python3

"""Show OAK camera livestream.

Source:   https://github.com/maxsitt/insect-detect
License:  GNU GPLv3 (https://choosealicense.com/licenses/gpl-3.0/)
Author:   Maximilian Sittinger (https://github.com/maxsitt)
Docs:     https://maxsitt.github.io/insect-detect-docs/

- show downscaled LQ frames + fps in a new window (e.g. via X11 forwarding)
- optional arguments:
  '-af'  set auto focus range in cm (min distance, max distance)
         -> e.g. '-af 14 20' to restrict auto focus range to 14-20 cm
  '-big' show a bigger preview window with 640x640 px size (default: 320x320 px)
         -> decreases frame rate to ~3 fps (default: ~11 fps)

based on open source scripts available at https://github.com/luxonis
"""

import argparse
import time

import cv2
import depthai as dai

from utils.oak_cam import set_focus_range

# Define optional arguments
parser = argparse.ArgumentParser()
parser.add_argument("-af", "--af_range", nargs=2, type=int,
    help="Set auto focus range in cm (min distance, max distance).", metavar=("CM_MIN", "CM_MAX"))
parser.add_argument("-big", "--big_preview", action="store_true",
    help="Show a bigger preview window with 640x640 px size (default: 320x320 px).")
args = parser.parse_args()

# Create depthai pipeline
pipeline = dai.Pipeline() # (1)!

# Create and configure color camera node and define output
cam_rgb = pipeline.create(dai.node.ColorCamera) # (2)!
#cam_rgb.setImageOrientation(dai.CameraImageOrientation.ROTATE_180_DEG) # (3) # rotate image 180°
cam_rgb.setResolution(dai.ColorCameraProperties.SensorResolution.THE_1080_P) # (4)!
if not args.big_preview:
    cam_rgb.setPreviewSize(320, 320)  # downscale frames -> LQ frames
else:
    cam_rgb.setPreviewSize(640, 640)
cam_rgb.setPreviewKeepAspectRatio(False) # (5) # stretch frames (16:9) to square (1:1)
cam_rgb.setInterleaved(False)  # planar layout
cam_rgb.setColorOrder(dai.ColorCameraProperties.ColorOrder.BGR)
cam_rgb.setFps(25)  # frames per second available for auto focus/exposure

xout_rgb = pipeline.create(dai.node.XLinkOut) # (6)!
xout_rgb.setStreamName("frame")
cam_rgb.preview.link(xout_rgb.input)

if args.af_range:
    # Create XLinkIn node to send control commands to color camera node
    xin_ctrl = pipeline.create(dai.node.XLinkIn)
    xin_ctrl.setStreamName("control")
    xin_ctrl.out.link(cam_rgb.inputControl)

# Connect to OAK device and start pipeline in USB2 mode
with dai.Device(pipeline, maxUsbSpeed=dai.UsbSpeed.HIGH) as device: # (7)!

    # Create output queue to get the frames from the output defined above
    q_frame = device.getOutputQueue(name="frame", maxSize=4, blocking=False) # (8)!

    if args.af_range:
        # Create input queue to send control commands to OAK camera
        q_ctrl = device.getInputQueue(name="control", maxSize=16, blocking=False)

        # Set auto focus range to specified cm values
        af_ctrl = set_focus_range(args.af_range[0], args.af_range[1])
        q_ctrl.send(af_ctrl)

    # Set start time of recording and create counter to measure fps
    start_time = time.monotonic()
    counter = 0

    while True:
        # Get LQ frames and show in new window together with fps
        if q_frame.has():
            frame_lq = q_frame.get().getCvFrame()

            counter += 1
            fps = round(counter / (time.monotonic() - start_time), 2)

            cv2.putText(frame_lq, f"fps: {fps}", (4, frame_lq.shape[0] - 10),
                        cv2.FONT_HERSHEY_SIMPLEX, 0.7, (255, 255, 255), 2) # (9)!
            cv2.imshow("cam_preview", frame_lq)

        # Stop script and close window by pressing "Q"
        if cv2.waitKey(1) == ord("q"):
            break

```

1.  More info about the [Pipeline](https://docs.luxonis.com/projects/api/en/latest/components/pipeline/){target=_blank}
    and creation of [nodes](https://docs.luxonis.com/projects/api/en/latest/components/nodes/){target=_blank}.
2.  More info about the
    [ColorCamera](https://docs.luxonis.com/projects/api/en/latest/components/nodes/color_camera/){target=_blank}
    node and possible configurations.
3.  If your image is shown upside down, you can rotate it
    on-device by activating this configuration (remove the `#`).
4.  `THE_1080_P` = 1920x1080 pixel (aspect ratio: 16:9). You can check all
    [supported sensors](https://docs.luxonis.com/projects/hardware/en/latest/pages/articles/supported_sensors.html){target=_blank}
    and their respective resolutions at the DepthAI Docs. IMX378 is used in the OAK-1.
5.  More info about other possible
    [downscaling options](https://docs.luxonis.com/projects/api/en/latest/tutorials/maximize_fov/){target=_blank}.
6.  The [XLinkOut node](https://docs.luxonis.com/projects/api/en/latest/components/nodes/xlink_out/){target=_blank}
    sends data from the OAK device to the host (e.g. Raspberry Pi) via XLink.
7.  If your host (e.g. RPi Zero 2 W) has no USB 3 port or you aren't using a
    USB 3 cable, it is recommended to
    [force USB2 communication](https://docs.luxonis.com/en/latest/pages/troubleshooting/#forcing-usb2-communication){target=_blank}
    by setting `maxUsbSpeed=dai.UsbSpeed.HIGH`. Remove the `maxUsbSpeed` limit for full USB 3 support.
8.  You can specify different
    [queue configurations](https://docs.luxonis.com/projects/api/en/latest/components/device/#device-queues){target=_blank},
    by changing the maximum queue size or the blocking behaviour.
9.  More info about the
    [`cv2.putText()`](https://www.askpython.com/python-modules/opencv-puttext){target=_blank}
    function to customize your output.

---

## YOLO preview

With the following Python script you can run a custom YOLO object detection model
([.blob format](https://docs.luxonis.com/software/ai-inference/conversion){target=_blank})
on the OAK device with downscaled LQ frames (e.g. 320x320 px) as model input
and show the frames together with the model output (bounding box, label,
confidence score) in a new window.

Run the script with the Python interpreter from the virtual environment where
you installed the required packages (e.g. `env_insdet`):

``` bash
env_insdet/bin/python3 insect-detect/yolo_preview.py
```

??? info "Optional arguments"

    Add after `env_insdet/bin/python3 insect-detect/yolo_preview.py`, separated by space:

    - `-af CM_MIN CM_MAX` set auto focus range in cm (min distance, max distance)
    - `-ae` use bounding box coordinates from detections to set auto exposure region
    - `-log` print available Raspberry Pi memory, RPi CPU utilization +
      temperature, OAK memory + CPU usage and OAK chip temperature

Stop the script by pressing ++ctrl+c++ in the Terminal
or by hitting ++q++ with the preview window selected.

``` py title="yolo_preview.py" linenums="1" hl_lines="15-19 50-51 78 102-103 119 160 176 178-180"
#!/usr/bin/env python3

"""Show OAK camera livestream with detection model output.

Source:   https://github.com/maxsitt/insect-detect
License:  GNU GPLv3 (https://choosealicense.com/licenses/gpl-3.0/)
Author:   Maximilian Sittinger (https://github.com/maxsitt)
Docs:     https://maxsitt.github.io/insect-detect-docs/

- run a custom YOLO object detection model (.blob format) on-device (Luxonis OAK)
  -> inference on downscaled + stretched LQ frames (default: 320x320 px)
- show downscaled LQ frames + model output (bounding box, label, confidence) + fps
  in a new window (e.g. via X11 forwarding)
- optional arguments:
  '-af'  set auto focus range in cm (min distance, max distance)
         -> e.g. '-af 14 20' to restrict auto focus range to 14-20 cm
  '-ae'  use bounding box coordinates from detections to set auto exposure region
  '-log' print available Raspberry Pi memory, RPi CPU utilization + temperature,
         OAK memory + CPU usage and OAK chip temperature

based on open source scripts available at https://github.com/luxonis
"""

import argparse
import json
import logging
import time
from pathlib import Path

import cv2
import depthai as dai
from apscheduler.schedulers.background import BackgroundScheduler

from utils.general import frame_norm
from utils.log import print_logs
from utils.oak_cam import bbox_set_exposure_region, set_focus_range

# Define optional arguments
parser = argparse.ArgumentParser()
parser.add_argument("-af", "--af_range", nargs=2, type=int,
    help="Set auto focus range in cm (min distance, max distance).", metavar=("CM_MIN", "CM_MAX"))
parser.add_argument("-ae", "--bbox_ae_region", action="store_true",
    help="Use bounding box coordinates from detections to set auto exposure region.")
parser.add_argument("-log", "--print_logs", action="store_true",
    help=("Print RPi available memory, RPi CPU utilization + temperature, "
          "OAK memory + CPU usage and OAK chip temperature."))
args = parser.parse_args()

# Set file paths to the detection model and corresponding config JSON
MODEL_PATH = Path("insect-detect/models/yolov5n_320_openvino_2022.1_4shave.blob")
CONFIG_PATH = Path("insect-detect/models/json/yolov5_v7_320.json") # (1)!

# Get detection model metadata from config JSON
with CONFIG_PATH.open(encoding="utf-8") as config_json:
    config = json.load(config_json)
nn_config = config.get("nn_config", {})
nn_metadata = nn_config.get("NN_specific_metadata", {})
classes = nn_metadata.get("classes", {})
coordinates = nn_metadata.get("coordinates", {})
anchors = nn_metadata.get("anchors", {})
anchor_masks = nn_metadata.get("anchor_masks", {})
iou_threshold = nn_metadata.get("iou_threshold", {})
confidence_threshold = nn_metadata.get("confidence_threshold", {})
nn_mappings = config.get("mappings", {})
labels = nn_mappings.get("labels", {})

# Create depthai pipeline
pipeline = dai.Pipeline()

# Create and configure color camera node
cam_rgb = pipeline.create(dai.node.ColorCamera)
#cam_rgb.setImageOrientation(dai.CameraImageOrientation.ROTATE_180_DEG)  # rotate image 180°
cam_rgb.setResolution(dai.ColorCameraProperties.SensorResolution.THE_1080_P)
cam_rgb.setPreviewSize(320, 320)  # downscale frames for model input -> LQ frames
cam_rgb.setPreviewKeepAspectRatio(False)  # stretch frames (16:9) to square (1:1) for model input
cam_rgb.setInterleaved(False)  # planar layout
cam_rgb.setColorOrder(dai.ColorCameraProperties.ColorOrder.BGR)
cam_rgb.setFps(25) # (2) # frames per second available for auto focus/exposure and model input

# Get sensor resolution
SENSOR_RES = cam_rgb.getResolutionSize()

# Create detection network node and define input + outputs
nn = pipeline.create(dai.node.YoloDetectionNetwork) # (3)!
cam_rgb.preview.link(nn.input) # (4) # downscaled + stretched LQ frames as model input
nn.input.setBlocking(False) # (5)!

xout_rgb = pipeline.create(dai.node.XLinkOut)
xout_rgb.setStreamName("frame")
nn.passthrough.link(xout_rgb.input)

xout_nn = pipeline.create(dai.node.XLinkOut)
xout_nn.setStreamName("nn")
nn.out.link(xout_nn.input)

# Set detection model specific settings
nn.setBlobPath(MODEL_PATH)
nn.setNumClasses(classes)
nn.setCoordinateSize(coordinates)
nn.setAnchors(anchors)
nn.setAnchorMasks(anchor_masks)
nn.setIouThreshold(iou_threshold)
nn.setConfidenceThreshold(confidence_threshold) # (6)!
nn.setNumInferenceThreads(2)

if args.af_range or args.bbox_ae_region:
    # Create XLinkIn node to send control commands to color camera node
    xin_ctrl = pipeline.create(dai.node.XLinkIn)
    xin_ctrl.setStreamName("control")
    xin_ctrl.out.link(cam_rgb.inputControl)

# Connect to OAK device and start pipeline in USB2 mode
with dai.Device(pipeline, maxUsbSpeed=dai.UsbSpeed.HIGH) as device:

    if args.print_logs:
        # Print RPi + OAK info every second
        logging.getLogger("apscheduler").setLevel(logging.WARNING)
        scheduler = BackgroundScheduler()
        scheduler.add_job(print_logs, "interval", seconds=1, id="log") # (7)!
        scheduler.start()

        device.setLogLevel(dai.LogLevel.INFO)
        device.setLogOutputLevel(dai.LogLevel.INFO)

    # Create output queues to get the frames and detections from the outputs defined above
    q_frame = device.getOutputQueue(name="frame", maxSize=4, blocking=False)
    q_nn = device.getOutputQueue(name="nn", maxSize=4, blocking=False)

    if args.af_range or args.bbox_ae_region:
        # Create input queue to send control commands to OAK camera
        q_ctrl = device.getInputQueue(name="control", maxSize=16, blocking=False)

    if args.af_range:
        # Set auto focus range to specified cm values
        af_ctrl = set_focus_range(args.af_range[0], args.af_range[1])
        q_ctrl.send(af_ctrl)

    # Set start time of recording and create counter to measure fps
    start_time = time.monotonic()
    counter = 0

    while True:
        # Get LQ frames + model output (detections) and show in new window together with fps
        if q_frame.has() and q_nn.has():
            frame_lq = q_frame.get().getCvFrame()
            dets = q_nn.get().detections

            counter += 1
            fps = round(counter / (time.monotonic() - start_time), 2)

            for detection in dets:
                # Get bounding box from detection model
                bbox_orig = (detection.xmin, detection.ymin, detection.xmax, detection.ymax)
                bbox_norm = frame_norm(frame_lq, bbox_orig)

                # Get metadata from detection model
                label = labels[detection.label]
                det_conf = round(detection.confidence, 2)

                if args.bbox_ae_region and detection == dets[0]: # (8)!
                    # Use bbox from earliest detection to set auto exposure region
                    ae_ctrl = bbox_set_exposure_region(bbox_orig, SENSOR_RES)
                    q_ctrl.send(ae_ctrl)
                    # using bbox from latest detection (dets[-1]) is also possible,
                    # but can lead to "flickering effect" in some cases

                cv2.putText(frame_lq, label, (bbox_norm[0], bbox_norm[3] + 13),
                            cv2.FONT_HERSHEY_SIMPLEX, 0.4, (255, 255, 255), 1)
                cv2.putText(frame_lq, f"{det_conf}", (bbox_norm[0], bbox_norm[3] + 25),
                            cv2.FONT_HERSHEY_SIMPLEX, 0.4, (255, 255, 255), 1)
                cv2.rectangle(frame_lq, (bbox_norm[0], bbox_norm[1]),
                              (bbox_norm[2], bbox_norm[3]), (0, 0, 255), 2)

            cv2.putText(frame_lq, f"fps: {fps}", (4, frame_lq.shape[0] - 10),
                        cv2.FONT_HERSHEY_SIMPLEX, 0.7, (255, 255, 255), 2)
            cv2.imshow("yolo_preview", frame_lq)

            #print(f"fps: {fps}")
            # streaming the frames via SSH (X11 forwarding) will slow down fps
            # comment out "cv2.imshow()" and print fps to console for true fps

        # Stop script and close window by pressing "Q"
        if cv2.waitKey(1) == ord("q"):
            break

```

1.  Specify the path to your detection model (.blob format) and config JSON
    that the OAK can use it for on-device inference.
2.  For testing purposes, adjust the camera fps to the maximum fps of the YOLO model that is used for inference
    ([more info](https://docs.luxonis.com/projects/api/en/latest/tutorials/low-latency/#lowering-camera-fps-to-match-nn-fps){target=_blank}).
3.  More info about the
    [YoloDetectionNetwork node](https://docs.luxonis.com/projects/api/en/latest/components/nodes/yolo_detection_network/){target=_blank}.
4.  The downscaled LQ preview frames are used as model input. More info about
    [linking nodes](https://docs.luxonis.com/projects/api/en/latest/components/nodes/){target=_blank}.
5.  To avoid freezing of the pipeline, we will set
    [`blocking=False`](https://docs.luxonis.com/projects/api/en/latest/components/device/#output-queue-maxsize-and-blocking){target=_blank}
    for the frames that are used as model input.
6.  All metadata that is necessary to successfully run the model on-device is
    read from the corresponding config .json file. However, you could also change
    your IoU or confidence threshold here for experimenting with different settings.
7.  To match the output frequency of the OAK logs
    ([`device.setLogOutputLevel()`](https://docs.luxonis.com/projects/api/en/latest/tutorials/debugging/){target=_blank}),
    the interval for printing the RPi logs is set to one second. Increase this
    interval for easier reading of the logs.
8.  The `-ae` setting is still experimental and can lead to unexpected behaviour
    in some cases. By default, the bounding box coordinates from the earliest
    detection are used to set the auto exposure region.

---

## YOLO + object tracker preview

In the following Python script an
[ObjectTracker](https://docs.luxonis.com/software/depthai-components/nodes/object_tracker){target=_blank},
node based on the [Intel DL Streamer](https://dlstreamer.github.io/dev_guide/object_tracking.html){target=_blank}
framework, is added to the pipeline. The object tracker uses the detection model
output as input for tracking detected objects and assigning unique tracking IDs
on-device. The frames, together with the model and tracker output (bounding box
from tracker output and bbox from detection model, label, confidence score,
tracking ID, tracking status), are shown in a new window.

Run the script with the Python interpreter from the virtual environment where
you installed the required packages (e.g. `env_insdet`):

``` bash
env_insdet/bin/python3 insect-detect/yolo_tracker_preview.py
```

??? info "Optional arguments"

    Add after `env_insdet/bin/python3 insect-detect/yolo_tracker_preview.py`, separated by space:

    - `-af CM_MIN CM_MAX` set auto focus range in cm (min distance, max distance)
    - `-ae` use bounding box coordinates from detections to set auto exposure region
    - `-log` print available Raspberry Pi memory, RPi CPU utilization +
      temperature, OAK memory + CPU usage and OAK chip temperature

Stop the script by pressing ++ctrl+c++ in the Terminal
or by hitting ++q++ with the preview window selected.

``` py title="yolo_tracker_preview.py" linenums="1" hl_lines="17-21 52-53 80 102 179 192-195 199 201-203"
#!/usr/bin/env python3

"""Show OAK camera livestream with detection model and object tracker output.

Source:   https://github.com/maxsitt/insect-detect
License:  GNU GPLv3 (https://choosealicense.com/licenses/gpl-3.0/)
Author:   Maximilian Sittinger (https://github.com/maxsitt)
Docs:     https://maxsitt.github.io/insect-detect-docs/

- run a custom YOLO object detection model (.blob format) on-device (Luxonis OAK)
  -> inference on downscaled + stretched LQ frames (default: 320x320 px)
- use an object tracker to track detected objects and assign unique tracking IDs
  -> accuracy depends on object motion speed and inference speed of the detection model
- show downscaled LQ frames + model/tracker output (bounding box, label, confidence,
  tracking ID, tracking status) + fps in a new window (e.g. via X11 forwarding)
- optional arguments:
  '-af'  set auto focus range in cm (min distance, max distance)
         -> e.g. '-af 14 20' to restrict auto focus range to 14-20 cm
  '-ae'  use bounding box coordinates from detections to set auto exposure region
  '-log' print available Raspberry Pi memory, RPi CPU utilization + temperature,
         OAK memory + CPU usage and OAK chip temperature

based on open source scripts available at https://github.com/luxonis
"""

import argparse
import json
import logging
import time
from pathlib import Path

import cv2
import depthai as dai
from apscheduler.schedulers.background import BackgroundScheduler

from utils.general import frame_norm
from utils.log import print_logs
from utils.oak_cam import bbox_set_exposure_region, set_focus_range

# Define optional arguments
parser = argparse.ArgumentParser()
parser.add_argument("-af", "--af_range", nargs=2, type=int,
    help="Set auto focus range in cm (min distance, max distance).", metavar=("CM_MIN", "CM_MAX"))
parser.add_argument("-ae", "--bbox_ae_region", action="store_true",
    help="Use bounding box coordinates from detections to set auto exposure region.")
parser.add_argument("-log", "--print_logs", action="store_true",
    help=("Print RPi available memory, RPi CPU utilization + temperature, "
          "OAK memory + CPU usage and OAK chip temperature."))
args = parser.parse_args()

# Set file paths to the detection model and corresponding config JSON
MODEL_PATH = Path("insect-detect/models/yolov5n_320_openvino_2022.1_4shave.blob")
CONFIG_PATH = Path("insect-detect/models/json/yolov5_v7_320.json")

# Get detection model metadata from config JSON
with CONFIG_PATH.open(encoding="utf-8") as config_json:
    config = json.load(config_json)
nn_config = config.get("nn_config", {})
nn_metadata = nn_config.get("NN_specific_metadata", {})
classes = nn_metadata.get("classes", {})
coordinates = nn_metadata.get("coordinates", {})
anchors = nn_metadata.get("anchors", {})
anchor_masks = nn_metadata.get("anchor_masks", {})
iou_threshold = nn_metadata.get("iou_threshold", {})
confidence_threshold = nn_metadata.get("confidence_threshold", {})
nn_mappings = config.get("mappings", {})
labels = nn_mappings.get("labels", {})

# Create depthai pipeline
pipeline = dai.Pipeline()

# Create and configure color camera node
cam_rgb = pipeline.create(dai.node.ColorCamera)
#cam_rgb.setImageOrientation(dai.CameraImageOrientation.ROTATE_180_DEG)  # rotate image 180°
cam_rgb.setResolution(dai.ColorCameraProperties.SensorResolution.THE_1080_P)
cam_rgb.setPreviewSize(320, 320)  # downscale frames for model input -> LQ frames
cam_rgb.setPreviewKeepAspectRatio(False)  # stretch frames (16:9) to square (1:1) for model input
cam_rgb.setInterleaved(False)  # planar layout
cam_rgb.setColorOrder(dai.ColorCameraProperties.ColorOrder.BGR)
cam_rgb.setFps(25) # (1) # frames per second available for auto focus/exposure and model input

# Get sensor resolution
SENSOR_RES = cam_rgb.getResolutionSize()

# Create detection network node and define input
nn = pipeline.create(dai.node.YoloDetectionNetwork)
cam_rgb.preview.link(nn.input)  # downscaled + stretched LQ frames as model input
nn.input.setBlocking(False)

# Set detection model specific settings
nn.setBlobPath(MODEL_PATH)
nn.setNumClasses(classes)
nn.setCoordinateSize(coordinates)
nn.setAnchors(anchors)
nn.setAnchorMasks(anchor_masks)
nn.setIouThreshold(iou_threshold)
nn.setConfidenceThreshold(confidence_threshold)
nn.setNumInferenceThreads(2)

# Create and configure object tracker node and define inputs + outputs
tracker = pipeline.create(dai.node.ObjectTracker) # (2)!
tracker.setTrackerType(dai.TrackerType.ZERO_TERM_IMAGELESS) # (3)!
#tracker.setTrackerType(dai.TrackerType.SHORT_TERM_IMAGELESS)  # better for low fps
tracker.setTrackerIdAssignmentPolicy(dai.TrackerIdAssignmentPolicy.UNIQUE_ID)
nn.passthrough.link(tracker.inputTrackerFrame)
nn.passthrough.link(tracker.inputDetectionFrame)
nn.out.link(tracker.inputDetections)

xout_rgb = pipeline.create(dai.node.XLinkOut)
xout_rgb.setStreamName("frame")
tracker.passthroughTrackerFrame.link(xout_rgb.input)

xout_tracker = pipeline.create(dai.node.XLinkOut)
xout_tracker.setStreamName("track")
tracker.out.link(xout_tracker.input)

if args.af_range or args.bbox_ae_region:
    # Create XLinkIn node to send control commands to color camera node
    xin_ctrl = pipeline.create(dai.node.XLinkIn)
    xin_ctrl.setStreamName("control")
    xin_ctrl.out.link(cam_rgb.inputControl)

# Connect to OAK device and start pipeline in USB2 mode
with dai.Device(pipeline, maxUsbSpeed=dai.UsbSpeed.HIGH) as device:

    if args.print_logs:
        # Print RPi + OAK info every second
        logging.getLogger("apscheduler").setLevel(logging.WARNING)
        scheduler = BackgroundScheduler()
        scheduler.add_job(print_logs, "interval", seconds=1, id="log")
        scheduler.start()

        device.setLogLevel(dai.LogLevel.INFO)
        device.setLogOutputLevel(dai.LogLevel.INFO)

    # Create output queues to get the frames and tracklets (+ detections) from the outputs defined above
    q_frame = device.getOutputQueue(name="frame", maxSize=4, blocking=False)
    q_track = device.getOutputQueue(name="track", maxSize=4, blocking=False)

    if args.af_range or args.bbox_ae_region:
        # Create input queue to send control commands to OAK camera
        q_ctrl = device.getInputQueue(name="control", maxSize=16, blocking=False)

    if args.af_range:
        # Set auto focus range to specified cm values
        af_ctrl = set_focus_range(args.af_range[0], args.af_range[1])
        q_ctrl.send(af_ctrl)

    # Set start time of recording and create counter to measure fps
    start_time = time.monotonic()
    counter = 0

    while True:
        # Get LQ frames + tracker output (including detections) and show in new window together with fps
        if q_frame.has() and q_track.has():
            frame_lq = q_frame.get().getCvFrame()
            tracks = q_track.get().tracklets

            counter += 1
            fps = round(counter / (time.monotonic() - start_time), 2)

            for tracklet in tracks:
                # Get bounding box from passthrough detections
                bbox_orig = (tracklet.srcImgDetection.xmin, tracklet.srcImgDetection.ymin,
                             tracklet.srcImgDetection.xmax, tracklet.srcImgDetection.ymax)
                bbox_norm = frame_norm(frame_lq, bbox_orig)

                # Get bounding box from object tracker
                roi = tracklet.roi.denormalize(frame_lq.shape[1], frame_lq.shape[0])
                bbox_tracker = (int(roi.topLeft().x), int(roi.topLeft().y),
                                int(roi.bottomRight().x), int(roi.bottomRight().y))

                # Get metadata from tracker output (including passthrough detections)
                label = labels[tracklet.srcImgDetection.label]
                det_conf = round(tracklet.srcImgDetection.confidence, 2)
                track_id = tracklet.id
                track_status = tracklet.status.name

                if args.bbox_ae_region and tracklet == tracks[-1]: # (4)!
                    # Use model bbox from latest tracking ID to set auto exposure region
                    ae_ctrl = bbox_set_exposure_region(bbox_orig, SENSOR_RES)
                    q_ctrl.send(ae_ctrl)

                cv2.putText(frame_lq, label, (bbox_norm[0], bbox_norm[3] + 13),
                            cv2.FONT_HERSHEY_SIMPLEX, 0.4, (255, 255, 255), 1)
                cv2.putText(frame_lq, f"{det_conf}", (bbox_norm[0], bbox_norm[3] + 25),
                            cv2.FONT_HERSHEY_SIMPLEX, 0.4, (255, 255, 255), 1)
                cv2.putText(frame_lq, f"ID:{track_id}", (bbox_norm[0], bbox_norm[3] + 40),
                            cv2.FONT_HERSHEY_SIMPLEX, 0.5, (255, 255, 255), 1)
                cv2.putText(frame_lq, track_status, (bbox_norm[0], bbox_norm[3] + 50),
                            cv2.FONT_HERSHEY_SIMPLEX, 0.3, (255, 255, 255), 1)
                cv2.rectangle(frame_lq, (bbox_norm[0], bbox_norm[1]),
                              (bbox_norm[2], bbox_norm[3]), (0, 0, 255), 2)
                cv2.rectangle(frame_lq, (bbox_tracker[0], bbox_tracker[1]),
                              (bbox_tracker[2], bbox_tracker[3]), (0, 255, 130), 1) # (5)!

            cv2.putText(frame_lq, f"fps: {fps}", (4, frame_lq.shape[0] - 10),
                        cv2.FONT_HERSHEY_SIMPLEX, 0.7, (255, 255, 255), 2)
            cv2.imshow("tracker_preview", frame_lq)

            #print(f"fps: {fps}")
            # streaming the frames via SSH (X11 forwarding) will slow down fps
            # comment out "cv2.imshow()" and print fps to console for true fps

        # Stop script and close window by pressing "Q"
        if cv2.waitKey(1) == ord("q"):
            break

```

1.  Maximum fps of the detection model is slightly lower (~2 fps) when adding
    the object tracker.
2.  More info about the
    [ObjectTracker](https://docs.luxonis.com/projects/api/en/latest/components/nodes/object_tracker/){target=_blank} node.
3.  More info about the supported
    [tracking types](https://dlstreamer.github.io/dev_guide/object_tracking.html){target=_blank}.
4.  The `-ae` setting is still experimental and can lead to unexpected behaviour
    in some cases. By default, the bounding box coordinates from the latest
    tracking ID are used to set the auto exposure region.
5.  You can use the bounding box coordinates from the tracker output, as
    defined in this segment, and/or the bbox coordinates from the passthrough
    detections to draw the bboxes on the frame. The bboxes from the passthrough
    detections are usually more stable than from the object tracker output, you
    can decide for yourself which one is best for your use case.

---

## Automated monitoring script

!!! warning "Outdated Script Versions"

    The versions of the Python scripts shown in this subsection are currently
    **not up-to-date** with the versions in the
    [`insect-detect`](https://github.com/maxsitt/insect-detect){target=_blank}
    GitHub repo. Will be updated as soon as possible.

The following Python script is the main script for fully
[automated insect monitoring](../deployment/detection.md){target=_blank}.

All relevant configuration parameters can be modified in the
[`configs/config_custom.yaml`](https://github.com/maxsitt/insect-detect/tree/main/configs/config_custom.yaml) file.

Processing pipeline for the
[`yolo_tracker_save_hqsync.py`](https://github.com/maxsitt/insect-detect/blob/main/yolo_tracker_save_hqsync.py)
script that can be used for automated insect monitoring:

- A custom **YOLO insect detection model** is run in real time on device (OAK) and uses a
  continuous stream of downscaled LQ frames as input (default: 320x320 px at 20 fps).
- An **object tracker** uses the bounding box coordinates of detected insects to assign a unique
  tracking ID to each individual present in the frame and track its movement through time.
- The tracker + model output from inference on LQ frames is synchronized with
  **MJPEG-encoded HQ frames** (default: 3840x2160 px) on device (OAK) using the respective timestamps.
- The encoded HQ frames are saved to the microSD card at the configured intervals if
  an insect is detected (default: 1 s) and independent of detections (default: 10 min).
- Corresponding **metadata** from the detection model and tracker output (including timestamp, label,
  confidence score, tracking ID, tracking status and bounding box coordinates) is saved to a
  metadata .csv file for each detected and tracked insect at the configured interval.
- The metadata can be used to **crop detected insects** from the HQ frames and save them as individual
  .jpg images. Depending on the post-processing configuration, the original HQ frames will be optionally deleted after the processing to save storage space.
- During the recording, a maximum pipeline speed of **~19 FPS** for 4K resolution (3840x2160) and
  **~42 FPS** for 1080p resolution (1920x1080) can be reached if the capture interval is set to 0
  and the camera frame rate is adjusted accordingly.
- With the default configuration, the recording pipeline consumes **~3.8 W** of power.
- If a power management board (Witty Pi 4 L3V7 or PiJuice Zero) is connected and enabled in the
  configuration, intelligent power management is activated which includes battery charge level
  monitoring and conditional recording durations.

For fully automated monitoring in the field, set up a
[cron job](pisetup.md#schedule-cron-job){target=_blank} that will run the script
automatically after each boot.

Run the script with the Python interpreter from the virtual environment where
you installed the required packages (e.g. `env_insdet`):

``` bash
env_insdet/bin/python3 insect-detect/yolo_tracker_save_hqsync.py
```

Stop the script by pressing ++ctrl+c++ in the Terminal.

``` py title="yolo_tracker_save_hqsync_pijuice.py" linenums="1" hl_lines="32-55 103-104 107-108 112 115 118 133 138-147 151-152 195-197 216-217 222 301 318 329 338 366 402"
#!/usr/bin/env python3

"""Save cropped detections with associated metadata from detection model and object tracker.

Source:   https://github.com/maxsitt/insect-detect
License:  GNU GPLv3 (https://choosealicense.com/licenses/gpl-3.0/)
Author:   Maximilian Sittinger (https://github.com/maxsitt)
Docs:     https://maxsitt.github.io/insect-detect-docs/

- write info and error (+ traceback) messages to log file
- shut down Raspberry Pi without recording if free disk space or current PiJuice
  battery charge level are lower than the specified thresholds (default: 100 MB and 10%)
- duration of each recording interval conditional on current PiJuice battery charge level
  -> increases efficiency of battery usage and can prevent gaps in recordings
- create directory for each day, recording interval and object class to save images + metadata
- run a custom YOLO object detection model (.blob format) on-device (Luxonis OAK)
  -> inference on downscaled + stretched LQ frames (default: 320x320 px)
- use an object tracker to track detected objects and assign unique tracking IDs
  -> accuracy depends on object motion speed and inference speed of the detection model
- synchronize tracker output (including detections) from inference on LQ frames with
  HQ frames (default: 1920x1080 px) on-device using the respective message timestamps
  -> pipeline speed (= inference speed): ~13.4 fps (1080p sync) or ~3.4 fps (4K sync)
- save detections (bounding box area) cropped from HQ frames to .jpg at the
  specified capture frequency (default: 1 s), optionally together with full frames
- save corresponding metadata from tracker (+ model) output (time, label, confidence,
  tracking ID, relative bbox coordinates, .jpg file path) to .csv
- write info about recording interval (rec ID, start/end time, duration, number of cropped
  detections, unique tracking IDs, free disk space, battery charge level) to 'record_log.csv'
- shut down Raspberry Pi after recording interval is finished or if charge level or
  free disk space drop below the specified thresholds or if an error occurs
- optional arguments:
  '-4k'      crop detections from (+ save HQ frames in) 4K resolution (default: 1080p)
             -> decreases pipeline speed to ~3.4 fps (1080p: ~13.4 fps)
  '-af'      set auto focus range in cm (min distance, max distance)
             -> e.g. '-af 14 20' to restrict auto focus range to 14-20 cm
  '-ae'      use bounding box coordinates from detections to set auto exposure region
             -> can improve image quality of crops and thereby classification accuracy
  '-crop'    default:  save cropped detections with aspect ratio 1:1 ('-crop square') OR
             optional: keep original bbox size with variable aspect ratio ('-crop tight')
             -> '-crop square' increases bbox size on both sides of the minimum dimension,
                               or only on one side if object is localized at frame margin
                -> can increase classification accuracy by avoiding stretching of the
                   cropped insect image during resizing for classification inference
  '-full'    additionally save full HQ frames to .jpg (e.g. for training data collection)
             -> '-full det'  save full frame together with cropped detections
                             -> slightly decreases pipeline speed
             -> '-full freq' save full frame at specified frequency (default: 60 s)
  '-overlay' additionally save full HQ frames with overlays (bbox + info) to .jpg
             -> slightly decreases pipeline speed
  '-log'     write RPi CPU + OAK chip temperature, RPi available memory (MB) +
             CPU utilization (%) and battery info to .csv file at specified frequency
  '-zip'     store all captured data in an uncompressed .zip file for each day
             and delete original directory
             -> increases file transfer speed from microSD to computer
                but also on-device processing time and power consumption

based on open source scripts available at https://github.com/luxonis
"""

import argparse
import json
import logging
import subprocess
import threading
import time
from datetime import datetime, timedelta
from pathlib import Path

import depthai as dai
import psutil
from apscheduler.schedulers.background import BackgroundScheduler
from pijuice import PiJuice

from utils.general import frame_norm, zip_data
from utils.log import record_log, save_logs
from utils.oak_cam import bbox_set_exposure_region, set_focus_range
from utils.save_data import save_crop_metadata, save_full_frame, save_overlay_frame

# Define optional arguments
parser = argparse.ArgumentParser()
parser.add_argument("-4k", "--four_k_resolution", action="store_true",
    help="Set camera resolution to 4K (3840x2160 px) (default: 1080p).")
parser.add_argument("-af", "--af_range", nargs=2, type=int,
    help="Set auto focus range in cm (min distance, max distance).", metavar=("CM_MIN", "CM_MAX"))
parser.add_argument("-ae", "--bbox_ae_region", action="store_true",
    help="Use bounding box coordinates from detections to set auto exposure region.")
parser.add_argument("-crop", "--crop_bbox", choices=["square", "tight"], default="square", type=str,
    help=("Save cropped detections with aspect ratio 1:1 ('square') or "
          "keep original bbox size with variable aspect ratio ('tight')."))
parser.add_argument("-full", "--save_full_frames", choices=["det", "freq"], default=None, type=str,
    help="Additionally save full HQ frames to .jpg together with cropped detections ('det') "
         "or at specified frequency, independent of detections ('freq').")
parser.add_argument("-overlay", "--save_overlay_frames", action="store_true",
    help="Additionally save full HQ frames with overlays (bbox + info) to .jpg.")
parser.add_argument("-log", "--save_logs", action="store_true",
    help=("Write RPi CPU + OAK chip temperature, RPi available memory (MB) + "
          "CPU utilization (%%) and battery info to .csv file."))
parser.add_argument("-zip", "--zip_data", action="store_true",
    help="Store data in an uncompressed .zip file for each day and delete original directory.")
args = parser.parse_args()

# Set file paths to the detection model and corresponding config JSON
MODEL_PATH = Path("insect-detect/models/yolov5n_320_openvino_2022.1_4shave.blob")
CONFIG_PATH = Path("insect-detect/models/json/yolov5_v7_320.json")

# Set threshold values required to start and continue a recording
MIN_DISKSPACE = 100   # minimum free disk space (MB) (default: 100 MB)
MIN_CHARGELEVEL = 10  # minimum PiJuice battery charge level (default: 10%)

# Set capture frequency (default: 1 second)
# -> wait for specified amount of seconds between saving cropped detections + metadata
CAPTURE_FREQ = 1 # (1)!

# Set frequency for saving full frames if "-full freq" is used (default: 60 seconds)
FULL_FREQ = 60 # (2)!

# Set frequency for saving logs to .csv file if "-log" is used (default: 30 seconds)
LOG_FREQ = 30

# Set logging level and format, write logs to file
Path("insect-detect/data").mkdir(parents=True, exist_ok=True)
script_name = Path(__file__).stem
logging.basicConfig(level=logging.INFO, format="%(asctime)s - %(levelname)s: %(message)s",
                    filename=f"insect-detect/data/{script_name}_log.log", encoding="utf-8")
logger = logging.getLogger() # (3)!

# Instantiate PiJuice
pijuice = PiJuice(1, 0x14)

# Shut down Raspberry Pi if battery charge level or free disk space (MB) are lower than thresholds
chargelevel_start = pijuice.status.GetChargeLevel().get("data", -1)
disk_free = round(psutil.disk_usage("/").free / 1048576)
if chargelevel_start < MIN_CHARGELEVEL or disk_free < MIN_DISKSPACE: # (4)!
    logger.info("Shut down without recording | Charge level: %s%%\n", chargelevel_start)
    subprocess.run(["sudo", "shutdown", "-h", "now"], check=True)

# Set recording time conditional on PiJuice battery charge level
if chargelevel_start >= 70: # (5)!
    REC_TIME = 60 * 40             # PiJuice battery charge level > 70:  40 min
elif 50 <= chargelevel_start < 70:
    REC_TIME = 60 * 30             # PiJuice battery charge level 50-70: 30 min
elif 30 <= chargelevel_start < 50:
    REC_TIME = 60 * 20             # PiJuice battery charge level 30-50: 20 min
elif 15 <= chargelevel_start < 30:
    REC_TIME = 60 * 10             # PiJuice battery charge level 15-30: 10 min
else:
    REC_TIME = 60 * 5              # PiJuice battery charge level < 15:   5 min

# Optional: Disable charging of PiJuice battery if charge level is higher than threshold
#           -> can prevent overcharging and extend battery life
#if chargelevel_start > 80: # (6)
#    pijuice.config.SetChargingConfig({"charging_enabled": False})

# Get last recording ID from text file and increment by 1 (create text file for first recording)
rec_id_file = Path("insect-detect/data/last_rec_id.txt")
rec_id = int(rec_id_file.read_text(encoding="utf-8")) + 1 if rec_id_file.exists() else 1
rec_id_file.write_text(str(rec_id), encoding="utf-8")

# Create directory per day and recording interval to save images + metadata + logs
rec_start = datetime.now()
rec_start_format = rec_start.strftime("%Y-%m-%d_%H-%M-%S")
save_path = Path(f"insect-detect/data/{rec_start.date()}/{rec_start_format}")
save_path.mkdir(parents=True, exist_ok=True)
if args.save_full_frames is not None:
    (save_path / "full").mkdir(parents=True, exist_ok=True)
if args.save_overlay_frames:
    (save_path / "overlay").mkdir(parents=True, exist_ok=True)

# Get detection model metadata from config JSON
with CONFIG_PATH.open(encoding="utf-8") as config_json:
    config = json.load(config_json)
nn_config = config.get("nn_config", {})
nn_metadata = nn_config.get("NN_specific_metadata", {})
classes = nn_metadata.get("classes", {})
coordinates = nn_metadata.get("coordinates", {})
anchors = nn_metadata.get("anchors", {})
anchor_masks = nn_metadata.get("anchor_masks", {})
iou_threshold = nn_metadata.get("iou_threshold", {})
confidence_threshold = nn_metadata.get("confidence_threshold", {})
nn_mappings = config.get("mappings", {})
labels = nn_mappings.get("labels", {})

# Create folders for each object class to save cropped detections
for det_class in labels:
    (save_path / f"crop/{det_class}").mkdir(parents=True, exist_ok=True)

# Create depthai pipeline
pipeline = dai.Pipeline()

# Create and configure color camera node
cam_rgb = pipeline.create(dai.node.ColorCamera)
#cam_rgb.setImageOrientation(dai.CameraImageOrientation.ROTATE_180_DEG)  # rotate image 180°
cam_rgb.setResolution(dai.ColorCameraProperties.SensorResolution.THE_4_K)
if not args.four_k_resolution:
    cam_rgb.setIspScale(1, 2)     # downscale 4K to 1080p resolution -> HQ frames
cam_rgb.setPreviewSize(320, 320)  # downscale frames for model input -> LQ frames
cam_rgb.setPreviewKeepAspectRatio(False)  # stretch frames (16:9) to square (1:1) for model input
cam_rgb.setInterleaved(False)  # planar layout
cam_rgb.setColorOrder(dai.ColorCameraProperties.ColorOrder.BGR)
cam_rgb.setFps(25)  # frames per second available for auto focus/exposure and model input

# Get sensor resolution
SENSOR_RES = cam_rgb.getResolutionSize()

# Create detection network node and define input
nn = pipeline.create(dai.node.YoloDetectionNetwork)
cam_rgb.preview.link(nn.input)  # downscaled + stretched LQ frames as model input
nn.input.setBlocking(False)

# Set detection model specific settings
nn.setBlobPath(MODEL_PATH)
nn.setNumClasses(classes)
nn.setCoordinateSize(coordinates)
nn.setAnchors(anchors)
nn.setAnchorMasks(anchor_masks)
nn.setIouThreshold(iou_threshold)
nn.setConfidenceThreshold(confidence_threshold)
nn.setNumInferenceThreads(2)

# Create and configure object tracker node and define inputs
tracker = pipeline.create(dai.node.ObjectTracker)
tracker.setTrackerType(dai.TrackerType.ZERO_TERM_IMAGELESS)
#tracker.setTrackerType(dai.TrackerType.SHORT_TERM_IMAGELESS)  # better for low fps
tracker.setTrackerIdAssignmentPolicy(dai.TrackerIdAssignmentPolicy.UNIQUE_ID)
nn.passthrough.link(tracker.inputTrackerFrame)
nn.passthrough.link(tracker.inputDetectionFrame)
nn.out.link(tracker.inputDetections)

# Create and configure sync node and define inputs
sync = pipeline.create(dai.node.Sync)
sync.setSyncThreshold(timedelta(milliseconds=200))
cam_rgb.video.link(sync.inputs["frames"])  # HQ frames
tracker.out.link(sync.inputs["tracker"])   # tracker output

# Create message demux node and define input + outputs
demux = pipeline.create(dai.node.MessageDemux)
sync.out.link(demux.input)

xout_rgb = pipeline.create(dai.node.XLinkOut)
xout_rgb.setStreamName("frame")
demux.outputs["frames"].link(xout_rgb.input)  # synced HQ frames

xout_tracker = pipeline.create(dai.node.XLinkOut)
xout_tracker.setStreamName("track")
demux.outputs["tracker"].link(xout_tracker.input)  # synced tracker output

if args.af_range or args.bbox_ae_region:
    # Create XLinkIn node to send control commands to color camera node
    xin_ctrl = pipeline.create(dai.node.XLinkIn)
    xin_ctrl.setStreamName("control")
    xin_ctrl.out.link(cam_rgb.inputControl)

# Connect to OAK device and start pipeline in USB2 mode
with dai.Device(pipeline, maxUsbSpeed=dai.UsbSpeed.HIGH) as device:

    if args.save_logs or (args.save_full_frames == "freq"):
        logging.getLogger("apscheduler").setLevel(logging.WARNING)
        scheduler = BackgroundScheduler()
    else:
        scheduler = None

    if args.save_logs:
        # Write RPi + OAK + battery info to .csv file at specified frequency
        scheduler.add_job(save_logs, "interval", seconds=LOG_FREQ, id="log",
                          args=[device, rec_id, rec_start, save_path, pijuice])
        scheduler.start()

    if args.save_full_frames == "freq":
        # Save full HQ frame at specified frequency
        scheduler.add_job(save_full_frame, "interval", seconds=FULL_FREQ, id="full",
                          args=[None, save_path])
        if not scheduler.running:
            scheduler.start()

    # Write info on start of recording to log file
    logger.info("Rec ID: %s | Rec time: %s min | Charge level: %s%%",
                rec_id, int(REC_TIME / 60), chargelevel_start)

    # Create output queues to get the frames and tracklets (+ detections) from the outputs defined above
    q_frame = device.getOutputQueue(name="frame", maxSize=4, blocking=False)
    q_track = device.getOutputQueue(name="track", maxSize=4, blocking=False)

    if args.af_range or args.bbox_ae_region:
        # Create input queue to send control commands to OAK camera
        q_ctrl = device.getInputQueue(name="control", maxSize=16, blocking=False)

    if args.af_range:
        # Set auto focus range to specified cm values
        af_ctrl = set_focus_range(args.af_range[0], args.af_range[1])
        q_ctrl.send(af_ctrl)

    # Set start time of recording and create empty lists to save charge level and threads
    start_time = time.monotonic()
    chargelevel_list = []
    threads = []

    try:
        # Record until recording time is finished
        # Stop recording early if free disk space drops below threshold OR
        # if charge level dropped below threshold for 10 times
        while time.monotonic() < start_time + REC_TIME and disk_free > MIN_DISKSPACE and len(chargelevel_list) < 10: # (7)!

            # Get synchronized HQ frame + tracker output (including passthrough detections)
            if q_frame.has() and q_track.has():
                frame_hq = q_frame.get().getCvFrame()
                tracks = q_track.get().tracklets

                if args.save_full_frames == "freq":
                    # Save full HQ frame at specified frequency
                    scheduler.modify_job("full", args=[frame_hq, save_path])

                if args.save_overlay_frames:
                    # Copy frame for drawing overlays
                    frame_hq_copy = frame_hq.copy()

                for tracklet in tracks:
                    # Only use tracklets that are currently tracked (not "NEW", "LOST" or "REMOVED")
                    if tracklet.status.name == "TRACKED": # (8)!
                        # Get bounding box from passthrough detections
                        bbox_orig = (tracklet.srcImgDetection.xmin, tracklet.srcImgDetection.ymin,
                                     tracklet.srcImgDetection.xmax, tracklet.srcImgDetection.ymax)
                        bbox_norm = frame_norm(frame_hq, bbox_orig)

                        # Get metadata from tracker output (including passthrough detections)
                        label = labels[tracklet.srcImgDetection.label]
                        det_conf = round(tracklet.srcImgDetection.confidence, 2)
                        track_id = tracklet.id

                        if args.bbox_ae_region and tracklet == tracks[-1]:
                            # Use model bbox from latest tracking ID to set auto exposure region
                            ae_ctrl = bbox_set_exposure_region(bbox_orig, SENSOR_RES)
                            q_ctrl.send(ae_ctrl)

                        # Save detections cropped from HQ frame together with metadata
                        save_crop_metadata(frame_hq, bbox_norm, rec_id, label, det_conf, track_id,
                                           bbox_orig, rec_start_format, save_path, args.crop_bbox)

                        if args.save_full_frames == "det" and tracklet == tracks[-1]:
                            # Save full HQ frame
                            thread_full = threading.Thread(target=save_full_frame,
                                                           args=(frame_hq, save_path))
                            thread_full.start()
                            threads.append(thread_full)

                        if args.save_overlay_frames:
                            # Save full HQ frame with overlays
                            thread_overlay = threading.Thread(target=save_overlay_frame,
                                                              args=(frame_hq_copy, bbox_norm, label,
                                                                    det_conf, track_id, tracklet, tracks,
                                                                    save_path, args.four_k_resolution))
                            thread_overlay.start()
                            threads.append(thread_overlay)

            # Update free disk space (MB)
            disk_free = round(psutil.disk_usage("/").free / 1048576)

            # Update charge level (return "99" if not readable, add to list if lower than threshold)
            chargelevel = pijuice.status.GetChargeLevel().get("data", 99)
            if chargelevel < MIN_CHARGELEVEL:
                chargelevel_list.append(chargelevel)

            # Keep only active threads in list
            threads = [thread for thread in threads if thread.is_alive()]

            # Wait for specified amount of seconds (default: 1)
            time.sleep(CAPTURE_FREQ)

        # Write info on end of recording to log file
        logger.info("Recording %s finished | Charge level: %s%%\n", rec_id, chargelevel)

    except KeyboardInterrupt:
        # Write info on KeyboardInterrupt (Ctrl+C) to log file
        logger.info("Recording %s stopped by Ctrl+C | Charge level: %s%%\n", rec_id, chargelevel)

    except Exception:
        # Write info on error + traceback during recording to log file
        logger.exception("Error during recording %s | Charge level: %s%%", rec_id, chargelevel)

    finally:
        # Shut down scheduler (wait until currently executing jobs are finished)
        if scheduler:
            scheduler.shutdown()

        # Wait for active threads to finish
        for thread in threads:
            thread.join()

        # Write record logs to .csv file
        rec_end = datetime.now()
        record_log(rec_id, rec_start, rec_start_format, rec_end, save_path,
                   chargelevel_start, chargelevel) # (9)!

        if args.zip_data:
            # Store data in uncompressed .zip file and delete original folder
            zip_data(save_path)

        # (Re-)activate charging of PiJuice battery if charge level is lower than threshold
        if chargelevel < 80:
            pijuice.config.SetChargingConfig({"charging_enabled": True})

        # Shut down Raspberry Pi
        subprocess.run(["sudo", "shutdown", "-h", "now"], check=True) # (10)!

```

1.  In this line you can change the time interval with which the cropped
    detections will be saved to .jpg. This does not affect the detection model
    and object tracker speed, which are both run on-device even if no detections
    are saved.
2.  If you are using the optional argument `-full freq`, e.g. to collect training
    data, you can specify the frequency to save full frames in this line. Full frames
    will be saved at the specified frequency, even if no detections are made. With
    the default configuration, the overall pipeline speed will not decrease even if
    saving the full frames at a high frequency (> ~5).
3.  With the logger set up, you can write any information you need (e.g. for
    finding problems if something went wrong) to the log file. This is a good
    alternative to `print()`, if you don't have a terminal output. You can add
    your own custom logging command in any line with:

    ``` py
    logger.info("I want to write this to my log file.")
    ```

4.  If the PiJuice battery charge level or the free space left on the microSD
    card is below the specified threshold, no recording will be made and the
    Raspberry Pi is immediately shut down. You can specify your custom
    thresholds (e.g. when using a different battery capacity) above.
5.  You can specify your own recording durations and charge level thresholds in
    this code section. The suggested values can provide an efficient recording
    behaviour if you are using the 12,000 mAh PiJuice battery and set up the
    [Wakeup Alarm](pisetup.md#configure-pijuice-zero){target=_blank} for
    3-6 times per day. Depending on the number of Wakeups per day, as well as
    the season and sun exposure of the solar panel, it can make sense to
    increase or decrease the recording duration.
6.  Activate this option only if you have two batteries installed! Disable
    charging of the PiJuice battery if the charge level is higher than the
    specified threshold to extend battery life. The charge level is checked
    again at the end of the script to re-enable charging if the charge level
    dropped below a specified threshold.
7.  The recording will be stopped after the recording time is finished or if the
    charge level of the PiJuice battery drops below the specified threshold for
    ten times. This avoids immediate stopping of the recording if the
    battery charge level is falsely returned < 10, which can happen sometimes.
8.  A tracked object can have 4 possible statuses: `NEW`, `TRACKED`, `LOST` and
    `REMOVED`. It is highly recommended to save the cropped detections only when
    `tracking status == TRACKED`, but you could change this configuration here and
    e.g. write the `track.status.name` as additional column to the metadata .csv.
9.  This function will be called after a recording interval is finished, or if
    an error occurs during the recording and will write some info about the
    respective recording interval to `record_log.csv`.
10. If you are still in the testing phase, comment out the shutdown command in
    this line by adding `#` in front of the line. Otherwise your Raspberry Pi
    will automatically shut down everytime you run (and stop) the script.

---

## Frame capture

If you want to only capture images, e.g. for training data collection, you can
use the following script to save HQ frames (e.g. 1920x1080 px) to .jpg at a
specified time interval. Optionally, the downscaled LQ frames (e.g. 320x320 px)
can be saved to .jpg additionally, e.g. to include them in the training data,
as the detection model will use LQ frames for inference. However, it is recommended
to use the original HQ images for annotation and downscale them only before/during training.

Run the script with the Python interpreter from the virtual environment where
you installed the required packages (e.g. `env_insdet`):

``` bash
env_insdet/bin/python3 insect-detect/frame_capture.py
```

??? info "Optional arguments"

    Add after `env_insdet/bin/python3 insect-detect/frame_capture.py`, separated by space:

    - `-min` set recording time in minutes (default: 2)
    - `-4k` set camera resolution to 4K (3840x2160 px) (default: 1080p)
    - `-lq` additionally save downscaled LQ frames (e.g. 320x320 px)
    - `-af CM_MIN CM_MAX` set auto focus range in cm (min distance, max distance)
    - `-zip` store data in an uncompressed .zip file for each day and delete original directory

Stop the script by pressing ++ctrl+c++ in the Terminal.

``` py title="frame_capture.py" linenums="1" hl_lines="14-23 56 61"
#!/usr/bin/env python3

"""Save frames from OAK camera.

Source:   https://github.com/maxsitt/insect-detect
License:  GNU GPLv3 (https://choosealicense.com/licenses/gpl-3.0/)
Author:   Maximilian Sittinger (https://github.com/maxsitt)
Docs:     https://maxsitt.github.io/insect-detect-docs/

- save HQ frames (default: 1920x1080 px) to .jpg at the
  specified capture frequency (default: 1 s)
  -> stop recording early if free disk space drops below threshold
- optional arguments:
  '-min' set recording time in minutes (default: 2 [min])
         -> e.g. '-min 5' for 5 min recording time
  '-4k'  set camera resolution to 4K (3840x2160 px) (default: 1080p)
  '-lq'  additionally save downscaled LQ frames (e.g. 320x320 px)
  '-af'  set auto focus range in cm (min distance, max distance)
         -> e.g. '-af 14 20' to restrict auto focus range to 14-20 cm
  '-zip' store all captured data in an uncompressed .zip file for each day
         and delete original directory
         -> increases file transfer speed from microSD to computer
            but also on-device processing time and power consumption

based on open source scripts available at https://github.com/luxonis
"""

import argparse
import logging
import time
from datetime import datetime
from pathlib import Path

import cv2
import depthai as dai
import psutil

from utils.general import zip_data
from utils.oak_cam import set_focus_range

# Define optional arguments
parser = argparse.ArgumentParser()
parser.add_argument("-min", "--min_rec_time", type=int, choices=range(1, 721), default=2,
    help="Set recording time in minutes (default: 2 [min]).", metavar="1-720")
parser.add_argument("-4k", "--four_k_resolution", action="store_true",
    help="Set camera resolution to 4K (3840x2160 px) (default: 1080p).")
parser.add_argument("-lq", "--save_lq_frames", action="store_true",
    help="Additionally save downscaled LQ frames (320x320 px).")
parser.add_argument("-af", "--af_range", nargs=2, type=int,
    help="Set auto focus range in cm (min distance, max distance).", metavar=("CM_MIN", "CM_MAX"))
parser.add_argument("-zip", "--zip_data", action="store_true",
    help="Store data in an uncompressed .zip file for each day and delete original directory.")
args = parser.parse_args()

# Set threshold value required to start and continue a recording
MIN_DISKSPACE = 100  # minimum free disk space (MB) (default: 100 MB)

# Set capture frequency (default: 1 second)
# -> wait for specified amount of seconds between saving HQ frames
# 'CAPTURE_FREQ = 0.8' (0.2 for 4K) saves ~58 frames per minute to .jpg
CAPTURE_FREQ = 0.8 if not args.four_k_resolution else 0.2 # (1)!

# Set recording time (default: 2 minutes)
REC_TIME = args.min_rec_time * 60

# Set logging level and format
logging.basicConfig(level=logging.INFO, format="%(message)s")

# Create directory per day and recording interval to save HQ frames (+ LQ frames)
rec_start = datetime.now()
rec_start_format = rec_start.strftime("%Y-%m-%d_%H-%M-%S")
save_path = Path(f"insect-detect/frames/{rec_start.date()}/{rec_start_format}")
save_path.mkdir(parents=True, exist_ok=True)
if args.save_lq_frames:
    (save_path / "LQ_frames").mkdir(parents=True, exist_ok=True)

# Create depthai pipeline
pipeline = dai.Pipeline()

# Create and configure color camera node and define output(s)
cam_rgb = pipeline.create(dai.node.ColorCamera)
#cam_rgb.setImageOrientation(dai.CameraImageOrientation.ROTATE_180_DEG)  # rotate image 180°
cam_rgb.setResolution(dai.ColorCameraProperties.SensorResolution.THE_4_K)
if not args.four_k_resolution:
    cam_rgb.setIspScale(1, 2)  # downscale 4K to 1080p resolution -> HQ frames
cam_rgb.setInterleaved(False)  # planar layout
cam_rgb.setColorOrder(dai.ColorCameraProperties.ColorOrder.BGR)
cam_rgb.setFps(25)  # frames per second available for auto focus/exposure
if args.save_lq_frames:
    cam_rgb.setPreviewSize(320, 320)  # downscale frames -> LQ frames
    cam_rgb.setPreviewKeepAspectRatio(False)  # stretch frames (16:9) to square (1:1)

xout_rgb = pipeline.create(dai.node.XLinkOut)
xout_rgb.setStreamName("frame")
cam_rgb.video.link(xout_rgb.input)  # HQ frames

if args.save_lq_frames:
    xout_lq = pipeline.create(dai.node.XLinkOut)
    xout_lq.setStreamName("frame_lq")
    cam_rgb.preview.link(xout_lq.input)  # LQ frames

if args.af_range:
    # Create XLinkIn node to send control commands to color camera node
    xin_ctrl = pipeline.create(dai.node.XLinkIn)
    xin_ctrl.setStreamName("control")
    xin_ctrl.out.link(cam_rgb.inputControl)

# Connect to OAK device and start pipeline in USB2 mode
with dai.Device(pipeline, maxUsbSpeed=dai.UsbSpeed.HIGH) as device:

    logging.info("Recording time: %s min\n", int(REC_TIME / 60))

    # Get free disk space (MB)
    disk_free = round(psutil.disk_usage("/").free / 1048576)

    # Create output queue(s) to get the frames from the output(s) defined above
    q_frame = device.getOutputQueue(name="frame", maxSize=4, blocking=False)
    if args.save_lq_frames:
        q_frame_lq = device.getOutputQueue(name="frame_lq", maxSize=4, blocking=False)

    if args.af_range:
        # Create input queue to send control commands to OAK camera
        q_ctrl = device.getInputQueue(name="control", maxSize=16, blocking=False)

        # Set auto focus range to specified cm values
        af_ctrl = set_focus_range(args.af_range[0], args.af_range[1])
        q_ctrl.send(af_ctrl)

    # Set start time of recording
    start_time = time.monotonic()

    # Record until recording time is finished
    # Stop recording early if free disk space drops below threshold
    while time.monotonic() < start_time + REC_TIME and disk_free > MIN_DISKSPACE:

        # Update free disk space (MB)
        disk_free = round(psutil.disk_usage("/").free / 1048576)

        # Get HQ (+ LQ) frames and save to .jpg
        if q_frame.has():
            frame_hq = q_frame.get().getCvFrame()
            timestamp_frame = datetime.now().strftime("%Y-%m-%d_%H-%M-%S-%f")
            path_hq = f"{save_path}/{timestamp_frame}.jpg"
            cv2.imwrite(path_hq, frame_hq)

        if args.save_lq_frames:
            if q_frame_lq.has():
                frame_lq = q_frame_lq.get().getCvFrame()
                path_lq = f"{save_path}/LQ_frames/{timestamp_frame}_LQ.jpg"
                cv2.imwrite(path_lq, frame_lq)

        # Wait for specified amount of seconds (default: 0.8 for 1080p; 0.2 for 4K)
        time.sleep(CAPTURE_FREQ)

# Print number and directory of saved frames
num_frames_hq = len(list(save_path.glob("*.jpg")))
if not args.save_lq_frames:
    logging.info("Saved %s HQ frames to %s\n", num_frames_hq, save_path)
else:
    num_frames_lq = len(list((save_path / "LQ_frames").glob("*.jpg")))
    logging.info("Saved %s HQ and %s LQ frames to %s\n", num_frames_hq, num_frames_lq, save_path)

if args.zip_data:
    # Store frames in uncompressed .zip file and delete original folder
    zip_data(save_path)
    logging.info("Stored all captured images in %s.zip\n", save_path.parent)

```

1.  You can increase the capture frequency in this line, e.g. if you want to
    only save a frame every 10 seconds, every minute etc. Keep in mind that
    the image processing will take some time and test different values until
    the frames are saved with your desired time interval.

---

## Video capture

With the following Python script you can save encoded HQ frames (1080p or 4K
resolution) with H.265 (HEVC) compression to a .mp4 video file. As there is no
encoding happening on the host (RPi), CPU and RAM usage is minimal, which makes
it possible to record 4K 30 fps video with almost no load on the Raspberry Pi.
As 4K 30 fps video can take up a lot of disk space, the remaining free disk
space is checked while recording and the recording is stopped if the free space
left drops below a specified threshold (e.g. 100 MB).

If you don't need the 25 fps you can decrease the frame rate which will
lead to a smaller video file size (e.g. `-fps 20`).

Run the script with the Python interpreter from the virtual environment where
you installed the required packages (e.g. `env_insdet`):

``` bash
env_insdet/bin/python3 insect-detect/video_capture.py
```

??? info "Optional arguments"

    Add after `env_insdet/bin/python3 insect-detect/video_capture.py`, separated by space:

    - `-min` set recording time in minutes (default: 2)
    - `-4k` set camera resolution to 4K (3840x2160 px) (default: 1080p)
    - `-fps` set camera frame rate (default: 25 fps)
    - `-af CM_MIN CM_MAX` set auto focus range in cm (min distance, max distance)

Stop the script by pressing ++ctrl+c++ in the Terminal.

``` py title="video_capture.py" linenums="1" hl_lines="12-18 49"
#!/usr/bin/env python3

"""Save video from OAK camera.

Source:   https://github.com/maxsitt/insect-detect
License:  GNU GPLv3 (https://choosealicense.com/licenses/gpl-3.0/)
Author:   Maximilian Sittinger (https://github.com/maxsitt)
Docs:     https://maxsitt.github.io/insect-detect-docs/

- save encoded HQ frames (1080p or 4K resolution) with H.265 (HEVC) compression to .mp4 video file
- optional arguments:
  '-min' set recording time in minutes (default: 2 [min])
         -> e.g. '-min 5' for 5 min recording time
  '-4k'  record video in 4K resolution (3840x2160 px) (default: 1080p)
  '-fps' set camera frame rate (default: 25 fps)
         -> e.g. '-fps 20' for 20 fps (less fps = smaller video file size)
  '-af'  set auto focus range in cm (min distance, max distance)
         -> e.g. '-af 14 20' to restrict auto focus range to 14-20 cm

based on open source scripts available at https://github.com/luxonis
"""

import argparse
import logging
import time
from datetime import datetime
from fractions import Fraction
from pathlib import Path

import av
import depthai as dai
import psutil

from utils.oak_cam import set_focus_range

# Define optional arguments
parser = argparse.ArgumentParser()
parser.add_argument("-min", "--min_rec_time", type=int, choices=range(1, 61), default=2,
    help="Set recording time in minutes (default: 2 [min]).", metavar="1-60")
parser.add_argument("-4k", "--four_k_resolution", action="store_true",
    help="Set camera resolution to 4K (3840x2160 px) (default: 1080p).")
parser.add_argument("-fps", "--frames_per_second", type=int, choices=range(1, 31), default=25,
    help="Set camera frame rate (default: 25 fps).", metavar="1-30")
parser.add_argument("-af", "--af_range", nargs=2, type=int,
    help="Set auto focus range in cm (min distance, max distance).", metavar=("CM_MIN", "CM_MAX"))
args = parser.parse_args()

# Set threshold value required to start and continue a recording
MIN_DISKSPACE = 100  # minimum free disk space (MB) (default: 100 MB)

# Set recording time (default: 2 minutes)
REC_TIME = args.min_rec_time * 60

# Set video resolution
RES = "1080p" if not args.four_k_resolution else "4K"

# Set frame rate (default: 25 fps)
FPS = args.frames_per_second

# Set logging level and format
logging.basicConfig(level=logging.INFO, format="%(message)s")

# Create directory per day to save video
rec_start = datetime.now()
save_path = Path(f"insect-detect/videos/{rec_start.date()}")
save_path.mkdir(parents=True, exist_ok=True)

# Create depthai pipeline
pipeline = dai.Pipeline()

# Create and configure color camera node
cam_rgb = pipeline.create(dai.node.ColorCamera)
#cam_rgb.setImageOrientation(dai.CameraImageOrientation.ROTATE_180_DEG)  # rotate image 180°
cam_rgb.setResolution(dai.ColorCameraProperties.SensorResolution.THE_4_K)
if not args.four_k_resolution:
    cam_rgb.setIspScale(1, 2)  # downscale 4K to 1080p resolution -> HQ frames
cam_rgb.setInterleaved(False)  # planar layout
cam_rgb.setFps(FPS)  # frames per second available for auto focus/exposure

# Create and configure video encoder node and define input + output
video_enc = pipeline.create(dai.node.VideoEncoder) # (1)!
video_enc.setDefaultProfilePreset(FPS, dai.VideoEncoderProperties.Profile.H265_MAIN)
cam_rgb.video.link(video_enc.input)

xout_vid = pipeline.create(dai.node.XLinkOut)
xout_vid.setStreamName("video")
video_enc.bitstream.link(xout_vid.input)  # encoded HQ frames

if args.af_range:
    # Create XLinkIn node to send control commands to color camera node
    xin_ctrl = pipeline.create(dai.node.XLinkIn)
    xin_ctrl.setStreamName("control")
    xin_ctrl.out.link(cam_rgb.inputControl)

# Connect to OAK device and start pipeline in USB2 mode
with dai.Device(pipeline, maxUsbSpeed=dai.UsbSpeed.HIGH) as device:

    logging.info("Recording time: %s min\n", int(REC_TIME / 60))

    # Get free disk space (MB)
    disk_free = round(psutil.disk_usage("/").free / 1048576)

    # Create output queue to get the encoded frames from the output defined above
    q_video = device.getOutputQueue(name="video", maxSize=30, blocking=True)

    if args.af_range:
        # Create input queue to send control commands to OAK camera
        q_ctrl = device.getInputQueue(name="control", maxSize=16, blocking=False)

        # Set auto focus range to specified cm values
        af_ctrl = set_focus_range(args.af_range[0], args.af_range[1])
        q_ctrl.send(af_ctrl)

    # Create .mp4 container with H.265 (HEVC) compression
    timestamp_video = datetime.now().strftime("%Y-%m-%d_%H-%M-%S-%f")
    with av.open(f"{save_path}/{timestamp_video}_{FPS}fps_{RES}_video.mp4", "w") as container:
        stream = container.add_stream("hevc", rate=FPS, options={"x265-params": "log_level=none"})
        stream.width, stream.height = cam_rgb.getVideoSize()
        stream.time_base = Fraction(1, 1000 * 1000)

    # Set start time of recording
    start_time = time.monotonic()

    # Record until recording time is finished
    # Stop recording early if free disk space drops below threshold
    while time.monotonic() < start_time + REC_TIME and disk_free > MIN_DISKSPACE:

        # Update free disk space (MB)
        disk_free = round(psutil.disk_usage("/").free / 1048576)

        # Get encoded video frames and save to packet
        if q_video.has():
            enc_video = q_video.get().getData()
            packet = av.Packet(enc_video)
            packet_timestamp = int((time.monotonic() - start_time) * 1000 * 1000)
            packet.dts = packet_timestamp
            packet.pts = packet_timestamp

            # Mux packet into .mp4 container
            container.mux_one(packet)

# Print duration, resolution, fps and directory of saved video + free disk space
logging.info("Saved %s min %s video with %s fps to %s\n", int(REC_TIME / 60), RES, FPS, save_path)
logging.info("Free disk space left: %s MB\n", disk_free)

```

1.  More info about the
    [VideoEncoder node](https://docs.luxonis.com/projects/api/en/latest/components/nodes/video_encoder/){target=_blank}.
