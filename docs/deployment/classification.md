# Deployment: Classification

The recommended [processing pipeline](detection.md#processing-pipeline){target=_blank}
uses a [detection model](../index.md#detection-models){target=_blank} with only
one generic class ("insect"). The low input resolution enables a high inference
speed, which is necessary to reliably track moving/flying insects. Images of the
detected and tracked insects are cropped from synchronized HQ frames in real time.

By using the provided
[recording script](https://github.com/maxsitt/insect-detect/blob/main/yolo_tracker_save_hqsync.py){target=_blank},
for automated monitoring, cropped detections of individual insects are saved as
.jpg files and relevant [metadata](detection.md#metadata-csv){target=_blank}
is saved to .csv for each recording session. The insect images can be classified
in a subsequent step on your local PC, by using a
[classification model](../index.md#classification-model){target=_blank} exported to
[ONNX format](https://github.com/ultralytics/yolov5/issues/251){target=_blank}
for faster CPU inference. The classification results are added to the merged metadata
.csv files for [post-processing](post-processing.md){target=_blank} in the last step.

---

## Installation

If you followed the steps in [Local Setup](../software/localsetup.md#python){target=_blank},
you already have [Python](https://www.python.org/){target=_blank} installed on your computer.

???+ info "Recommended: Set up an isolated Python environment"

     You can [create a virtual environment](https://virtualenv.pypa.io/en/latest/user_guide.html){target=_blank}
     before installing the required packages for YOLOv5 to avoid version and dependency
     conflicts of the packages, especially if you are working on different Python projects.

     - Install [`virtualenv`](https://virtualenv.pypa.io/en/latest/){target=_blank}:

         ``` powershell
         py -m pip install virtualenv
         ```

     - Navigate to the `YOLOv5-cls` folder:

         ``` powershell
         cd YOLOv5-cls
         ```

     - Create a new Python environment (folder in your current directory):

         ``` powershell
         py -m virtualenv env_yolov5
         ```

     - Activate the environment by running:

         ``` powershell
         .\env_yolov5\Scripts\activate
         ```

         If you cannot run the activate script because of your Execution Policy
         settings, open a new PowerShell as administrator and run the following:

         ``` powershell
         Set-ExecutionPolicy RemoteSigned
         ```

         You can find more info [here](https://virtualenv.pypa.io/en/latest/user_guide.html#activators){target=_blank}
         and [here](https://stackoverflow.com/questions/18713086/virtualenv-wont-activate-on-windows){target=_blank}.

     - Now all packages will be installed only in the virtual environment,
       which you have to activate everytime you want to run a script that
       depends on these packages.
     - You can deactivate the virtual environment by running:

         ``` powershell
         deactivate
         ```

- Create the new folder `C:\Users\<username>\YOLOv5-cls`.
- [Download](https://github.com/maxsitt/yolov5/archive/refs/heads/master.zip){target=_blank}
  the custom [`yolov5`](https://github.com/maxsitt/yolov5){target=_blank} fork
  and extract it to the `YOLOv5-cls` folder.
- Open a new Terminal (PowerShell) and navigate to the `YOLOv5-cls` folder:

    ``` powershell
    cd YOLOv5-cls
    ```

- Update [`pip`](https://pypi.org/project/pip/){target=_blank}:

    ``` powershell
    py -m pip install --upgrade pip
    ```

- Install the required packages by running:

    ``` powershell
    py -m pip install -r yolov5-master/requirements.txt
    ```

    !!! tip ""

        If you are running the image classification on a computer with CUDA-enabled
        GPU, please edit the `requirements.txt` file and change `onnxruntime` to
        `onnxruntime-gpu` before installing the packages.

---

## Run image classification

We will use the modified
[`classify/predict.py`](https://github.com/maxsitt/yolov5/blob/master/classify/predict.py){target=_blank}
script from the custom [`yolov5`](https://github.com/maxsitt/yolov5){target=_blank} repo
together with the provided [classification model](../index.md#classification-model){target=_blank}
to classify all insect images in the `insect-detect/data` folder (camera trap output)
and add the prediction results to the merged metadata .csv files.

- [Download](https://github.com/maxsitt/insect-detect-ml/archive/refs/heads/main.zip){target=_blank}
  the [`insect-detect-ml`](https://github.com/maxsitt/insect-detect-ml){target=_blank}
  repo and extract it to the `YOLOv5-cls` folder.
- Copy your `insect-detect/data` folder,
  [saved](../software/localsetup.md#diskinternals-linux-reader){target=_blank}
  from the Raspberry Pi's SD card to the `YOLOv5-cls` folder. Make sure that
  **only cropped detections** are present! If you additionally saved full
  HQ frames, delete them before running the classification script.
- Navigate to the `YOLOv5-cls` folder and start the classification script by running:

    ``` powershell
    py yolov5-master/classify/predict.py --name camtrap1 --source "insect-detect/data/**/" --weights "insect-detect-ml-main/models/efficientnet-b0_imgsz128.onnx" --img 128 --sort-top1 --sort-prob --concat-csv --crops-only
    ```

    !!! tip ""

        Change the `--name` of your prediction run accordingly, e.g. by including
        information about the camera trap location and collection date of the images.
        Instead of copying the `insect-detect/data` folder to the `YOLOv5-cls`
        folder, you can also insert its full path after `--source`.

    ??? info "Optional arguments"

        - `--sort-top1` sort images to folders with predicted top1 class as folder name
          and do not write results on to image as text (which is the default configuration)
        - `--sort-prob` sort images first by probability and then by top1 class
          (requires `--sort-top1`)
        - `--concat-csv` concatenate all metadata .csv files and
          append classification results to new columns
        - `--crops-only` only process images with `crop` in filename

    ??? bug "Image Not Found"

        While running the classification script, in some cases
        you might run into the following error:

        ``` bash
        AssertionError: Image Not Found <image path>
        ```

        This error can be caused by corrupt .jpg images, that are rarely generated during image capture. Run the
        [`process_images.py`](https://github.com/maxsitt/insect-detect-ml/blob/main/process_images.py){target=_blank}
        script to find corrupt images in the data folder and move them to a new folder with:

        ``` powershell
        py insect-detect-ml-main/process_images.py -source "insect-detect/data"
        ```

        After removing the corrupt .jpg images, the classification
        script should now run without throwing an error.

All results are saved to `yolov5-master/runs/predict-cls/{name}`. The
`*metadata_classified.csv` still contains multiple rows for each
tracked insect (= `track_ID`). During [post-processing](post-processing.md){target=_blank}
in the last step, the respective class with the highest weighted probability
is calculated for each tracked insect. This will create the final .csv file,
in which each row corresponds to an individual tracked insect.

!!! tip ""

    If you used `--sort-top1` as optional argument, the insect images are sorted
    to folders with the predicted top1 class as folder name. This allows for a
    quick identification of edge cases and these images can be used to
    [retrain](../modeltraining/train_classification.md){target=_blank} your
    classification model. By using `--sort-prob` additionally, the images are
    sorted by top1 probability first and then by top1 class. This can give you
    more information about cases where the model is unsure about its prediction.
    Adding images with a low probability to your training dataset can increase
    classification accuracy over time.
