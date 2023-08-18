# Deployment: Classification

The **Insect Detect** DIY camera trap for automated insect monitoring will
yield cropped detections of individual insects saved as .jpg files and relevant
[metadata](detection.md#metadata-csv){target=_blank} saved to .csv for each
recording interval when using the provided script for
[automated monitoring](../software/programming.md#automated-monitoring-script){target=_blank}.

The recommended [processing pipeline](detection.md#processing-pipeline){target=_blank}
uses a [YOLOv5n](../index.md#detection-models){target=_blank} detection model with
only one generic class ("insect"). The low input resolution enables a high inference
speed, which is necessary to reliably track moving/flying insects. Images of the
detected and tracked insects are then cropped from synchronized HQ frames, which
increases classification accuracy due to the higher resolution and less background noise.

The saved insect images are classified in a subsequent step on your local PC, by
using a [YOLOv5s-cls](../index.md#classification-model){target=_blank}
image classification model exported to
[ONNX format](https://github.com/ultralytics/yolov5/issues/251){target=_blank}
for faster CPU inference.

---

## Installation

If you followed the steps in [Local Setup](../software/localsetup.md#python){target=_blank},
you already have [Python](https://www.python.org/){target=_blank} installed on your computer.

- Create the new folder `C:\Users\<username>\YOLOv5-cls`.
- [Download](https://github.com/maxsitt/yolov5/archive/refs/heads/master.zip){target=_blank}
  the custom YOLOv5 repo and extract it to the `YOLOv5-cls` folder.
- Open a new Terminal (PowerShell) and navigate to the `YOLOv5-cls` folder:

    ``` powershell
    cd YOLOv5-cls
    ```

- Update [`pip`](https://pypi.org/project/pip/){target=_blank}:

    ``` powershell
    python.exe -m pip install --upgrade pip
    ```

- Install the required packages by running:

    ``` powershell
    python.exe -m pip install -r yolov5-master/requirements.txt
    ```

??? info "Set up an isolated Python environment"

    You can [create a virtual environment](https://virtualenv.pypa.io/en/latest/user_guide.html){target=_blank}
    before installing the required packages for YOLOv5 to avoid version and dependency
    conflicts of the packages, especially if you are working on different Python projects.

    - Install [`virtualenv`](https://virtualenv.pypa.io/en/latest/){target=_blank}:

        ``` powershell
        python.exe -m pip install virtualenv
        ```

    - Navigate to the `YOLOv5-cls` folder:

        ``` powershell
        cd YOLOv5-cls
        ```

    - Create a new Python environment (folder in your current directory):

        ``` powershell
        virtualenv --python python3.11.4 env_yolov5
        ```

        If you are using a different Python version, change it accordingly.
    - Activate the environment by running:

        ``` powershell
        .\env_yolov5\Scripts\activate
        ```

    - Now all packages will be installed only in the virtual environment,
      which you have to activate everytime you want to run a script that
      depends on these packages.
    - You can deactivate the virtual environment by running:

        ``` powershell
        deactivate
        ```

---

## Run image classification

We will use the modified
[`classify/predict.py`](https://github.com/maxsitt/yolov5/blob/master/classify/predict.py){target=_blank}
script from the [custom YOLOv5](https://github.com/maxsitt/yolov5){target=_blank}
repo together with the provided image classification model to classify all insect
images in the `data` folder (camera trap output) and write the results to the
merged metadata .csv files.

- [Download](https://github.com/maxsitt/insect-detect-ml/archive/refs/heads/main.zip){target=_blank}
  the [`insect-detect-ml`](https://github.com/maxsitt/insect-detect-ml){target=_blank}
  repo and extract it to the `YOLOv5-cls` folder.
- Copy your `data` folder, [saved](../software/localsetup.md#diskinternals-linuxreader){target=_blank}
  from the Raspberry Pi's SD card to the `YOLOv5-cls` folder. Make sure that
  **only cropped detections** are present! If you additionally saved full
  HQ frames, delete them before running the classification script.
- Navigate to the `YOLOv5-cls` folder and start the classification script by running:

    ``` powershell
    python.exe yolov5-master\classify\predict.py --name camtrap1 --source data/**/ --weights insect-detect-ml/yolov5s-cls_128.onnx --img 128 --sort-top1 --concat-csv
    ```

    Change the `--name` of your prediction run accordingly, e.g. by including
    information about the camera trap location and collection date of the images.

??? info "Optional arguments"

    - `--sort-top1` to sort classified images to folders with predicted top1
      class as folder name and do not write results on to image as text (which
      is the default configuration)
    - `--concat-csv` to concatenate all metadata .csv files and
      append classification results to new columns
    - `--new-csv` to create a new .csv file with classification results,
      e.g. if no metadata .csv files are available
    - `--save-txt` to save the classification results to individual .txt files
      for each image

All results are saved to `yolov5-master\runs\predict-cls\{name}`. Use the
`results\{name}_metadata_classified.csv` for [analysis](analysis.md){target=_blank}
and post-processing in the last step.

If you used `--sort-top1` as optional argument, you will find the classified images
sorted to folders with the predicted top1 class as folder name in `top1_classes`.
This allows for a quick identification of edge cases and these images can be used to
[retrain](../modeltraining/train_classification.md){target=_blank} your classification model.

The `{name}_metadata_classified.csv` still contains multiple rows for each tracked insect
(= `track_ID`). In the last [processing step](analysis.md){target=_blank}, you will use the
[`csv_analysis.py`](https://github.com/maxsitt/insect-detect-ml/blob/main/csv_analysis.py){target=_blank}
script to automatically post-process the metadata .csv file and calculate the respective class
with the overall highest probability for each tracked insect. This will yield the final
.csv file, in which each row corresponds to an individual tracked insect.
