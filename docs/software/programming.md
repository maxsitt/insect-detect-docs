# Programming

!!! tip "Adapt the software to your use case"

    You will find all Python scripts to get started with the development of
    automated monitoring pipelines with the Raspberry Pi + OAK-1 in this
    section, together with details on possible modifications. Click on the
    :material-plus-circle: symbol to open the code annotations for more
    info. You can find the full
    [DepthAI Python API reference](https://docs.luxonis.com/projects/api/en/latest/references/python/)
    at the DepthAI Docs.

The latest versions of the Python scripts are available in the
[`insect-detect` GitHub repo](https://github.com/maxsitt/insect-detect).
Download the whole repository and copy it to the `home/pi` folder of your
Raspberry Pi, with simple Drag and drop in the VS Code remote window explorer.
If you run into any problems, find a bug or something that could be optimized,
please post an [issue](https://github.com/maxsitt/insect-detect/issues) at the
GitHub repo.

---

## OAK camera preview

This Python script will create and configure the
[ColorCamera node](https://docs.luxonis.com/projects/api/en/latest/components/nodes/color_camera/),
to send frames to the host (Raspberry Pi) and show them in a new window. If you
are connected to the Raspberry Pi via SSH,
[X11 forwarding](pisetup.md#configure-x11-forwarding) has to be set up together
with an active [X server](localsetup.md#vcxsrv-windows-x-server), to show the
frames in a window on your local PC. In the proposed script, the sensor
resolution is set to 4K (3840x2160 px) and the full FOV 4K frames are
downscaled to LQ preview frames (416x416 px), which is the same configuration
as used for the full
[automated monitoring pipeline](../deployment/detection.md).

Run the script with:

``` bash
python3 ./insect-detect/cam_preview.py
```

``` py title="cam_preview.py" hl_lines="17 18 20"
'''
Author:   Maximilian Sittinger (https://github.com/maxsitt)
License:  GNU GPLv3 (https://choosealicense.com/licenses/gpl-3.0/)

compiled with open source scripts available at https://github.com/luxonis
'''

import cv2
import depthai as dai

# Create depthai pipeline
pipeline = dai.Pipeline()

# Define camera source and output
cam_rgb = pipeline.create(dai.node.ColorCamera) # (1)!
#cam_rgb.setImageOrientation(dai.CameraImageOrientation.ROTATE_180_DEG) # (2)
cam_rgb.setResolution(dai.ColorCameraProperties.SensorResolution.THE_4_K) # (3)!
cam_rgb.setPreviewSize(416, 416) # downscaled LQ frames
cam_rgb.setInterleaved(False)
cam_rgb.setPreviewKeepAspectRatio(False) # squash full FOV frames to square # (4)
cam_rgb.setFps(20) # frames per second available for auto-focus/-exposure

xout_rgb = pipeline.create(dai.node.XLinkOut) # (5)!
xout_rgb.setStreamName("frame")
cam_rgb.preview.link(xout_rgb.input)

# Connect to OAK device and start pipeline
with dai.Device(pipeline, usb2Mode=True) as device: # (6)!

    # Create output queue to get the frames from the output defined above
    q_frame = device.getOutputQueue(name="frame", maxSize=4, blocking=False) # (7)!

    # Get LQ preview frames and show in window (e.g. via X11 forwarding)
    while True:
        frame = q_frame.get().getCvFrame()
        cv2.imshow("cam_preview", frame)

        if cv2.waitKey(1) == ord("q"): # (8)!
            break

```

1.  You can find more info about the
    [ColorCamera node](https://docs.luxonis.com/projects/api/en/latest/components/nodes/color_camera/)
    and possible configurations at the DepthAI Docs.
2.  If your image is shown upside down (on older OAK-1 cameras), you can rotate
    it on-device by activating this configuration.
3.  `THE_4_K` = 3840x2160 pixel. Default sensor resolution is `THE_1080_P`.
    Aspect ratio for both is 16:9. You can check all
    [supported sensors](https://docs.luxonis.com/projects/hardware/en/latest/pages/articles/supported_sensors.html)
    and their respective resolutions at the DepthAI Docs. IMX378 is used in the OAK-1.
4.  More info about other possible
    [downscaling options](https://github.com/luxonis/depthai-experiments/tree/master/gen2-full-fov-nn).
5.  The [XLinkOut node](https://docs.luxonis.com/projects/api/en/latest/components/nodes/xlink_out/)
    sends data from the OAK device to the host (e.g. Raspberry Pi) via XLink.
6.  If your host (e.g. RPi Zero 2 W) has no USB 3 port or you aren't using a
    USB 3 cable, it is recommended to
    [force USB 2 communication](https://docs.luxonis.com/en/latest/pages/troubleshooting/#forcing-usb2-communication)
    with `usb2Mode=True`.
7.  You can
    [specify different queue configurations](https://docs.luxonis.com/projects/api/en/latest/components/device/#specifying-arguments-for-getoutputqueue-method),
    by changing the maximum queue size or the blocking behaviour.
8.  Press ++q++ on your keyboard while the window is selected to close it and
    stop the script execution.

---

## YOLOv5 preview

With the following Python script you can run a custom YOLOv5 object detection
model ([.blob format](https://docs.luxonis.com/en/latest/pages/model_conversion))
on the OAK device with full FOV 4K frames downscaled to LQ frames (e.g. 416x416
px) as model input and show the frames together with the model output (bounding
box, label, confidence score) in a new window.

If you copied the whole [`insect-detect` GitHub repo](githublink) to your
Raspberry Pi, the provided YOLOv5s detection model and config .json will be
used by the OAK-1 device. If you want to use your own model, change the
`MODEL_PATH` and `CONFIG_PATH` accordingly.

Run the script with:

``` bash
python3 ./insect-detect/yolov5_preview.py
```

??? info "Optional arguments"

    Add after `yolov5_preview.py`, separated by space:

    - `-log` to print available Rasperry Pi memory (MB) and RPi CPU utilization
      (%) to console

``` py title="yolov5_preview.py" hl_lines="28 29"
'''
Author:   Maximilian Sittinger (https://github.com/maxsitt)
License:  GNU GPLv3 (https://choosealicense.com/licenses/gpl-3.0/)

compiled with open source scripts available at https://github.com/luxonis
'''

import argparse
import json
import sys
import time
from pathlib import Path

import cv2
import depthai as dai
import numpy as np

# Define optional arguments
parser = argparse.ArgumentParser()
parser.add_argument("-log", "--print-log", action="store_true",
    help="print RPi available memory (MB) + CPU utilization (percent)")
args = parser.parse_args()

if args.print_log:
    import psutil

# Set file paths to the detection model and config JSON
MODEL_PATH = Path("./insect-detect/models/yolov5s_416_openvino_2022.1_9shave.blob") # (1)!
CONFIG_PATH = Path("./insect-detect/models/json/yolov5s_416.json") # (2)!

# Extract detection model metadata from config JSON
with CONFIG_PATH.open(encoding="utf-8") as f:
    config = json.load(f)
nn_config = config.get("nn_config", {})
nn_metadata = nn_config.get("NN_specific_metadata", {})
classes = nn_metadata.get("classes", {})
coordinates = nn_metadata.get("coordinates", {})
anchors = nn_metadata.get("anchors", {})
anchor_masks = nn_metadata.get("anchor_masks", {})
iou_threshold = nn_metadata.get("iou_threshold", {}) # (3)!
confidence_threshold = nn_metadata.get("confidence_threshold", {})
nn_mappings = config.get("mappings", {})
labels = nn_mappings.get("labels", {})

# Create depthai pipeline
pipeline = dai.Pipeline()

# Define camera source
cam_rgb = pipeline.create(dai.node.ColorCamera)
#cam_rgb.setImageOrientation(dai.CameraImageOrientation.ROTATE_180_DEG)
cam_rgb.setResolution(dai.ColorCameraProperties.SensorResolution.THE_4_K)
cam_rgb.setPreviewSize(416, 416) # downscaled LQ frames for model input
cam_rgb.setInterleaved(False)
cam_rgb.setPreviewKeepAspectRatio(False) # squash full FOV frames to square
cam_rgb.setFps(20) # frames per second available for focus/exposure/model input

# Define detection model source and input + output
nn = pipeline.create(dai.node.YoloDetectionNetwork) # (4)!
cam_rgb.preview.link(nn.input) # downscaled LQ frames as model input # (5)
nn.input.setBlocking(False) # (6)!

xout_nn = pipeline.create(dai.node.XLinkOut)
xout_nn.setStreamName("nn")
nn.out.link(xout_nn.input)

xout_rgb = pipeline.create(dai.node.XLinkOut)
xout_rgb.setStreamName("frame")
nn.passthrough.link(xout_rgb.input)

# Set detection model specific settings
nn.setBlobPath(MODEL_PATH)
nn.setNumClasses(classes)
nn.setCoordinateSize(coordinates)
nn.setAnchors(anchors)
nn.setAnchorMasks(anchor_masks)
nn.setIouThreshold(iou_threshold)
nn.setConfidenceThreshold(confidence_threshold)
nn.setNumInferenceThreads(2)

# Define function to convert relative bounding box coordinates (0-1) to pixel coordinates
def frame_norm(frame, bbox):
    """Convert relative bounding box coordinates (0-1) to pixel coordinates."""
    norm_vals = np.full(len(bbox), frame.shape[0])
    norm_vals[::2] = frame.shape[1]
    return (np.clip(np.array(bbox), 0, 1) * norm_vals).astype(int)

# Connect to OAK device and start pipeline
with dai.Device(pipeline, usb2Mode=True) as device:

    # Create output queues to get the frames and detections from the outputs defined above
    q_frame = device.getOutputQueue(name="frame", maxSize=4, blocking=False)
    q_nn = device.getOutputQueue(name="nn", maxSize=4, blocking=False)

    # Create start_time and counter variables to measure fps of the detection model
    start_time = time.monotonic()
    counter = 0

    # Get LQ preview frames and model output (detections) and show in window
    while True:
        if args.print_log:
            print(f"Available RPi memory: {round(psutil.virtual_memory().available / 1048576)} MB")
            print(f"RPi CPU utilization:  {psutil.cpu_percent(interval=None)}%")
            print("\n")

        frame = q_frame.get().getCvFrame()
        nn_out = q_nn.get()

        if nn_out is not None:
            dets = nn_out.detections
            counter += 1

        if frame is not None:
            for detection in dets:
                bbox = frame_norm(frame, (detection.xmin, detection.ymin,
                                          detection.xmax, detection.ymax))
                cv2.putText(frame, labels[detection.label], (bbox[0], bbox[3] + 20),
                            cv2.FONT_HERSHEY_SIMPLEX, 0.5, (255, 255, 255), 1) # (7)!
                cv2.putText(frame, f"{round(detection.confidence, 2)}", (bbox[0], bbox[3] + 40),
                            cv2.FONT_HERSHEY_SIMPLEX, 0.6, (255, 255, 255), 1)
                cv2.rectangle(frame, (bbox[0], bbox[1]), (bbox[2], bbox[3]), (0, 0, 255), 2)

            cv2.putText(frame, "NN fps: {:.2f}".format(counter / (time.monotonic() - start_time)),
                        (2, frame.shape[0] - 4), cv2.FONT_HERSHEY_SIMPLEX, 0.7, (0, 255, 255), 2)

            cv2.imshow("yolov5_preview", frame)

        if cv2.waitKey(1) == ord("q"):
            break

```

1.  Specify the path to your detection model (in .blob format) that the OAK is
    able to use it for on-device inference.
2.  Specify the path to your config .json file that you created after training
    and converting the model.
3.  All metadata that is necessary to successfully run the model on-device is
    extracted from the corresponding config .json file. However, you could also
    change your IoU or confidence threshold in this line for experimenting with
    different settings.
4.  More info about the
    [YoloDetectionNetwork node](https://docs.luxonis.com/projects/api/en/latest/components/nodes/yolo_detection_network/).
5.  The downscaled LQ preview frames are used as model input. More info about
    [linking nodes](https://docs.luxonis.com/projects/api/en/latest/components/nodes/).
6.  To avoid freezing of the pipeline, we will set `blocking=False` for the
    frames that are used as model input.
7.  More info on
    [`cv2.putText()`](https://www.geeksforgeeks.org/python-opencv-cv2-puttext-method/)
    to customize your output.

---

## YOLOv5 + object tracker preview

In the next Python script we will add an
[ObjectTracker node](https://docs.luxonis.com/projects/api/en/latest/components/nodes/object_tracker/),
based on the [Intel DL Streamer](https://dlstreamer.github.io/dev_guide/object_tracking.html)
framework, which uses the detection model output as input for tracking detected
objects and assigning unique tracking IDs on-device. The frames, together with
the model and tracker output (bounding box from tracker output and bbox from
detection model, label, confidence score, tracking ID, tracking status), are
shown in a new window.

Run the script with:

``` bash
python3 ./insect-detect/yolov5_tracker_preview.py
```

??? info "Optional arguments"

    Add after `yolov5_tracker_preview.py`, separated by space:

    - `-log` to print available Rasperry Pi memory (MB) and RPi CPU utilization
      (%) to console

``` py title="yolov5_tracker_preview.py" hl_lines="28 29 74 140 141"
'''
Author:   Maximilian Sittinger (https://github.com/maxsitt)
License:  GNU GPLv3 (https://choosealicense.com/licenses/gpl-3.0/)

compiled with open source scripts available at https://github.com/luxonis
'''

import argparse
import json
import sys
import time
from pathlib import Path

import cv2
import depthai as dai
import numpy as np

# Define optional arguments
parser = argparse.ArgumentParser()
parser.add_argument("-log", "--print-log", action="store_true",
    help="print RPi available memory (MB) + CPU utilization (percent)")
args = parser.parse_args()

if args.print_log:
    import psutil

# Set file paths to the detection model and config JSON
MODEL_PATH = Path("./insect-detect/models/yolov5s_416_openvino_2022.1_9shave.blob")
CONFIG_PATH = Path("./insect-detect/models/json/yolov5s_416.json")

# Extract detection model metadata from config JSON
with CONFIG_PATH.open(encoding="utf-8") as f:
    config = json.load(f)
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

# Define camera source
cam_rgb = pipeline.create(dai.node.ColorCamera)
#cam_rgb.setImageOrientation(dai.CameraImageOrientation.ROTATE_180_DEG)
cam_rgb.setResolution(dai.ColorCameraProperties.SensorResolution.THE_4_K)
cam_rgb.setPreviewSize(416, 416) # downscaled LQ frames for model input
cam_rgb.setInterleaved(False)
cam_rgb.setPreviewKeepAspectRatio(False) # squash full FOV frames to square
cam_rgb.setFps(20) # frames per second available for focus/exposure/model input

# Define detection model source and input
nn = pipeline.create(dai.node.YoloDetectionNetwork)
cam_rgb.preview.link(nn.input) # downscaled LQ frames as model input
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

# Define object tracker source and input + output
tracker = pipeline.create(dai.node.ObjectTracker) # (1)!
tracker.setTrackerType(dai.TrackerType.SHORT_TERM_IMAGELESS) # (2)!
#tracker.setTrackerType(dai.TrackerType.ZERO_TERM_IMAGELESS)
#tracker.setTrackerType(dai.TrackerType.ZERO_TERM_COLOR_HISTOGRAM)
tracker.setTrackerIdAssignmentPolicy(dai.TrackerIdAssignmentPolicy.UNIQUE_ID)
nn.passthrough.link(tracker.inputTrackerFrame)
nn.passthrough.link(tracker.inputDetectionFrame)
nn.out.link(tracker.inputDetections)

xout_tracker = pipeline.create(dai.node.XLinkOut)
xout_tracker.setStreamName("track")
tracker.out.link(xout_tracker.input)

xout_rgb = pipeline.create(dai.node.XLinkOut)
xout_rgb.setStreamName("frame")
tracker.passthroughTrackerFrame.link(xout_rgb.input)

# Define function to convert relative bounding box coordinates (0-1) to pixel coordinates
def frame_norm(frame, bbox):
    """Convert relative bounding box coordinates (0-1) to pixel coordinates."""
    norm_vals = np.full(len(bbox), frame.shape[0])
    norm_vals[::2] = frame.shape[1]
    return (np.clip(np.array(bbox), 0, 1) * norm_vals).astype(int)

# Connect to OAK device and start pipeline
with dai.Device(pipeline, usb2Mode=True) as device:

    # Create output queues to get the frames and detections from the outputs defined above
    q_frame = device.getOutputQueue(name="frame", maxSize=4, blocking=False)
    q_track = device.getOutputQueue(name="track", maxSize=4, blocking=False)

    # Create start_time and counter variables to measure fps of the detection model + tracker
    start_time = time.monotonic()
    counter = 0

    # Get LQ preview frames and tracker output (detections + t.ID) and show in window
    while True:
        if args.print_log:
            print(f"Available RPi memory: {round(psutil.virtual_memory().available / 1048576)} MB")
            print(f"RPi CPU utilization:  {psutil.cpu_percent(interval=None)}%")
            print("\n")

        frame = q_frame.get().getCvFrame()
        track_out = q_track.get()

        if track_out is not None:
            tracklets_data = track_out.tracklets
            counter+=1
            
        if frame is not None:
            for t in tracklets_data:
                roi = t.roi.denormalize(frame.shape[1], frame.shape[0]) # (3)!
                x1 = int(roi.topLeft().x)
                y1 = int(roi.topLeft().y)
                x2 = int(roi.bottomRight().x)
                y2 = int(roi.bottomRight().y)

                bbox = frame_norm(frame, (t.srcImgDetection.xmin, t.srcImgDetection.ymin,
                                          t.srcImgDetection.xmax, t.srcImgDetection.ymax)) # (4)!
                cv2.putText(frame, labels[t.srcImgDetection.label], (bbox[0], bbox[3] + 20),
                            cv2.FONT_HERSHEY_SIMPLEX, 0.5, (255, 255, 255), 1)
                cv2.putText(frame, f"{round(t.srcImgDetection.confidence, 2)}", (bbox[0], bbox[3] + 40),
                            cv2.FONT_HERSHEY_SIMPLEX, 0.6, (255, 255, 255), 1)
                cv2.putText(frame, f"t.ID:{t.id}", (bbox[0], bbox[3] + 60),
                            cv2.FONT_HERSHEY_SIMPLEX,  0.6, (255, 255, 255), 1)
                cv2.putText(frame, t.status.name, (bbox[0], bbox[3] + 75),
                            cv2.FONT_HERSHEY_SIMPLEX,  0.4, (255, 255, 255), 1)
                cv2.rectangle(frame, (bbox[0], bbox[1]), (bbox[2], bbox[3]), (0, 0, 255), 2) # model output
                cv2.rectangle(frame, (x1, y1), (x2, y2), (0, 255, 130), 1) # tracker output

            cv2.putText(frame, "NN fps: {:.2f}".format(counter / (time.monotonic() - start_time)),
                        (2, frame.shape[0] - 4), cv2.FONT_HERSHEY_SIMPLEX, 0.7, (0, 255, 255), 2)

            cv2.imshow("tracker_preview", frame)

        if cv2.waitKey(1) == ord("q"):
            break

```

1.  More info about the
    [ObjectTracker node](https://docs.luxonis.com/projects/api/en/latest/components/nodes/object_tracker/).
2.  More info on the supported
    [tracking types](https://dlstreamer.github.io/dev_guide/object_tracking.html).
3.  You can use the bounding box coordinates from the tracker output, as
    defined in this segment, and/or the bbox coordinates from the passthrough
    detections to draw the bboxes on the frame. In this example we are using
    both, try it out and check if you notice some differences in the outputs!
4.  The bounding boxes from the passthrough detections might be more stable
    than from the object tracker output, you can decide for yourself which one
    is best for your use case.

---

## Automated monitoring script

The following Python script is the main script for fully
[automated insect monitoring](../deployment/detection.md).

- The object tracker output (+ passthrough detections) from inference on LQ
  frames (e.g. 416x416) is synchronized with the HQ frames (e.g. 3840x2160) in a
  [script node](https://docs.luxonis.com/projects/api/en/latest/components/nodes/script/)
  on-device, using the respective sequence numbers.
- Detections (area of the bounding box) are cropped from the synced HQ frames
  and saved to .jpg. See the optional arguments to save the full raw HQ frames
  additionally (e.g. for training data collection).
- All relevant metadata from the detection model and tracker output (timestamp,
  label, confidence score, tracking ID, relative bbox coordinates, .jpg file
  path) is saved to a metadata .csv file for each cropped detection.
- Info and error (stderr) + traceback messages are written to log file and
  recording info (recording ID, start/end time, duration, number of cropped
  detections, number of unique tracking IDs, free disk space and battery charge
  level) is written to a `record_log.csv` file for each recording interval.
- After a recording interval is finished, or if the PiJuice battery charge
  level drops below a specified threshold, or if an error occurs, the Raspberry
  Pi is safely shut down and waits for the next wake up from the PiJuice Zero.
- The [PiJuice I2C Command API](https://github.com/PiSupply/PiJuice/tree/master/Software#i2c-command-api)
  is used for power management, which means that a recording will only be made
  if the PiJuice battery charge level is higher than a specified threshold and
  the respective recording duration is conditional on the current charge level.

??? question "No PiJuice Zero?"

    If you want to try the script without the PiJuice Zero pHAT connected to
    your Raspberry Pi, use the `yolov5_tracker_save_hqsync_nopj.py` script,
    available at the [`insect-detect` GitHub repo](https://github.com/maxsitt/insect-detect).

Run the script with:

``` bash
python3 ./insect-detect/yolov5_tracker_save_hqsync.py
```

For fully automated monitoring in the field, set up a
[cron job](../pisetup/#set-up-cron-job) that will run the script automatically
at each boot (after wake up by PiJuice Zero pHAT).

??? info "Optional arguments"

    Add after `yolov5_tracker_save_hqsync.py`, separated by space:

    - `-log` to save additional battery, temperature and RPi memory/CPU logs to .csv
    - `-raw` to save cropped detections + full raw HQ frames (e.g. for training data)
    - `-overlay` to save cropped detections + full HQ frames with overlay (bbox + info)

``` py title="yolov5_tracker_save_hqsync.py" hl_lines="39 40 48 191 306 307 320 332 334 343"
'''
Author:   Maximilian Sittinger (https://github.com/maxsitt)
License:  GNU GPLv3 (https://choosealicense.com/licenses/gpl-3.0/)

includes segments from open source scripts available at https://github.com/luxonis
'''

import argparse
import csv
import json
import logging
import subprocess
import sys
import time
import traceback
from datetime import datetime
from pathlib import Path

import cv2
import depthai as dai
import numpy as np
import pandas as pd
import psutil
from gpiozero import CPUTemperature
from pijuice import PiJuice

# Create data folder if not already present
Path("./insect-detect/data").mkdir(parents=True, exist_ok=True)

# Create logger and send info + error messages to log file
logging.basicConfig(filename = "./insect-detect/data/script_log.log",
                    encoding = "utf-8",
                    format = "%(asctime)s - %(levelname)s: %(message)s",
                    level = logging.DEBUG)
logger = logging.getLogger() # (1)!
sys.stderr.write = logger.error

# Set file paths to the detection model and config JSON
MODEL_PATH = Path("./insect-detect/models/yolov5s_416_openvino_2022.1_9shave.blob")
CONFIG_PATH = Path("./insect-detect/models/json/yolov5s_416.json")

# Instantiate PiJuice
pijuice = PiJuice(1, 0x14)

# Continue script only if PiJuice battery charge level and free disk space are higher than thresholds
chargelevel_start = pijuice.status.GetChargeLevel().get("data", -1)
disk_free = round(psutil.disk_usage("/").free / 1048576) # free disk space in MB
if chargelevel_start < 10 or disk_free < 200: # (2)!
    logger.info(f"Shut down without recording | Charge level: {chargelevel_start}%\n")
    subprocess.run(["sudo", "shutdown", "-h", "now"], check=True)
    time.sleep(5) # wait 5 seconds for RPi to shut down

# Define optional arguments
parser = argparse.ArgumentParser()
group = parser.add_mutually_exclusive_group()
group.add_argument("-raw", "--save-raw-frames", action="store_true",
    help="additionally save full raw HQ frames in separate folder (e.g. for training data)")
group.add_argument("-overlay", "--save-overlay-frames", action="store_true",
    help="additionally save full HQ frames with overlay (bbox + info) in separate folder")
parser.add_argument("-log", "--save-logs", action="store_true",
    help="save battery info + temperature, RPi CPU + OAK VPU temperatures and \
          RPi available memory (MB) + CPU utilization (percent) to .csv file")
args = parser.parse_args()

# Extract detection model metadata from config JSON
with CONFIG_PATH.open(encoding="utf-8") as f:
    config = json.load(f)
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

# Define camera source
cam_rgb = pipeline.create(dai.node.ColorCamera)
#cam_rgb.setImageOrientation(dai.CameraImageOrientation.ROTATE_180_DEG)
cam_rgb.setResolution(dai.ColorCameraProperties.SensorResolution.THE_4_K)
cam_rgb.setPreviewSize(416, 416) # downscaled LQ frames for model input
cam_rgb.setInterleaved(False)
cam_rgb.setPreviewKeepAspectRatio(False) # squash full FOV frames to square
cam_rgb.setVideoSize(3840, 2160) # HQ frames for syncing, aspect ratio 16:9 (4K)
cam_rgb.setFps(20) # frames per second available for focus/exposure/model input

# Define detection model source and input
nn = pipeline.create(dai.node.YoloDetectionNetwork)
cam_rgb.preview.link(nn.input) # downscaled LQ frames as model input
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

# Define object tracker source and input + output
tracker = pipeline.create(dai.node.ObjectTracker)
tracker.setTrackerType(dai.TrackerType.SHORT_TERM_IMAGELESS)
#tracker.setTrackerType(dai.TrackerType.ZERO_TERM_IMAGELESS)
#tracker.setTrackerType(dai.TrackerType.ZERO_TERM_COLOR_HISTOGRAM)
tracker.setTrackerIdAssignmentPolicy(dai.TrackerIdAssignmentPolicy.UNIQUE_ID)
nn.passthrough.link(tracker.inputTrackerFrame)
nn.passthrough.link(tracker.inputDetectionFrame)
nn.out.link(tracker.inputDetections)

# Define script node and input to sync detections with HQ frames
script = pipeline.create(dai.node.Script)
tracker.out.link(script.inputs["tracker"]) # tracker output + passthrough detections
cam_rgb.video.link(script.inputs["frames"]) # HQ frames
script.inputs["frames"].setBlocking(False)

# Set script that will be run on-device (Luxonis OAK) # (3)
script.setScript('''
# Create empty list to save HQ frames + sequence numbers
lst = []

def get_synced_frame(track_seq):
    """Compare tracker with frame sequence number and send frame if equal."""
    global lst
    for i, frame in enumerate(lst):
        if track_seq == frame.getSequenceNum():
            lst = lst[i:]
            break
    return lst[0]

# Sync tracker output with HQ frames
while True:
    lst.append(node.io["frames"].get())
    tracks = node.io["tracker"].tryGet()
    if tracks is not None:
        track_seq = node.io["tracker"].get().getSequenceNum()
        if len(lst) == 0: continue
        node.io["frame_out"].send(get_synced_frame(track_seq))
        node.io["track_out"].send(tracks)
        lst.pop(0) # remove synchronized frame from the list
''')

# Get synced tracker output (+ passthrough detections)
xout_track = pipeline.create(dai.node.XLinkOut)
xout_track.setStreamName("track")
script.outputs["track_out"].link(xout_track.input)

# Get synced HQ frames
xout_frame = pipeline.create(dai.node.XLinkOut)
xout_frame.setStreamName("frame")
script.outputs["frame_out"].link(xout_frame.input)

# Create new folders for each day, recording interval and object class
rec_start = datetime.now().strftime("%Y%m%d_%H-%M")
save_path = f"./insect-detect/data/{rec_start[:8]}/{rec_start}"
for text in labels:
    Path(f"{save_path}/cropped/{text}").mkdir(parents=True, exist_ok=True)
    if args.save_raw_frames:
        Path(f"{save_path}/raw/{text}").mkdir(parents=True, exist_ok=True)
    if args.save_overlay_frames:
        Path(f"{save_path}/overlay/{text}").mkdir(parents=True, exist_ok=True)

# Calculate current recording ID by subtracting number of directories with date-prefix
folders_dates = len([f for f in Path("./insect-detect/data").glob("**/20*") if f.is_dir()])
folders_days = len([f for f in Path("./insect-detect/data").glob("20*") if f.is_dir()])
rec_id = folders_dates - folders_days


def frame_norm(frame, bbox):
    """Convert relative bounding box coordinates (0-1) to pixel coordinates."""
    norm_vals = np.full(len(bbox), frame.shape[0])
    norm_vals[::2] = frame.shape[1]
    return (np.clip(np.array(bbox), 0, 1) * norm_vals).astype(int)

def store_data(frame, tracklets):
    """Save synced cropped (+ full) frames and tracker output (+ detections) to .jpg and metadata .csv."""
    with open(f"{save_path}/metadata_{rec_start}.csv", "a", encoding="utf-8") as metadata_file:
        metadata = csv.DictWriter(metadata_file, fieldnames=
            ["rec_ID", "timestamp", "label", "confidence", "track_ID",
             "x_min", "y_min", "x_max", "y_max", "file_path"])
        if metadata_file.tell() == 0:
            metadata.writeheader() # write header only once
        for t in tracklets:
            # Do not save (cropped) frames when tracking status == "NEW" or "LOST" or "REMOVED"
            if t.status.name == "TRACKED": # (4)!
                timestamp = datetime.now().strftime("%Y%m%d_%H-%M-%S.%f")

                # Save detection cropped from synced HQ frame
                bbox = frame_norm(frame, (t.srcImgDetection.xmin, t.srcImgDetection.ymin,
                                          t.srcImgDetection.xmax, t.srcImgDetection.ymax))
                det_crop = frame[bbox[1]:bbox[3], bbox[0]:bbox[2]]
                cropped_path = f"{save_path}/cropped/{labels[t.srcImgDetection.label]}/{timestamp}_{t.id}_cropped.jpg"
                cv2.imwrite(cropped_path, det_crop)

                # Save full raw HQ frame (e.g. for training data collection)
                if args.save_raw_frames:
                    raw_path = f"{save_path}/raw/{labels[t.srcImgDetection.label]}/{timestamp}_{t.id}_raw.jpg"
                    cv2.imwrite(raw_path, frame)

                # Save full HQ frame with overlay (bounding box, label, confidence, tracking ID) drawn on frame
                # text position, font size and thickness optimized for 3840x2160 HQ frame size
                if args.save_overlay_frames: # (5)!
                    overlay_frame = frame.copy()
                    cv2.putText(overlay_frame, labels[t.srcImgDetection.label], (bbox[0], bbox[3] + 80),
                                cv2.FONT_HERSHEY_SIMPLEX, 3, (255, 255, 255), 6)
                    cv2.putText(overlay_frame, f"{round(t.srcImgDetection.confidence, 2)}", (bbox[0], bbox[3] + 140),
                                cv2.FONT_HERSHEY_SIMPLEX, 2, (255, 255, 255), 5)
                    cv2.putText(overlay_frame, f"t.ID:{t.id}", (bbox[0], bbox[3] + 240),
                                cv2.FONT_HERSHEY_SIMPLEX, 3, (255, 255, 255), 6)
                    cv2.rectangle(overlay_frame, (bbox[0], bbox[1]), (bbox[2], bbox[3]), (0, 0, 255), 3)
                    overlay_path = f"{save_path}/overlay/{labels[t.srcImgDetection.label]}/{timestamp}_{t.id}_overlay.jpg"
                    cv2.imwrite(overlay_path, overlay_frame)

                # Save corresponding metadata to .csv file for each cropped detection
                data = {
                    "rec_ID": rec_id,
                    "timestamp": timestamp,
                    "label": labels[t.srcImgDetection.label],
                    "confidence": round(t.srcImgDetection.confidence, 2),
                    "track_ID": t.id,
                    "x_min": round(t.srcImgDetection.xmin, 4),
                    "y_min": round(t.srcImgDetection.ymin, 4),
                    "x_max": round(t.srcImgDetection.xmax, 4),
                    "y_max": round(t.srcImgDetection.ymax, 4),
                    "file_path": cropped_path
                }
                metadata.writerow(data)
                metadata_file.flush() # write data immediately to .csv to avoid potential data loss

def record_log(): # (6)!
    """Write information about each recording interval to .csv file."""
    try:
        df_meta = pd.read_csv(f"{save_path}/metadata_{rec_start}.csv", encoding="utf-8")
        unique_ids = df_meta.track_ID.nunique()
    except pd.errors.EmptyDataError:
        unique_ids = 0
    with open("./insect-detect/data/record_log.csv", "a", encoding="utf-8") as log_rec_file:
        log_rec = csv.DictWriter(log_rec_file, fieldnames=
            ["rec_ID", "record_start_date", "record_start_time", "record_end_time", "record_time_min",
             "num_crops", "num_IDs", "disk_free_gb", "chargelevel_start", "chargelevel_end"])
        if log_rec_file.tell() == 0:
            log_rec.writeheader()
        logs_rec = {
            "rec_ID": rec_id,
            "record_start_date": rec_start[:8],
            "record_start_time": rec_start[9:],
            "record_end_time": datetime.now().strftime("%H-%M"),
            "record_time_min": round((time.monotonic() - time_start) / 60, 2),
            "num_crops": len(list(Path(f"{save_path}/cropped").glob("**/*.jpg"))),
            "num_IDs": unique_ids,
            "disk_free_gb": round(psutil.disk_usage("/").free / 1073741824, 1),
            "chargelevel_start": chargelevel_start,
            "chargelevel_end": chargelevel
        }
        log_rec.writerow(logs_rec)

def save_logs(): # (7)!
    """
    Write time, PiJuice battery info + temp, RPi CPU + OAK VPU temp and
    RPi available memory (MB) + CPU utilization (percent) to .csv file.
    """
    with open(f"./insect-detect/data/{rec_start[:8]}/info_log_{rec_start[:8]}.csv",
              "a", encoding="utf-8") as log_info_file:
        log_info = csv.DictWriter(log_info_file, fieldnames=
            ["rec_ID", "timestamp", "power_input", "charge_status", "charge_level",
             "current_batt_mA", "current_gpio_mA", "voltage_batt_mV", "temp_batt",
             "temp_pi", "temp_oak", "pi_mem_available", "pi_cpu_used"])
        if log_info_file.tell() == 0:
            log_info.writeheader()
        logs_info = {
            "rec_ID": rec_id,
            "timestamp": datetime.now().strftime("%Y%m%d_%H-%M-%S"),
            "power_input": pijuice.status.GetStatus().get("data", {}).get("powerInput", 0),
            "charge_status": pijuice.status.GetStatus().get("data", {}).get("battery", 0),
            "charge_level": chargelevel,
            "current_batt_mA": pijuice.status.GetBatteryCurrent().get("data", 0),
            "current_gpio_mA": pijuice.status.GetIoCurrent().get("data", 0),
            "voltage_batt_mV": pijuice.status.GetBatteryVoltage().get("data", 0),
            "temp_batt": pijuice.status.GetBatteryTemperature().get("data", 0),
            "temp_pi": round(CPUTemperature().temperature),
            "temp_oak": round(device.getChipTemperature().average),
            "pi_mem_available": round(psutil.virtual_memory().available / 1048576),
            "pi_cpu_used": psutil.cpu_percent(interval=None)
        }
        log_info.writerow(logs_info)
        log_info_file.flush()


# Connect to OAK device and start pipeline
with dai.Device(pipeline, usb2Mode=True) as device:

    # Create output queues to get the frames and detections from the outputs defined above
    q_frame = device.getOutputQueue(name="frame", maxSize=4, blocking=False)
    q_track = device.getOutputQueue(name="track", maxSize=4, blocking=False)

    chargelevel = chargelevel_start
    time_start = time.monotonic()

    # Define recording time conditional on PiJuice battery charge level
    if chargelevel >= 70: # (8)!
        rec_time = 60 * 40
    elif 50 <= chargelevel < 70:
        rec_time = 60 * 30
    elif 30 <= chargelevel < 50:
        rec_time = 60 * 20
    elif 15 <= chargelevel < 30:
        rec_time = 60 * 10
    else:
        rec_time = 60 * 5
    logger.info(f"Rec ID: {rec_id} | Rec time: {int(rec_time / 60)} min | Charge level: {chargelevel}%")

    try:
        # Record until recording time is finished or chargelevel drops below threshold
        while time.monotonic() < time_start + rec_time and chargelevel >= 10: # (9)!

            # Update PiJuice battery charge level
            chargelevel = pijuice.status.GetChargeLevel().get("data", -1)

            # Get synced HQ frames and tracker output (detections + tracking IDs)
            frame_synced = q_frame.get().getCvFrame()
            track_synced = q_track.get()
            if track_synced is not None:
                tracklets_data = track_synced.tracklets
                if frame_synced is not None:
                    store_data(frame_synced, tracklets_data)
                    time.sleep(1) # wait 1 second to save cropped detections # (10)
                    if args.save_raw_frames or args.save_overlay_frames:
                        time.sleep(2) # wait 2 seconds longer to save additional raw frames

            # Write PiJuice battery info + temp, RPi CPU and OAK VPU temp, RPi info to .csv log file
            if args.save_logs:
                save_logs()

        # Write record logs to .csv and shutdown Raspberry Pi after recording time is finished
        record_log()
        logger.info(f"Recording {rec_id} finished | Charge level: {chargelevel}%\n")
        subprocess.run(["sudo", "shutdown", "-h", "now"], check=True) # (11)!

    # Write record logs to .csv, log error traceback and shutdown Raspberry Pi if an error occurs
    except Exception:
        logger.error(traceback.format_exc())
        logger.error(f"Error during recording {rec_id} | Charge level: {chargelevel}%\n")
        record_log()
        subprocess.run(["sudo", "shutdown", "-h", "now"], check=True)

```

1.  With the logger set up, you can send any information you need (e.g. for
    finding problems if something went wrong) to the log file. This is a good
    alternative to `print()`, if you don't have a terminal output. You can add
    your own custom logging command in any line with:

    ``` py
    logger.info("I want to write this to my log file.")
    ```

2.  If the PiJuice battery charge level is below **10%** or the free space left
    on the SD card is below **200 MB**, no recording will be made and the
    Raspberry Pi is immediately shut down. You can specify your custom
    thresholds (e.g. when using a different battery capacity) in this line.
3.  This script is run on the OAK device and will synchronize the output from
    the object tracker node (+ passthrough model detections) with the HQ frames
    by comparing their respective sequence numbers. The synchronized tracker
    output and HQ frame are then send to the host (RPi).
4.  A tracked object can have 4 possible statuses: `NEW`, `TRACKED`, `LOST` and
    `REMOVED`. It is highly recommended to save the cropped detections only
    when the `tracking status == TRACKED`, but you could change this
    configuration here and e.g. write the `t.status.name` as additional column
    to the metadata .csv file.
5.  If you are using a different resolution for the HQ frames than 4K
    (3840x2160 px), you have to adjust the position, font size and thickness of
    the text and rectangle (bounding box) that is drawn on the overlay frame
    accordingly.
6.  This function will be called after a recording interval is finished, or if
    an error occurs during the recording and will write some info about the
    respective recording interval to the `record_log.csv` file.
7.  This function will be called if you are using the optional `-log` argument
    and will save the specified logging info to a .csv file.
8.  You can specify your own recording durations and charge level thresholds in
    this code section. The suggested values can provide an efficient recording
    behaviour if you are using the 12,000 mAh PiJuice battery and set up the
    [Wakeup Alarm](../pisetup/#pijuice-zero-configuration) for 2-5 times per
    day. Depending on the number of Wakeups per day, as well as the season and
    sun exposure of the solar panel, it could make sense to increase or
    decrease the recording duration of the camera trap.
9.  The recording will be stopped if the charge level of the PiJuice battery
    that is updated in each while loop drops below the specified threshold. You
    can adjust this threshold, e.g. when using a different battery capacity.
10. In this line you can change the time interval with which the cropped
    detections will be saved to .jpg. This does not affect the detection model
    and object tracker, which are both run on-device even if no detections are
    saved. To avoid potential memory errors, it is recommended to wait at least
    2 seconds when saving additional raw HQ frames (`-raw`).
11. If you are still in the testing phase, comment out the shutdown commands in
    this line and the last line by adding `#` in front of the line.

---

## Time-lapse image capture

This Python script enables the capture of images at the highest possible
resolution of the
[supported sensors](https://docs.luxonis.com/projects/hardware/en/latest/pages/articles/supported_sensors.html)
at a specified time interval (e.g. every 3 seconds). This will lead to a bigger
field of view (FOV), compared to the other scripts where the camera is set to
4K resolution. You can find more information on sensor resolution and image
types at the
[DepthAI API Docs](https://docs.luxonis.com/projects/api/en/latest/components/nodes/color_camera/).
You can use this script for training data collection if the provided detection
model does not work for you, or if you want to detect other objects than
insects.

Run the script with:

``` bash
python3 ./insect-detect/still_capture_timelapse.py
```

``` py title="still_capture_timelapse.py" hl_lines="20 24 43 61"
'''
Author:   Maximilian Sittinger (https://github.com/maxsitt)
License:  GNU GPLv3 (https://choosealicense.com/licenses/gpl-3.0/)

includes segments from open source scripts available at https://github.com/luxonis
'''

from datetime import datetime
from pathlib import Path

import cv2
import depthai as dai

# Create depthai pipeline
pipeline = dai.Pipeline()

# Define camera source
cam_rgb = pipeline.create(dai.node.ColorCamera)
#cam_rgb.setImageOrientation(dai.CameraImageOrientation.ROTATE_180_DEG)
cam_rgb.setResolution(dai.ColorCameraProperties.SensorResolution.THE_12_MP) # OAK-1 (IMX378) # (1)
#cam_rgb.setResolution(dai.ColorCameraProperties.SensorResolution.THE_13_MP) # OAK-1 Lite (IMX214)
#cam_rgb.setResolution(dai.ColorCameraProperties.SensorResolution.THE_5312X6000) # OAK-1 MAX (LCM48)
cam_rgb.setNumFramesPool(2,2,2,2,2) # (2)!
cam_rgb.setFps(10) # frames per second available for focus/exposure # (3)

# Define MJPEG encoder
still_enc = pipeline.create(dai.node.VideoEncoder) # (4)!
still_enc.setDefaultProfilePreset(1, dai.VideoEncoderProperties.Profile.MJPEG)
still_enc.setNumFramesPool(1)

# Define script node
script = pipeline.create(dai.node.Script)

# Set script that will be run on-device (Luxonis OAK) # (5)
script.setScript('''
import time

ctrl = CameraControl()
ctrl.setCaptureStill(True)

while True:
    node.io["capture_still"].send(ctrl)
    time.sleep(3) # capture still image every 3 seconds
''')

# Send capture command to camera and still image to the MJPEG encoder
script.outputs["capture_still"].link(cam_rgb.inputControl)
cam_rgb.still.link(still_enc.input)

xout_still = pipeline.create(dai.node.XLinkOut)
xout_still.setStreamName("still")
still_enc.bitstream.link(xout_still.input)

# Connect to OAK device and start pipeline
with dai.Device(pipeline, usb2Mode=True) as device:

    # Create output queue to get the encoded still images
    q_still = device.getOutputQueue(name="still", maxSize=1, blocking=False)

    rec_start = datetime.now().strftime("%Y%m%d_%H-%M")
    save_path = f"./insect-detect/still/{rec_start[:8]}/{rec_start}" # (6)!
    Path(f"{save_path}").mkdir(parents=True, exist_ok=True)

    while True:
        enc_still = q_still.get()
        timestamp = datetime.now().strftime("%Y%m%d_%H-%M-%S.%f")
        with open(f"{save_path}/{timestamp}_still.jpg", "wb") as still_jpg:
            still_jpg.write(bytearray(enc_still.getData()))

```

1.  `THE_12_MP` = 4032x3040 pixel. Check out other possible
    [sensor resolutions](https://docs.luxonis.com/projects/api/en/latest/references/python/#depthai.ColorCameraProperties.SensorResolution).
2.  The maximum number of frames in all pools (raw, isp, preview, video, still)
    is set to 2, to avoid a potential out-of-memory error, especially when
    saving images with the OAK-1 MAX at 5312x6000 px.
3.  10 fps is currently the maximum framerate supported by the OAK-1 MAX at
    full resolution. You can increase this to e.g. 20 fps for faster auto-focus/
    -exposure/-white balance if you are using lower sensor resolutions.
4.  More info about the [VideoEncoder node](https://docs.luxonis.com/projects/api/en/latest/components/nodes/video_encoder/)
5.  You can set the time interval to capture the still images in the script
    that will be run on the OAK device. It is recommended to wait at least one
    second (`time.sleep(1)`) between capturing and saving an image, as the OAK
    could run into an out-of-memory error otherwise.
6.  For each recording day and time a respective folder is created. If you want
    to save all still images into the same directory, you can change this here.
    Be careful not to save any folder with date prefix into the
    `./insect-detect/data` directory, as this will lead to the calculation of
    wrong recording IDs if the monitoring script is used with the same system.
