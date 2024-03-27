# Programming

!!! tip "Adapt the software to your use case"

    You will find all Python scripts to deploy the **Insect Detect** DIY camera
    trap for automated insect monitoring in this section, together with
    suggestions on possible modifications. Click on the :material-plus-circle:
    symbol to open the code annotations for more information. More details
    about the API that is used can be found at the
    [DepthAI API Docs](https://docs.luxonis.com/projects/api/en/latest/){target=_blank}.

!!! bug ""

    2024-03-27: The scripts in this section are currently out of date! Will be updated soon.

    Please check the [`insect-detect`](https://github.com/maxsitt/insect-detect){target=_blank}
    GitHub repo for the latest updates.

The latest versions of the Python scripts are available in the
[`insect-detect`](https://github.com/maxsitt/insect-detect){target=_blank} GitHub repo.
[Download](https://github.com/maxsitt/insect-detect/archive/refs/heads/main.zip){target=_blank}
the whole repository, extract it and change its foldername to `insect-detect`. Copy
the renamed folder to the `home/pi` directory of your Raspberry Pi, by simply dragging
& dropping it into the SSH FS Workspace folder (or VS Code remote window explorer).

If you run into any problems, find a bug or something that could be optimized,
please post an [issue](https://github.com/maxsitt/insect-detect/issues){target=_blank}
at the GitHub repo. Get OAK-specific [support](https://docs.luxonis.com/en/latest/pages/support/){target=_blank}
directly from Luxonis.

??? bug "X_LINK_DEVICE_ALREADY_IN_USE"

    If you try to run a script and the following error occurs:

    ``` bash
    RuntimeError: Failed to connect to device, error message: X_LINK_DEVICE_ALREADY_IN_USE
    ```

    ...chances are high that a previously started script (e.g. because of an
    active cron job at boot) is still running in the background. You can see
    all currently running processes by using Raspberry Pi's task manager
    [htop](https://en.wikipedia.org/wiki/Htop){target=_blank}. Start the
    task manager by running:

    ``` bash
    htop
    ```

    If one of the Python scripts that is using the OAK-1 camera is currently
    running, press ++f9++ with the script selected (usually one of the processes
    with highest CPU utilization at the top). This will open the `SIGTERM` option
    and by confirming with ++enter++ the script will be stopped. Close htop by
    pressing ++q++.

---

## OAK camera preview

This Python script will create and configure the
[ColorCamera node](https://docs.luxonis.com/projects/api/en/latest/components/nodes/color_camera/){target=_blank}
to send downscaled LQ frames (e.g. 320x320 px) to the host (Raspberry Pi) and
show them in a new window. If you are connected to the RPi via SSH,
[X11 forwarding](pisetup.md#ssh-connection-and-x11-forwarding){target=_blank} has to be
set up together with an active
[X server](localsetup.md#vcxsrv-windows-x-server){target=_blank} to show the
frames in a window on your local PC.

??? bug "libGL error"

    When running one of the preview scripts, you might get the following error:

    ``` bash
    libGL error: No matching fbConfigs or visuals found
    libGL error: failed to load driver: swrast
    ```

    You can ignore this error, as everything will still work as expected.
    It will only be printed to the console once after each boot.

Run the script with:

``` bash
python3 insect-detect/cam_preview.py
```

Stop the script by hitting ++q++ with the preview window selected, or by
pressing ++ctrl+c++ in the Terminal.

``` py title="cam_preview.py" hl_lines="18 20 21 24"
'''
Author:   Maximilian Sittinger (https://github.com/maxsitt)
License:  GNU GPLv3 (https://choosealicense.com/licenses/gpl-3.0/)

based on open source scripts available at https://github.com/luxonis
'''

import time

import cv2
import depthai as dai

# Create depthai pipeline
pipeline = dai.Pipeline() # (1)!

# Create and configure camera node and define output
cam_rgb = pipeline.create(dai.node.ColorCamera) # (2)!
#cam_rgb.setImageOrientation(dai.CameraImageOrientation.ROTATE_180_DEG) # (3)
cam_rgb.setResolution(dai.ColorCameraProperties.SensorResolution.THE_1080_P) # (4)!
cam_rgb.setPreviewSize(320, 320) # downscaled LQ frames
cam_rgb.setPreviewKeepAspectRatio(False) # "squeeze" frames to square # (5)
cam_rgb.setInterleaved(False) # planar layout
cam_rgb.setColorOrder(dai.ColorCameraProperties.ColorOrder.BGR)
cam_rgb.setFps(25) # frames per second available for focus/exposure

xout_rgb = pipeline.create(dai.node.XLinkOut) # (6)!
xout_rgb.setStreamName("frame")
cam_rgb.preview.link(xout_rgb.input)

# Connect to OAK device and start pipeline in USB2 mode
with dai.Device(pipeline, maxUsbSpeed=dai.UsbSpeed.HIGH) as device: # (7)!

    # Create output queue to get the frames from the output defined above
    q_frame = device.getOutputQueue(name="frame", maxSize=4, blocking=False) # (8)!

    # Create start_time and counter variable to measure fps
    start_time = time.monotonic()
    counter = 0

    # Get LQ frames and show in new window
    while True:
        frame = q_frame.get().getCvFrame()
        counter += 1
        fps = round(counter / (time.monotonic() - start_time), 2)

        cv2.putText(frame, f"fps: {fps}", (4, frame.shape[0] - 10),
                    cv2.FONT_HERSHEY_SIMPLEX, 0.7, (255, 255, 255), 2) # (9)!
        cv2.imshow("cam_preview", frame)

        # Stop script and close window by pressing "Q"
        if cv2.waitKey(1) == ord("q"):
            break

```

1.  More info about the [Pipeline](https://docs.luxonis.com/projects/api/en/latest/components/pipeline/){target=_blank}
    and creation of nodes.
2.  More info about the
    [ColorCamera node](https://docs.luxonis.com/projects/api/en/latest/components/nodes/color_camera/){target=_blank}
    and possible configurations.
3.  If your image is shown upside down (on older OAK-1 cameras), you can rotate
    it on-device by activating this configuration.
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
    by setting `maxUsbSpeed=dai.UsbSpeed.HIGH`.
8.  You can
    [specify different queue configurations](https://docs.luxonis.com/projects/api/en/latest/components/device/#specifying-arguments-for-getoutputqueue-method){target=_blank},
    by changing the maximum queue size or the blocking behaviour.
9.  More info about
    [`cv2.putText()`](https://www.askpython.com/python-modules/opencv-puttext){target=_blank}
    to customize your output.

---

## YOLO preview

With the following Python script you can run a custom YOLO object detection model
([.blob format](https://docs.luxonis.com/en/latest/pages/model_conversion){target=_blank})
on the OAK device with downscaled LQ frames (e.g. 320x320 px) as model input
and show the frames together with the model output (bounding box, label,
confidence score) in a new window.

If you copied the whole
[`insect-detect`](https://github.com/maxsitt/insect-detect){target=_blank}
GitHub repo to your Raspberry Pi, the provided YOLOv5n detection model and
config .json will be used by default. If you want to use a different model,
change the `MODEL_PATH` and `CONFIG_PATH` accordingly.

Run the script with:

``` bash
python3 insect-detect/yolo_preview.py
```

??? info "Optional argument"

    Add after `python3 insect-detect/yolo_preview.py`, separated by space:

    - `-log` to print available Raspberry Pi memory, RPi CPU utilization +
      temperature, OAK memory + CPU usage and OAK chip temperature to console

Stop the script by hitting ++q++ with the preview window selected, or by
pressing ++ctrl+c++ in the Terminal.

``` py title="yolo_preview.py" hl_lines="30 31 58 79 80 102 136 137"
'''
Author:   Maximilian Sittinger (https://github.com/maxsitt)
License:  GNU GPLv3 (https://choosealicense.com/licenses/gpl-3.0/)

based on open source scripts available at https://github.com/luxonis
'''

import argparse
import json
import time
from pathlib import Path

import cv2
import depthai as dai
import numpy as np

# Define optional argument
parser = argparse.ArgumentParser()
parser.add_argument("-log", "--print_logs", action="store_true",
    help="print RPi available memory, RPi CPU utilization + temperature, \
          OAK memory + CPU usage and OAK chip temperature to console")
args = parser.parse_args()

if args.print_logs:
    import psutil
    from apscheduler.schedulers.background import BackgroundScheduler
    from gpiozero import CPUTemperature

# Set file paths to the detection model and config JSON
MODEL_PATH = Path("insect-detect/models/yolov5n_320_openvino_2022.1_4shave.blob")
CONFIG_PATH = Path("insect-detect/models/json/yolov5_v7_320.json") # (1)!

# Get detection model metadata from config JSON
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

# Create and configure camera node
cam_rgb = pipeline.create(dai.node.ColorCamera)
#cam_rgb.setImageOrientation(dai.CameraImageOrientation.ROTATE_180_DEG)
cam_rgb.setResolution(dai.ColorCameraProperties.SensorResolution.THE_1080_P)
cam_rgb.setPreviewSize(320, 320) # downscaled LQ frames for model input
cam_rgb.setPreviewKeepAspectRatio(False) # "squeeze" frames (16:9) to square (1:1)
cam_rgb.setInterleaved(False) # planar layout
cam_rgb.setColorOrder(dai.ColorCameraProperties.ColorOrder.BGR)
cam_rgb.setFps(49) # frames per second available for model input # (2)

# Create detection network node and define input + outputs
nn = pipeline.create(dai.node.YoloDetectionNetwork) # (3)!
cam_rgb.preview.link(nn.input) # downscaled LQ frames as model input # (4)
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
nn.setIouThreshold(iou_threshold) # (6)!
nn.setConfidenceThreshold(confidence_threshold)
nn.setNumInferenceThreads(2)

# Define functions
def frame_norm(frame, bbox):
    """Convert relative bounding box coordinates (0-1) to pixel coordinates."""
    norm_vals = np.full(len(bbox), frame.shape[0])
    norm_vals[::2] = frame.shape[1]
    return (np.clip(np.array(bbox), 0, 1) * norm_vals).astype(int)

def print_logs():
    """Print Raspberry Pi info to console."""
    print(f"\nAvailable RPi memory: {round(psutil.virtual_memory().available / 1048576)} MB")
    print(f"RPi CPU utilization:  {round(psutil.cpu_percent(interval=None))} %")
    print(f"RPi CPU temperature:  {round(CPUTemperature().temperature)} °C\n")

# Connect to OAK device and start pipeline in USB2 mode
with dai.Device(pipeline, maxUsbSpeed=dai.UsbSpeed.HIGH) as device:

    # Print RPi + OAK info to console every second
    if args.print_logs:
        scheduler = BackgroundScheduler()
        scheduler.add_job(print_logs, "interval", seconds=1, id="log") # (7)!
        scheduler.start()
        device.setLogLevel(dai.LogLevel.INFO)
        device.setLogOutputLevel(dai.LogLevel.INFO)

    # Create output queues to get the frames and detections from the outputs defined above
    q_frame = device.getOutputQueue(name="frame", maxSize=4, blocking=False)
    q_nn = device.getOutputQueue(name="nn", maxSize=4, blocking=False)

    # Create start_time and counter variable to measure fps
    start_time = time.monotonic()
    counter = 0

    # Get LQ frames + model output (detections) and show in new window
    while True:
        if q_frame.has():
            frame = q_frame.get().getCvFrame()

            if q_nn.has():
                dets = q_nn.get().detections
                counter += 1
                fps = round(counter / (time.monotonic() - start_time), 2)

                for detection in dets:
                    bbox = frame_norm(frame, (detection.xmin, detection.ymin,
                                              detection.xmax, detection.ymax))
                    cv2.putText(frame, labels[detection.label], (bbox[0], bbox[3] + 13),
                                cv2.FONT_HERSHEY_SIMPLEX, 0.4, (255, 255, 255), 1)
                    cv2.putText(frame, f"{round(detection.confidence, 2)}", (bbox[0], bbox[3] + 25),
                                cv2.FONT_HERSHEY_SIMPLEX, 0.4, (255, 255, 255), 1)
                    cv2.rectangle(frame, (bbox[0], bbox[1]), (bbox[2], bbox[3]), (0, 0, 255), 2)

                cv2.putText(frame, f"fps: {fps}", (4, frame.shape[0] - 10),
                            cv2.FONT_HERSHEY_SIMPLEX, 0.7, (255, 255, 255), 2)
                cv2.imshow("yolo_preview", frame)
                #print(f"fps: {fps}")
                # streaming the frames via SSH (X11 forwarding) will slow down fps
                # comment out "cv2.imshow()" and print fps to console for true fps

        # Stop script and close window by pressing "Q"
        if cv2.waitKey(1) == ord("q"):
            break

```

1.  Specify the path to your detection model (.blob format) and config JSON
    that the OAK can use it for on-device inference.
2.  Adjust the camera fps to the maximum fps of the YOLO model that is used for inference
    ([more info](https://docs.luxonis.com/projects/api/en/latest/tutorials/low-latency/#lowering-camera-fps-to-match-nn-fps){target=_blank}).
3.  More info about the
    [YoloDetectionNetwork node](https://docs.luxonis.com/projects/api/en/latest/components/nodes/yolo_detection_network/){target=_blank}.
4.  The downscaled LQ preview frames are used as model input. More info about
    [linking nodes](https://docs.luxonis.com/projects/api/en/latest/components/nodes/){target=_blank}.
5.  To avoid freezing of the pipeline, we will set
    [`blocking=False`](https://docs.luxonis.com/projects/api/en/latest/components/device/#non-blocking-behaviour){target=_blank}
    for the frames that are used as model input.
6.  All metadata that is necessary to successfully run the model on-device is
    extracted from the corresponding config .json file. However, you could also change
    your IoU or confidence threshold here for experimenting with different settings.
7.  To match the output frequency of the OAK logs
    ([`device.setLogOutputLevel()`](https://docs.luxonis.com/projects/api/en/latest/tutorials/debugging/){target=_blank}),
    the interval for printing the RPi logs is set to one second. Increase this
    interval for easier reading of the logs.

---

## YOLO + object tracker preview

In the following Python script an
[ObjectTracker node](https://docs.luxonis.com/projects/api/en/latest/components/nodes/object_tracker/){target=_blank},
based on the [Intel DL Streamer](https://dlstreamer.github.io/dev_guide/object_tracking.html){target=_blank}
framework, is added to the pipeline. The object tracker uses the detection model
output as input for tracking detected objects and assigning unique tracking IDs
on-device. The frames, together with the model and tracker output (bounding box
from tracker output and bbox from detection model, label, confidence score,
tracking ID, tracking status), are shown in a new window.

Run the script with:

``` bash
python3 insect-detect/yolo_tracker_preview.py
```

??? info "Optional argument"

    Add after `python3 insect-detect/yolo_tracker_preview.py`, separated by space:

    - `-log` to print available Raspberry Pi memory, RPi CPU utilization +
      temperature, OAK memory + CPU usage and OAK chip temperature to console

Stop the script by hitting ++q++ with the preview window selected, or by
pressing ++ctrl+c++ in the Terminal.

``` py title="yolo_tracker_preview.py" hl_lines="30 31 58 77 135 151 152 156 157"
'''
Author:   Maximilian Sittinger (https://github.com/maxsitt)
License:  GNU GPLv3 (https://choosealicense.com/licenses/gpl-3.0/)

based on open source scripts available at https://github.com/luxonis
'''

import argparse
import json
import time
from pathlib import Path

import cv2
import depthai as dai
import numpy as np

# Define optional argument
parser = argparse.ArgumentParser()
parser.add_argument("-log", "--print_logs", action="store_true",
    help="print RPi available memory, RPi CPU utilization + temperature, \
          OAK memory + CPU usage and OAK chip temperature to console")
args = parser.parse_args()

if args.print_logs:
    import psutil
    from apscheduler.schedulers.background import BackgroundScheduler
    from gpiozero import CPUTemperature

# Set file paths to the detection model and config JSON
MODEL_PATH = Path("insect-detect/models/yolov5n_320_openvino_2022.1_4shave.blob")
CONFIG_PATH = Path("insect-detect/models/json/yolov5_v7_320.json")

# Get detection model metadata from config JSON
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

# Create and configure camera node
cam_rgb = pipeline.create(dai.node.ColorCamera)
#cam_rgb.setImageOrientation(dai.CameraImageOrientation.ROTATE_180_DEG)
cam_rgb.setResolution(dai.ColorCameraProperties.SensorResolution.THE_1080_P)
cam_rgb.setPreviewSize(320, 320) # downscaled LQ frames for model input
cam_rgb.setPreviewKeepAspectRatio(False) # "squeeze" frames (16:9) to square (1:1)
cam_rgb.setInterleaved(False) # planar layout
cam_rgb.setColorOrder(dai.ColorCameraProperties.ColorOrder.BGR)
cam_rgb.setFps(47) # frames per second available for model input # (1)

# Create detection network node and define input
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

# Create and configure object tracker node and define inputs + outputs
tracker = pipeline.create(dai.node.ObjectTracker) # (2)!
tracker.setTrackerType(dai.TrackerType.ZERO_TERM_IMAGELESS) # (3)!
#tracker.setTrackerType(dai.TrackerType.SHORT_TERM_IMAGELESS) # better for low fps
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

# Define functions
def frame_norm(frame, bbox):
    """Convert relative bounding box coordinates (0-1) to pixel coordinates."""
    norm_vals = np.full(len(bbox), frame.shape[0])
    norm_vals[::2] = frame.shape[1]
    return (np.clip(np.array(bbox), 0, 1) * norm_vals).astype(int)

def print_logs():
    """Print Raspberry Pi info to console."""
    print(f"\nAvailable RPi memory: {round(psutil.virtual_memory().available / 1048576)} MB")
    print(f"RPi CPU utilization:  {round(psutil.cpu_percent(interval=None))} %")
    print(f"RPi CPU temperature:  {round(CPUTemperature().temperature)} °C\n")

# Connect to OAK device and start pipeline in USB2 mode
with dai.Device(pipeline, maxUsbSpeed=dai.UsbSpeed.HIGH) as device:

    # Print RPi + OAK info to console every second
    if args.print_logs:
        scheduler = BackgroundScheduler()
        scheduler.add_job(print_logs, "interval", seconds=1, id="log")
        scheduler.start()
        device.setLogLevel(dai.LogLevel.INFO)
        device.setLogOutputLevel(dai.LogLevel.INFO)

    # Create output queues to get the frames and tracklets + detections from the outputs defined above
    q_frame = device.getOutputQueue(name="frame", maxSize=4, blocking=False)
    q_track = device.getOutputQueue(name="track", maxSize=4, blocking=False)

    # Create start_time and counter variable to measure fps
    start_time = time.monotonic()
    counter = 0

    # Get LQ frames + tracker output (passthrough detections) and show in new window
    while True:
        if q_frame.has():
            frame = q_frame.get().getCvFrame()

            if q_track.has():
                tracks = q_track.get().tracklets
                counter += 1
                fps = round(counter / (time.monotonic() - start_time), 2)

                for track in tracks:
                    roi = track.roi.denormalize(frame.shape[1], frame.shape[0]) # (4)!
                    x1 = int(roi.topLeft().x)
                    y1 = int(roi.topLeft().y)
                    x2 = int(roi.bottomRight().x)
                    y2 = int(roi.bottomRight().y)

                    bbox = frame_norm(frame, (track.srcImgDetection.xmin, track.srcImgDetection.ymin,
                                              track.srcImgDetection.xmax, track.srcImgDetection.ymax))
                    cv2.putText(frame, labels[track.srcImgDetection.label], (bbox[0], bbox[3] + 13),
                                cv2.FONT_HERSHEY_SIMPLEX, 0.4, (255, 255, 255), 1)
                    cv2.putText(frame, f"{round(track.srcImgDetection.confidence, 2)}", (bbox[0], bbox[3] + 25),
                                cv2.FONT_HERSHEY_SIMPLEX, 0.4, (255, 255, 255), 1)
                    cv2.putText(frame, f"ID:{track.id}", (bbox[0], bbox[3] + 40),
                                cv2.FONT_HERSHEY_SIMPLEX, 0.5, (255, 255, 255), 1)
                    cv2.putText(frame, track.status.name, (bbox[0], bbox[3] + 50),
                                cv2.FONT_HERSHEY_SIMPLEX, 0.3, (255, 255, 255), 1)
                    cv2.rectangle(frame, (bbox[0], bbox[1]), (bbox[2], bbox[3]), (0, 0, 255), 2) # model bbox
                    cv2.rectangle(frame, (x1, y1), (x2, y2), (0, 255, 130), 1) # tracker bbox

                cv2.putText(frame, f"fps: {fps}", (4, frame.shape[0] - 10),
                            cv2.FONT_HERSHEY_SIMPLEX, 0.7, (255, 255, 255), 2)
                cv2.imshow("tracker_preview", frame)
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
    [ObjectTracker node](https://docs.luxonis.com/projects/api/en/latest/components/nodes/object_tracker/){target=_blank}.
3.  More info about the supported
    [tracking types](https://dlstreamer.github.io/dev_guide/object_tracking.html){target=_blank}.
4.  You can use the bounding box coordinates from the tracker output, as
    defined in this segment, and/or the bbox coordinates from the passthrough
    detections to draw the bboxes on the frame. The bboxes from the passthrough
    detections are usually more stable than from the object tracker output, you
    can decide for yourself which one is best for your use case.

---

## Automated monitoring script

The following Python script is the main script for fully
[automated insect monitoring](../deployment/detection.md){target=_blank}.

- The object tracker output (+ passthrough detections) from inference on downscaled
  LQ frames (e.g. 320x320 px) is synchronized with HQ frames (e.g. 1920x1080 px) in a
  [sync node](https://docs.luxonis.com/projects/api/en/latest/components/nodes/sync_node/){target=_blank}
  on-device, using the respective message timestamps.
- Detections (bounding box area) are
  [cropped](https://maxsitt.github.io/insect-detect-docs/deployment/assets/images/hq_frame_sync_1080p.gif){target=_blank}
  from synced HQ frames and saved to .jpg. By default, cropped detections are saved
  with aspect ratio 1:1 (`-crop square`) which increases classification accuracy,
  as the images are not stretched during resizing and no distortion is added.
  Use option `-crop tight` to keep original bbox size with variable aspect ratio.
- All relevant metadata from the detection model and tracker output (timestamp,
  label, confidence score, tracking ID, relative bbox coordinates, .jpg file
  path) is saved to a [metadata .csv](../deployment/detection.md#metadata-csv){target=_blank}
  file for each cropped detection.
- Info and error messages are written to log file. Recording info (recording
  ID, start/end time, duration, number of cropped detections, number of unique
  tracking IDs, free disk space and battery charge level) is written to a
  `record_log.csv` file for each recording interval.
- The [PiJuice I2C Command API](https://github.com/PiSupply/PiJuice/tree/master/Software#i2c-command-api){target=_blank}
  is used for power management. A recording will only be made if the PiJuice
  battery charge level is higher than a specified threshold. The respective
  recording duration is conditional on the current charge level.
- After a recording interval is finished, or if the PiJuice battery charge
  level drops below a specified threshold, or if an error occurs, the Raspberry
  Pi is safely shut down and waits for the next wake up alarm from the PiJuice Zero.

Using the default 1080p resolution for the HQ frames will result in an
pipeline speed of **~13 fps**, which is fast enough to track many insects.
If 4K resolution is used instead (`-4k`), the pipeline speed will decrease to
**~3 fps**, which reduces tracking accuracy for fast moving insects.

For fully automated monitoring in the field, set up a
[cron job](../pisetup/#set-up-cron-job){target=_blank} that will run the script
automatically at each boot (after wake up by the PiJuice Zero).

??? question "No PiJuice Zero?"

    If you want to try the script without the PiJuice Zero pHAT connected to
    your Raspberry Pi, use the
    [`yolo_tracker_save_hqsync.py`](https://github.com/maxsitt/insect-detect/blob/main/yolo_tracker_save_hqsync.py){target=_blank}
    script, available in the [`insect-detect`](https://github.com/maxsitt/insect-detect){target=_blank} GitHub repo.

Run the script with:

``` bash
python3 insect-detect/yolo_tracker_save_hqsync_pijuice.py
```

??? info "Optional arguments"

    Add after `python3 insect-detect/yolo_tracker_save_hqsync_pijuice.py`, separated by space:

    - `-4k` crop detections from (+ save HQ frames in) 4K resolution (default: 1080p)
    - `-af CM_MIN CM_MAX` set auto focus range in cm (min distance, max distance)
    - `-ae` use bounding box coordinates from detections to set auto exposure region
    - `-crop` save cropped detections with aspect ratio 1:1 (default: `-crop square`)
              or keep original bbox size with variable aspect ratio (`-crop tight`)
    - `-raw` additionally save full HQ frames to .jpg (e.g. for training data collection)
    - `-overlay` additionally save full HQ frames with overlays (bbox + info) to .jpg
    - `-log` write RPi CPU + OAK chip temperature, RPi available
             memory + CPU utilization and battery info to .csv
    - `-zip` store data in an uncompressed .zip file for each day and delete original directory

Stop the script by pressing ++ctrl+c++ in the Terminal.

``` py title="yolo_tracker_save_hqsync.py" hl_lines="61 67 68 71 72 120 232 237 362 370 371 393 411 422 435"
'''
Author:   Maximilian Sittinger (https://github.com/maxsitt)
License:  GNU GPLv3 (https://choosealicense.com/licenses/gpl-3.0/)

based on open source scripts available at https://github.com/luxonis
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
from pijuice import PiJuice

# Create folder to save images + metadata + logs (if not already present)
Path("insect-detect/data").mkdir(parents=True, exist_ok=True)

# Create logger and write info + error messages to log file
logging.basicConfig(filename="insect-detect/data/script_log.log", encoding="utf-8",
                    format="%(asctime)s - %(levelname)s: %(message)s", level=logging.INFO)
logger = logging.getLogger() # (1)!
sys.stderr.write = logger.error

# Define optional arguments
parser = argparse.ArgumentParser()
parser.add_argument("-4k", "--four_k_resolution", action="store_true",
    help="crop detections from (+ save HQ frames in) 4K resolution; default = 1080p")
parser.add_argument("-crop", "--crop_bbox", choices=["square", "tight"], default="square", type=str,
    help="save cropped detections with aspect ratio 1:1 ('-crop square') or \
          keep original bbox size with variable aspect ratio ('-crop tight')")
parser.add_argument("-raw", "--save_raw_frames", action="store_true",
    help="additionally save full raw HQ frames in separate folder (e.g. for training data)")
parser.add_argument("-overlay", "--save_overlay_frames", action="store_true",
    help="additionally save full HQ frames with overlay (bbox + info) in separate folder")
parser.add_argument("-log", "--save_logs", action="store_true",
    help="save RPi CPU + OAK chip temperature, RPi available memory (MB) + \
          CPU utilization (%) and battery info to .csv file")
args = parser.parse_args()

if args.save_logs:
    from apscheduler.schedulers.background import BackgroundScheduler
    from gpiozero import CPUTemperature

# Instantiate PiJuice
pijuice = PiJuice(1, 0x14)

# Continue script only if battery charge level and free disk space (MB) are higher than thresholds
chargelevel_start = pijuice.status.GetChargeLevel().get("data", -1)
disk_free = round(psutil.disk_usage("/").free / 1048576)
if chargelevel_start < 10 or disk_free < 200: # (2)!
    logger.info(f"Shut down without recording | Charge level: {chargelevel_start}%\n")
    subprocess.run(["sudo", "shutdown", "-h", "now"], check=True)
    time.sleep(5) # wait 5 seconds for RPi to shut down

# Optional: Disable charging of PiJuice battery if charge level is higher than threshold
#if chargelevel_start > 80: # (3)
#    pijuice.config.SetChargingConfig({"charging_enabled": False})

# Set file paths to the detection model and config JSON
MODEL_PATH = Path("insect-detect/models/yolov5n_320_openvino_2022.1_4shave.blob")
CONFIG_PATH = Path("insect-detect/models/json/yolov5_v7_320.json")

# Get detection model metadata from config JSON
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

# Create and configure camera node
cam_rgb = pipeline.create(dai.node.ColorCamera)
#cam_rgb.setImageOrientation(dai.CameraImageOrientation.ROTATE_180_DEG)
cam_rgb.setResolution(dai.ColorCameraProperties.SensorResolution.THE_4_K)
if not args.four_k_resolution:
    cam_rgb.setIspScale(1, 2) # downscale 4K to 1080p HQ frames (1920x1080 px)
cam_rgb.setPreviewSize(320, 320) # downscaled LQ frames for model input
cam_rgb.setPreviewKeepAspectRatio(False) # "squeeze" frames (16:9) to square (1:1)
cam_rgb.setInterleaved(False) # planar layout
cam_rgb.setColorOrder(dai.ColorCameraProperties.ColorOrder.BGR)
cam_rgb.setFps(25) # frames per second available for focus/exposure/model input

# Create detection network node and define input
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

# Create and configure object tracker node and define inputs
tracker = pipeline.create(dai.node.ObjectTracker)
tracker.setTrackerType(dai.TrackerType.ZERO_TERM_IMAGELESS)
#tracker.setTrackerType(dai.TrackerType.SHORT_TERM_IMAGELESS) # better for low fps
tracker.setTrackerIdAssignmentPolicy(dai.TrackerIdAssignmentPolicy.UNIQUE_ID)
nn.passthrough.link(tracker.inputTrackerFrame)
nn.passthrough.link(tracker.inputDetectionFrame)
nn.out.link(tracker.inputDetections)

# Create script node and define inputs
script = pipeline.create(dai.node.Script)
script.setProcessor(dai.ProcessorType.LEON_CSS)
cam_rgb.video.link(script.inputs["frames"]) # HQ frames
script.inputs["frames"].setBlocking(False)
tracker.out.link(script.inputs["tracker"]) # tracklets + passthrough detections
script.inputs["tracker"].setBlocking(False)

# Set script that will be run on-device (Luxonis OAK) # (4)
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

# Define script node outputs
xout_rgb = pipeline.create(dai.node.XLinkOut)
xout_rgb.setStreamName("frame")
script.outputs["frame_out"].link(xout_rgb.input) # synced HQ frames

xout_tracker = pipeline.create(dai.node.XLinkOut)
xout_tracker.setStreamName("track")
script.outputs["track_out"].link(xout_tracker.input) # synced tracker output

# Create new folders for each day, recording interval and object class
rec_start = datetime.now().strftime("%Y%m%d_%H-%M")
save_path = f"insect-detect/data/{rec_start[:8]}/{rec_start}"
for text in labels:
    Path(f"{save_path}/cropped/{text}").mkdir(parents=True, exist_ok=True)
if args.save_raw_frames:
    Path(f"{save_path}/raw").mkdir(parents=True, exist_ok=True)
if args.save_overlay_frames:
    Path(f"{save_path}/overlay").mkdir(parents=True, exist_ok=True)

# Calculate current recording ID by subtracting number of directories with date-prefix
folders_dates = len([f for f in Path("insect-detect/data").glob("**/20*") if f.is_dir()])
folders_days = len([f for f in Path("insect-detect/data").glob("20*") if f.is_dir()])
rec_id = folders_dates - folders_days

# Define functions
def frame_norm(frame, bbox):
    """Convert relative bounding box coordinates (0-1) to pixel coordinates."""
    norm_vals = np.full(len(bbox), frame.shape[0])
    norm_vals[::2] = frame.shape[1]
    return (np.clip(np.array(bbox), 0, 1) * norm_vals).astype(int)

def make_bbox_square(bbox): # (5)!
    """Increase bbox size on both sides of the minimum dimension, or only on one side if localized at frame margin."""
    bbox_width = bbox[2] - bbox[0]
    bbox_height = bbox[3] - bbox[1]
    bbox_diff = (max(bbox_width, bbox_height) - min(bbox_width, bbox_height)) // 2
    if bbox_width < bbox_height:
        if bbox[0] - bbox_diff < 0:
            det_crop = frame[bbox[1]:bbox[3], 0:bbox[2] + (bbox_diff * 2 - bbox[0])]
        elif not args.four_k_resolution and bbox[2] + bbox_diff > 1920:
            det_crop = frame[bbox[1]:bbox[3], bbox[0] - (bbox_diff * 2 - (1920 - bbox[2])):1920]
        elif args.four_k_resolution and bbox[2] + bbox_diff > 3840:
            det_crop = frame[bbox[1]:bbox[3], bbox[0] - (bbox_diff * 2 - (3840 - bbox[2])):3840]
        else:
            det_crop = frame[bbox[1]:bbox[3], bbox[0] - bbox_diff:bbox[2] + bbox_diff]
    else:
        if bbox[1] - bbox_diff < 0:
            det_crop = frame[0:bbox[3] + (bbox_diff * 2 - bbox[1]), bbox[0]:bbox[2]]
        elif not args.four_k_resolution and bbox[3] + bbox_diff > 1080:
            det_crop = frame[bbox[1] - (bbox_diff * 2 - (1080 - bbox[3])):1080, bbox[0]:bbox[2]]
        elif args.four_k_resolution and bbox[3] + bbox_diff > 2160:
            det_crop = frame[bbox[1] - (bbox_diff * 2 - (2160 - bbox[3])):2160, bbox[0]:bbox[2]]
        else:
            det_crop = frame[bbox[1] - bbox_diff:bbox[3] + bbox_diff, bbox[0]:bbox[2]]
    return det_crop

def store_data(frame, tracks):
    """Save cropped detections (+ full HQ frames) to .jpg and tracker output to metadata .csv."""
    with open(f"{save_path}/metadata_{rec_start}.csv", "a", encoding="utf-8") as metadata_file:
        metadata = csv.DictWriter(metadata_file, fieldnames=
            ["rec_ID", "timestamp", "label", "confidence", "track_ID",
             "x_min", "y_min", "x_max", "y_max", "file_path"])
        if metadata_file.tell() == 0:
            metadata.writeheader() # write header only once

        # Save full raw HQ frame (e.g. for training data collection)
        if args.save_raw_frames:
            for track in tracks:
                if track == tracks[-1]:
                    timestamp = datetime.now().strftime("%Y%m%d_%H-%M-%S.%f")
                    raw_path = f"{save_path}/raw/{timestamp}_raw.jpg"
                    cv2.imwrite(raw_path, frame) # (6)!
                    #cv2.imwrite(raw_path, frame, [cv2.IMWRITE_JPEG_QUALITY, 70])

        for track in tracks:
            # Don't save cropped detections if tracking status == "NEW" or "LOST" or "REMOVED"
            if track.status.name == "TRACKED": # (7)!

                # Save detections cropped from HQ frame to .jpg
                bbox = frame_norm(frame, (track.srcImgDetection.xmin, track.srcImgDetection.ymin,
                                          track.srcImgDetection.xmax, track.srcImgDetection.ymax))
                if args.crop_bbox == "square":
                    det_crop = make_bbox_square(bbox)
                else:
                    det_crop = frame[bbox[1]:bbox[3], bbox[0]:bbox[2]]
                label = labels[track.srcImgDetection.label]
                timestamp = datetime.now().strftime("%Y%m%d_%H-%M-%S.%f")
                crop_path = f"{save_path}/cropped/{label}/{timestamp}_{track.id}_crop.jpg"
                cv2.imwrite(crop_path, det_crop)

                # Save corresponding metadata to .csv file for each cropped detection
                data = {
                    "rec_ID": rec_id,
                    "timestamp": timestamp,
                    "label": label,
                    "confidence": round(track.srcImgDetection.confidence, 2),
                    "track_ID": track.id,
                    "x_min": round(track.srcImgDetection.xmin, 4),
                    "y_min": round(track.srcImgDetection.ymin, 4),
                    "x_max": round(track.srcImgDetection.xmax, 4),
                    "y_max": round(track.srcImgDetection.ymax, 4),
                    "file_path": crop_path
                }
                metadata.writerow(data)
                metadata_file.flush() # write data immediately to .csv to avoid potential data loss

                # Save full HQ frame with overlay (bounding box, label, confidence, tracking ID) drawn on frame
                if args.save_overlay_frames:
                    # Text position, font size and thickness optimized for 1920x1080 px HQ frame size
                    if not args.four_k_resolution:
                        cv2.putText(frame, labels[track.srcImgDetection.label], (bbox[0], bbox[3] + 28),
                                    cv2.FONT_HERSHEY_SIMPLEX, 0.9, (255, 255, 255), 2)
                        cv2.putText(frame, f"{round(track.srcImgDetection.confidence, 2)}", (bbox[0], bbox[3] + 55),
                                    cv2.FONT_HERSHEY_SIMPLEX, 0.8, (255, 255, 255), 2)
                        cv2.putText(frame, f"ID:{track.id}", (bbox[0], bbox[3] + 92),
                                    cv2.FONT_HERSHEY_SIMPLEX, 1.1, (255, 255, 255), 2)
                        cv2.rectangle(frame, (bbox[0], bbox[1]), (bbox[2], bbox[3]), (0, 0, 255), 2)
                    # Text position, font size and thickness optimized for 3840x2160 px HQ frame size
                    else:
                        cv2.putText(frame, labels[track.srcImgDetection.label], (bbox[0], bbox[3] + 48),
                                    cv2.FONT_HERSHEY_SIMPLEX, 1.7, (255, 255, 255), 3)
                        cv2.putText(frame, f"{round(track.srcImgDetection.confidence, 2)}", (bbox[0], bbox[3] + 98),
                                    cv2.FONT_HERSHEY_SIMPLEX, 1.6, (255, 255, 255), 3)
                        cv2.putText(frame, f"ID:{track.id}", (bbox[0], bbox[3] + 164),
                                    cv2.FONT_HERSHEY_SIMPLEX, 2, (255, 255, 255), 3)
                        cv2.rectangle(frame, (bbox[0], bbox[1]), (bbox[2], bbox[3]), (0, 0, 255), 3)
                    if track == tracks[-1]:
                        timestamp = datetime.now().strftime("%Y%m%d_%H-%M-%S.%f")
                        overlay_path = f"{save_path}/overlay/{timestamp}_overlay.jpg"
                        cv2.imwrite(overlay_path, frame)
                        #cv2.imwrite(overlay_path, frame, [cv2.IMWRITE_JPEG_QUALITY, 70])

def record_log(): # (8)!
    """Write information about each recording interval to .csv file."""
    try:
        df_meta = pd.read_csv(f"{save_path}/metadata_{rec_start}.csv", encoding="utf-8")
        unique_ids = df_meta["track_ID"].nunique()
    except pd.errors.EmptyDataError:
        unique_ids = 0
    with open("insect-detect/data/record_log.csv", "a", encoding="utf-8") as log_rec_file:
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
            "record_time_min": round((time.monotonic() - start_time) / 60, 2),
            "num_crops": len(list(Path(f"{save_path}/cropped").glob("**/*.jpg"))),
            "num_IDs": unique_ids,
            "disk_free_gb": round(psutil.disk_usage("/").free / 1073741824, 1),
            "chargelevel_start": chargelevel_start,
            "chargelevel_end": chargelevel
        }
        log_rec.writerow(logs_rec)

def save_logs(): # (9)!
    """
    Write recording ID, time, RPi CPU + OAK chip temperature, RPi available memory (MB) +
    CPU utilization (%) and PiJuice battery info + temp to .csv file.
    """
    with open(f"insect-detect/data/{rec_start[:8]}/info_log_{rec_start[:8]}.csv", "a",
              encoding="utf-8") as log_info_file:
        log_info = csv.DictWriter(log_info_file, fieldnames=
            ["rec_ID", "timestamp", "temp_pi", "temp_oak", "pi_mem_available", "pi_cpu_used",
             "power_input", "charge_status", "charge_level", "temp_batt", "voltage_batt_mV"])
        if log_info_file.tell() == 0:
            log_info.writeheader()
        try:
            temp_oak = round(device.getChipTemperature().average)
        except RuntimeError:
            temp_oak = "NA"
        try:
            logs_info = {
                "rec_ID": rec_id,
                "timestamp": datetime.now().strftime("%Y%m%d_%H-%M-%S"),
                "temp_pi": round(CPUTemperature().temperature),
                "temp_oak": temp_oak,
                "pi_mem_available": round(psutil.virtual_memory().available / 1048576),
                "pi_cpu_used": psutil.cpu_percent(interval=None),
                "power_input": pijuice.status.GetStatus().get("data", {}).get("powerInput", "NA"),
                "charge_status": pijuice.status.GetStatus().get("data", {}).get("battery", "NA"),
                "charge_level": chargelevel,
                "temp_batt": pijuice.status.GetBatteryTemperature().get("data", "NA"),
                "voltage_batt_mV": pijuice.status.GetBatteryVoltage().get("data", "NA")
            }
        except IndexError:
            logs_info = {}
        log_info.writerow(logs_info)
        log_info_file.flush()

# Connect to OAK device and start pipeline in USB2 mode
with dai.Device(pipeline, maxUsbSpeed=dai.UsbSpeed.HIGH) as device:

    # Write RPi + OAK + battery info to .csv log file at specified interval
    if args.save_logs:
        logging.getLogger("apscheduler").setLevel(logging.WARNING)
        scheduler = BackgroundScheduler()
        scheduler.add_job(save_logs, "interval", seconds=30, id="log") # (10)!
        scheduler.start()

    # Create empty list to save charge level (if < 10) and set charge level
    lst_chargelevel = []
    chargelevel = chargelevel_start

    # Set recording time conditional on PiJuice battery charge level
    if chargelevel >= 70: # (11)!
        rec_time = 60 * 40
    elif 50 <= chargelevel < 70:
        rec_time = 60 * 30
    elif 30 <= chargelevel < 50:
        rec_time = 60 * 20
    elif 15 <= chargelevel < 30:
        rec_time = 60 * 10
    else:
        rec_time = 60 * 5

    # Write info on start of recording to log file
    logger.info(f"Rec ID: {rec_id} | Rec time: {int(rec_time / 60)} min | Charge level: {chargelevel}%")

    # Create output queues to get the frames and tracklets + detections from the outputs defined above
    q_frame = device.getOutputQueue(name="frame", maxSize=4, blocking=False)
    q_track = device.getOutputQueue(name="track", maxSize=4, blocking=False)

    # Set start time of recording
    start_time = time.monotonic()

    try:
        # Record until recording time is finished or charge level dropped below threshold for 10 times
        while time.monotonic() < start_time + rec_time and len(lst_chargelevel) < 10: # (12)!

            # Update charge level (return "99" if not readable and write to list if < 10)
            chargelevel = pijuice.status.GetChargeLevel().get("data", 99)
            if chargelevel < 10:
                lst_chargelevel.append(chargelevel)

            # Get synchronized HQ frames + tracker output (passthrough detections)
            if q_frame.has():
                frame = q_frame.get().getCvFrame()

                if q_track.has():
                    tracks = q_track.get().tracklets

                    # Save cropped detections (slower if saving additional HQ frames)
                    store_data(frame, tracks)

            # Wait for 1 second
            time.sleep(1) # (13)!

        # Write info on end of recording to log file and write record logs to .csv
        logger.info(f"Recording {rec_id} finished | Charge level: {chargelevel}%\n")
        record_log()

        # Enable charging of PiJuice battery if charge level is lower than threshold
        if chargelevel < 80:
            pijuice.config.SetChargingConfig({"charging_enabled": True})

        # Shutdown Raspberry Pi
        subprocess.run(["sudo", "shutdown", "-h", "now"], check=True) # (14)!

    # Write info on error during recording to log file and write record logs to .csv
    except Exception:
        logger.error(traceback.format_exc())
        logger.error(f"Error during recording {rec_id} | Charge level: {chargelevel}%\n")
        record_log()

        # Enable charging of PiJuice battery if charge level is lower than threshold
        if chargelevel < 80:
            pijuice.config.SetChargingConfig({"charging_enabled": True})

        # Shutdown Raspberry Pi
        subprocess.run(["sudo", "shutdown", "-h", "now"], check=True)

```

1.  With the logger set up, you can write any information you need (e.g. for
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
3.  Activate this option only if you have two batteries installed! Disable
    charging of the PiJuice battery if the charge level is higher than the
    specified threshold to extend battery life. The charge level is checked
    again at the end of the script to re-enable charging if the charge level
    dropped below a specified threshold.
4.  This script is run on the OAK device and will synchronize the output from
    the object tracker node (+ passthrough model detections) with the HQ frames
    by comparing their respective sequence numbers. The synchronized tracker
    output and HQ frame are then send to the host (RPi).
5.  This function will increase the bounding box size on both sides of the minimum
    dimension, or only on one side if the insect is localized at the frame margin.
    The cropped detections will thereby always have an aspect ratio of 1:1. This
    can improve classification model training and inference, as the images are
    not stretched during resizing and no distortion is added.
6.  Raw HQ frames will be saved every second if activated with `-raw`. To
    decrease the .jpg file size, you can save them with a higher JPEG
    compression (e.g. 70% quality instead of the default 95%) by commenting
    out this line and using the following line.
7.  A tracked object can have 4 possible statuses: `NEW`, `TRACKED`, `LOST` and
    `REMOVED`. It is highly recommended to save the cropped detections only when
    `tracking status == TRACKED`, but you could change this configuration here and
    e.g. write the `track.status.name` as additional column to the metadata .csv.
8.  This function will be called after a recording interval is finished, or if
    an error occurs during the recording and will write some info about the
    respective recording interval to `record_log.csv`.
9.  This function will be called if you are using the optional `-log` argument
    and will save the specified logging info to a .csv file.
10. You can change the
    [interval](https://apscheduler.readthedocs.io/en/3.x/modules/triggers/interval.html){target=_blank}
    at which logs will be written to the log .csv file in this line.
11. You can specify your own recording durations and charge level thresholds in
    this code section. The suggested values can provide an efficient recording
    behaviour if you are using the 12,000 mAh PiJuice battery and set up the
    [Wakeup Alarm](../pisetup/#pijuice-zero-configuration){target=_blank} for
    3-6 times per day. Depending on the number of Wakeups per day, as well as
    the season and sun exposure of the solar panel, it can make sense to
    increase or decrease the recording duration.
12. The recording will be stopped after the recording time is finished or if the
    charge level of the PiJuice battery drops below the specified threshold for
    over ten seconds. This avoids immediate stopping of the recording if the
    battery charge level is falsely returned < 10, which can happen sometimes.
13. In this line you can change the time interval with which the cropped
    detections will be saved to .jpg. This does not affect the detection model
    and object tracker speed, which are both run on-device even if no detections
    are saved. Using `-raw` or `-overlay` to additionally save the full HQ frames
    can significantly slow down the pipeline and inference speed.
14. If you are still in the testing phase, comment out the shutdown commands in
    this line and the last line by adding `#` in front of the line.

---

## Frame capture

If you want to only capture images, e.g. for training data collection, you can
use the following script to save HQ frames (e.g. 1920x1080 px) to .jpg at a
specified time interval. Optionally, the downscaled LQ frames (e.g. 320x320 px)
can be saved to .jpg additionally, e.g. to include them in the training data,
as the detection model will do inference on LQ frames (however it is recommended
downscale the annotated HQ images before training).

Run the script with:

``` bash
python3 insect-detect/frame_capture.py
```

??? info "Optional arguments"

    Add after `python3 insect-detect/frame_capture.py`, separated by space:

    - `-min` to set recording time in minutes (e.g. `-min 5` for 5 min
      recording time; default = 2)
    - `-4k` to save HQ frames in 4K resolution (3840x2160 px; default = 1080p)
    - `-lq` to additionally save downscaled LQ frames (e.g. 320x320 px)

Stop the script by pressing ++ctrl+c++ in the Terminal.

``` py title="frame_capture.py" hl_lines="28"
'''
Author:   Maximilian Sittinger (https://github.com/maxsitt)
License:  GNU GPLv3 (https://choosealicense.com/licenses/gpl-3.0/)

based on open source scripts available at https://github.com/luxonis
'''

import argparse
import time
from datetime import datetime
from pathlib import Path

import cv2
import depthai as dai

# Define optional arguments
parser = argparse.ArgumentParser()
parser.add_argument("-min", "--min_rec_time", type=int, choices=range(1, 721), default=2,
    help="set record time in minutes")
parser.add_argument("-4k", "--four_k_resolution", action="store_true",
    help="save HQ frames in 4K resolution; default = 1080p")
parser.add_argument("-lq", "--save_lq_frames", action="store_true",
    help="additionally save downscaled LQ frames")
args = parser.parse_args()

# Set capture frequency in seconds
# 'CAPTURE_FREQ = 0.8' (0.2 for 4K) saves ~58 frames per minute to .jpg (RPi Zero 2)
CAPTURE_FREQ = 0.8 # (1)!
if args.four_k_resolution:
    CAPTURE_FREQ = 0.2

# Create depthai pipeline
pipeline = dai.Pipeline()

# Create and configure camera node and define output(s)
cam_rgb = pipeline.create(dai.node.ColorCamera)
#cam_rgb.setImageOrientation(dai.CameraImageOrientation.ROTATE_180_DEG)
cam_rgb.setResolution(dai.ColorCameraProperties.SensorResolution.THE_4_K)
if not args.four_k_resolution:
    cam_rgb.setIspScale(1, 2) # downscale 4K to 1080p HQ frames (1920x1080 px)
cam_rgb.setFps(25) # frames per second available for focus/exposure
if args.save_lq_frames:
    cam_rgb.setPreviewSize(320, 320) # downscaled LQ frames
    cam_rgb.setPreviewKeepAspectRatio(False) # "squeeze" frames (16:9) to square (1:1)
    cam_rgb.setInterleaved(False) # planar layout

xout_rgb = pipeline.create(dai.node.XLinkOut)
xout_rgb.setStreamName("frame")
cam_rgb.video.link(xout_rgb.input) # HQ frame

if args.save_lq_frames:
    xout_lq = pipeline.create(dai.node.XLinkOut)
    xout_lq.setStreamName("frame_lq")
    cam_rgb.preview.link(xout_lq.input) # LQ frame

# Connect to OAK device and start pipeline in USB2 mode
with dai.Device(pipeline, maxUsbSpeed=dai.UsbSpeed.HIGH) as device:

    # Create output queue(s) to get the frames from the output(s) defined above
    q_frame = device.getOutputQueue(name="frame", maxSize=4, blocking=False)
    if args.save_lq_frames:
        q_frame_lq = device.getOutputQueue(name="frame_lq", maxSize=4, blocking=False)

    # Create folders to save the frames
    rec_start = datetime.now().strftime("%Y%m%d_%H-%M")
    save_path = f"insect-detect/frames/{rec_start[:8]}/{rec_start}"
    Path(f"{save_path}").mkdir(parents=True, exist_ok=True)
    if args.save_lq_frames:
        Path(f"{save_path}/LQ_frames").mkdir(parents=True, exist_ok=True)

    # Create start_time variable to set recording time
    start_time = time.monotonic()

    # Get recording time in min from optional argument (default: 2)
    rec_time = args.min_rec_time * 60
    print(f"Recording time: {args.min_rec_time} min")

    # Record until recording time is finished
    while time.monotonic() < start_time + rec_time:

        # Get HQ (+ LQ) frames and save to .jpg at specified time interval
        timestamp = datetime.now().strftime("%Y%m%d_%H-%M-%S.%f")
        hq_path = f"{save_path}/{timestamp}.jpg"
        hq_frame = q_frame.get().getCvFrame()
        cv2.imwrite(hq_path, hq_frame)

        if args.save_lq_frames:
            lq_path = f"{save_path}/LQ_frames/{timestamp}_LQ.jpg"
            lq_frame = q_frame_lq.get().getCvFrame()
            cv2.imwrite(lq_path, lq_frame)

        time.sleep(CAPTURE_FREQ)

# Print number and path of saved frames to console
frames_hq = len(list(Path(f"{save_path}").glob("*.jpg")))
if args.save_lq_frames:
    frames_lq = len(list(Path(f"{save_path}/LQ_frames").glob("*.jpg")))
    print(f"Saved {frames_hq} HQ and {frames_lq} LQ frames to {save_path}.")
else:
    print(f"Saved {frames_hq} HQ frames to {save_path}.") # (2)!

```

1.  You can increase the capture frequency in this line, e.g. if you want to
    only save a frame every 10 seconds, every minute etc. Keep in mind that
    the image processing will take some time and test different values until
    the frames are saved with your desired time interval.
2.  If you are running this script automatically, you can also write this
    info to a log file. Check the [monitoring script](#automated-monitoring-script)
    and copy the lines at the beginning to create a logger.

---

## Still capture

The following Python script enables the capture of still frames at the highest
possible resolution of the
[supported sensors](https://docs.luxonis.com/projects/hardware/en/latest/pages/articles/supported_sensors.html){target=_blank}
at a specified time interval. This will lead to a bigger field of view (FOV),
compared to the other scripts where the camera sensor is set to 4K or 1080p
resolution. You can find more information on sensor resolution and image types at the
[DepthAI API Docs](https://docs.luxonis.com/projects/api/en/latest/components/nodes/color_camera/){target=_blank}.

Run the script with:

``` bash
python3 insect-detect/still_capture.py
```

??? info "Optional argument"

    Add after `python3 insect-detect/still_capture.py`, separated by space:

    - `-min` to set recording time in minutes (e.g. `-min 5` for 5 min
      recording time; default = 2)

Stop the script by pressing ++ctrl+c++ in the Terminal.

``` py title="still_capture.py" hl_lines="23 31 35"
'''
Author:   Maximilian Sittinger (https://github.com/maxsitt)
License:  GNU GPLv3 (https://choosealicense.com/licenses/gpl-3.0/)

based on open source scripts available at https://github.com/luxonis
'''

import argparse
import time
from datetime import datetime
from pathlib import Path

import depthai as dai

# Define optional arguments
parser = argparse.ArgumentParser()
parser.add_argument("-min", "--min_rec_time", type=int, choices=range(1, 721), default=2,
    help="set record time in minutes")
args = parser.parse_args()

# Set capture frequency in seconds
# 'CAPTURE_FREQ = 1' saves ~57 still frames per minute to .jpg (RPi Zero 2)
CAPTURE_FREQ = 1 # (1)

# Create depthai pipeline
pipeline = dai.Pipeline()

# Create and configure camera node
cam_rgb = pipeline.create(dai.node.ColorCamera)
#cam_rgb.setImageOrientation(dai.CameraImageOrientation.ROTATE_180_DEG)
cam_rgb.setResolution(dai.ColorCameraProperties.SensorResolution.THE_12_MP) # (2)!
#cam_rgb.setResolution(dai.ColorCameraProperties.SensorResolution.THE_13_MP) # OAK-1 Lite (IMX214)
#cam_rgb.setResolution(dai.ColorCameraProperties.SensorResolution.THE_5312X6000) # OAK-1 MAX (IMX582)
cam_rgb.setNumFramesPool(2,2,2,2,2) # (3)!
cam_rgb.setFps(25) # frames per second available for focus/exposure # (4)

# Create and configure video encoder node and define input + output
still_enc = pipeline.create(dai.node.VideoEncoder) # (5)!
still_enc.setDefaultProfilePreset(1, dai.VideoEncoderProperties.Profile.MJPEG)
still_enc.setNumFramesPool(1)
cam_rgb.still.link(still_enc.input)

xout_still = pipeline.create(dai.node.XLinkOut)
xout_still.setStreamName("still")
still_enc.bitstream.link(xout_still.input)

# Create script node (to send capture still command)
script = pipeline.create(dai.node.Script)
script.setProcessor(dai.ProcessorType.LEON_CSS)

# Set script that will be run on-device (Luxonis OAK)
script.setScript('''
ctrl = CameraControl()
ctrl.setCaptureStill(True)

while True:
    node.io["capture_still"].send(ctrl)
''')

# Send script output to camera (capture still command)
script.outputs["capture_still"].link(cam_rgb.inputControl)

# Connect to OAK device and start pipeline in USB2 mode
with dai.Device(pipeline, maxUsbSpeed=dai.UsbSpeed.HIGH) as device:

    # Create output queue to get the encoded still frames from the output defined above
    q_still = device.getOutputQueue(name="still", maxSize=1, blocking=False)

    # Create folder to save the still frames
    rec_start = datetime.now().strftime("%Y%m%d_%H-%M")
    save_path = f"insect-detect/stills/{rec_start[:8]}/{rec_start}"
    Path(f"{save_path}").mkdir(parents=True, exist_ok=True)

    # Create start_time variable to set recording time
    start_time = time.monotonic()

    # Get recording time in min from optional argument (default: 2)
    rec_time = args.min_rec_time * 60
    print(f"Recording time: {args.min_rec_time} min")

    # Record until recording time is finished
    while time.monotonic() < start_time + rec_time:

        # Get encoded still frames and save to .jpg at specified time interval
        timestamp = datetime.now().strftime("%Y%m%d_%H-%M-%S.%f")
        enc_still = q_still.get().getData()
        with open(f"{save_path}/{timestamp}.jpg", "wb") as still_jpg:
            still_jpg.write(enc_still)

        time.sleep(CAPTURE_FREQ)

# Print number and path of saved still frames to console
frames_still = len(list(Path(f"{save_path}").glob("*.jpg")))
print(f"Saved {frames_still} still frames to {save_path}.") # (6)!

```

1.  You can increase the capture frequency in this line, e.g. if you want to
    only save a still frame every 10 seconds/every minute.
2.  `THE_12_MP` = 4032x3040 pixel, which is the maximum resolution of the IMX378
    camera sensor of the OAK-1. You can use other possible
    [sensor resolutions](https://docs.luxonis.com/projects/api/en/latest/references/python/#depthai.ColorCameraProperties.SensorResolution){target=_blank}
    depending on your device type.
3.  The maximum number of frames in all pools (raw, isp, preview, video, still)
    is set to 2, to avoid a potential out-of-memory error, especially when
    saving images with the OAK-1 MAX at 5312x6000 px.
4.  10 fps is currently the maximum framerate supported by the OAK-1 MAX at
    full resolution (will be automatically capped).
5.  More info about the
    [VideoEncoder node](https://docs.luxonis.com/projects/api/en/latest/components/nodes/video_encoder/){target=_blank}.
6.  If you are running this script automatically, you can also write this
    info to a log file. Check the [monitoring script](#automated-monitoring-script)
    and copy the lines at the beginning to create a logger.

---

## Video capture

With the following Python script you can save encoded HQ frames (1080p or 4K
resolution) with H.265 (HEVC) compression to a .mp4 video file. As there is no
encoding happening on the host (RPi), CPU and RAM usage is minimal, which makes
it possible to record 4K 30 fps video with almost no load on the Raspberry Pi.
As 4K 30 fps video can take up a lot of disk space, the remaining free disk
space is checked while recording and the recording is stopped if the free space
left drops below a specified threshold (e.g. 200 MB).

If you don't need the full 30 fps you can decrease the frame rate which will
lead to a smaller video file size (e.g. `-fps 20`).

Run the script with:

``` bash
python3 insect-detect/video_capture.py
```

??? info "Optional arguments"

    Add after `python3 insect-detect/video_capture.py`, separated by space:

    - `-min` to set recording time in minutes (e.g. `-min 5` for 5 min
      recording time; default = 2)
    - `-4k` to record video in 4K resolution (3840x2160 px) (default: 1080p)
    - `-fps` to set frame rate (frames per second) for video capture (default: 25)

Stop the script by pressing ++ctrl+c++ in the Terminal.

``` py title="video_capture.py" hl_lines="87"
'''
Author:   Maximilian Sittinger (https://github.com/maxsitt)
License:  GNU GPLv3 (https://choosealicense.com/licenses/gpl-3.0/)

based on open source scripts available at https://github.com/luxonis
'''

import argparse
import time
from datetime import datetime
from fractions import Fraction
from pathlib import Path

import av
import depthai as dai
import psutil

# Define optional arguments
parser = argparse.ArgumentParser()
parser.add_argument("-min", "--min_rec_time", type=int, choices=range(1, 61), default=2,
    help="set record time in minutes")
parser.add_argument("-4k", "--four_k_resolution", action="store_true",
    help="record video in 4K resolution (3840x2160 px); default = 1080p")
parser.add_argument("-fps", "--frames_per_second", type=int, choices=range(1, 31), default=25,
    help="set frame rate (frames per second) for video capture")
args = parser.parse_args()

# Get frame rate (frames per second) from optional argument (default: 25)
FPS = args.frames_per_second

# Create depthai pipeline
pipeline = dai.Pipeline()

# Create and configure camera node
cam_rgb = pipeline.create(dai.node.ColorCamera)
#cam_rgb.setImageOrientation(dai.CameraImageOrientation.ROTATE_180_DEG)
cam_rgb.setResolution(dai.ColorCameraProperties.SensorResolution.THE_4_K)
if not args.four_k_resolution:
    cam_rgb.setIspScale(1, 2) # downscale 4K to 1080p HQ frames (1920x1080 px)
cam_rgb.setFps(FPS) # frames per second available for focus/exposure

# Create and configure video encoder node and define input + output
video_enc = pipeline.create(dai.node.VideoEncoder) # (1)!
video_enc.setDefaultProfilePreset(FPS, dai.VideoEncoderProperties.Profile.H265_MAIN)
cam_rgb.video.link(video_enc.input)

xout_vid = pipeline.create(dai.node.XLinkOut)
xout_vid.setStreamName("video")
video_enc.bitstream.link(xout_vid.input)

# Connect to OAK device and start pipeline
with dai.Device(pipeline, maxUsbSpeed=dai.UsbSpeed.HIGH) as device:

    # Create output queue to get the encoded frames from the output defined above
    q_video = device.getOutputQueue(name="video", maxSize=30, blocking=True)

    # Create folder to save the videos
    rec_start = datetime.now().strftime("%Y%m%d")
    save_path = f"insect-detect/videos/{rec_start}"
    Path(f"{save_path}").mkdir(parents=True, exist_ok=True)

    # Create .mp4 container with H.265 (HEVC) compression
    timestamp = datetime.now().strftime("%Y%m%d_%H-%M-%S")
    RES = "1080p"
    if args.four_k_resolution:
        RES = "4K"
    with av.open(f"{save_path}/{timestamp}_{FPS}fps_{RES}_video.mp4", "w") as container:
        stream = container.add_stream("hevc", rate=FPS)
        stream.time_base = Fraction(1, 1000 * 1000)
        stream.width = 1920
        stream.height = 1080
        if args.four_k_resolution:
            stream.width = 3840
            stream.height = 2160

    # Create start_time variable to set recording time
    start_time = time.monotonic()

    # Get recording time in min from optional argument (default: 2)
    rec_time = args.min_rec_time * 60
    print(f"Recording time: {args.min_rec_time} min\n")

    # Get free disk space (MB)
    disk_free = round(psutil.disk_usage("/").free / 1048576)

    # Record until recording time is finished or free disk space drops below threshold
    while time.monotonic() < start_time + rec_time and disk_free > 200: # (2)!

        # Update free disk space (MB)
        disk_free = round(psutil.disk_usage("/").free / 1048576)

        # Get encoded video frames and save to packet
        enc_video = q_video.get().getData()
        packet = av.Packet(enc_video)
        packet.dts = int((time.monotonic() - start_time) * 1000 * 1000)
        packet.pts = int((time.monotonic() - start_time) * 1000 * 1000)

        # Mux packet into the .mp4 container
        container.mux_one(packet)

# Print duration, fps and path of saved video + free disk space to console
if args.four_k_resolution:
    print(f"\nSaved {args.min_rec_time} min 4K video with {args.frames_per_second} fps to {save_path}.")
else:
    print(f"\nSaved {args.min_rec_time} min 1080p video with {args.frames_per_second} fps to {save_path}.")
print(f"Free disk space left: {disk_free} MB") # (3)!

```

1.  More info about the
    [VideoEncoder node](https://docs.luxonis.com/projects/api/en/latest/components/nodes/video_encoder/){target=_blank}.
2.  Depending on the available disk space, it might make sense to change this
    threshold to a higher or lower value.
3.  If you are running this script automatically, you can also write this
    info to a log file. Check the [monitoring script](#automated-monitoring-script)
    and copy the lines at the beginning to create a logger.
