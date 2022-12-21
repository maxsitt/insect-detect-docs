# Deployment: Classification

The proposed DIY camera trap system for automated insect monitoring will yield
cropped detections of individual insects saved as .jpg files and relevant
metadata saved to .csv for each recording event when using the provided script
for [automated monitoring](../software/programming.md#automated-monitoring-script).
The recommended [processing pipeline](detection.md) uses a YOLOv5s detection
model with only one generic class ("insect"), as the low input resolution would
result in a low classification accuracy in most cases. As the detections are
cropped from HQ frames, their resolution is high enough to enable good
classification results in a subsequent step on your local PC, using a
[YOLOv5-cls](https://github.com/ultralytics/yolov5/pull/8956) classification
model [exported to ONNX format](https://github.com/ultralytics/yolov5/issues/251)
for faster CPU inference.

---

## Local setup (Windows)

To be able to run the classification and subsequent [analysis](analysis.md)
Python scripts on your PC,
[install Python 3.11](https://www.python.org/downloads/windows/) to your PC. If
you followed the steps in the [Software](../software/localsetup.md) section,
you already have VS Code installed together with the
[Python](https://bit.ly/2Zm3Ypq) extension. Activate Python in VS Code by using
the `Python: Select Interpreter` command. You can find more information on
[Python in VS Code](https://code.visualstudio.com/docs/languages/python) and a
[Getting started tutorial](https://code.visualstudio.com/docs/python/python-tutorial)
at the official VS Code website.

After VS Code and Python are installed on your system, download the
[YOLOv5 repository](https://github.com/ultralytics/yolov5) from GitHub as .zip
file or `git clone` it, if you have [git](https://git-scm.com/) installed.

![YOLOv5 repository download](assets/images/yolov5_download.png){ width="800" }

Extract the `yolov5-master.zip` file to a folder of your choice, e.g.
`C:\Users\user\YOLOv5-cls`.

Open a new local Terminal (PowerShell) in VS Code, navigate to the folder into
which you extracted the YOLOv5 repository and install the required Python
packages for YOLOv5 by running:

``` powershell
python.exe -m pip install -r yolov5-master/requirements.txt
```

![YOLOv5 install requirements](assets/images/yolov5_requirements.png){ width="800" }

---

## Run local classification

To be able to use the direct output (.jpg images and .csv metadata) from the
[automated monitoring script](../software/programming.md#automated-monitoring-script),
download the provided `predict_mod.py` script from the
[`insect-detect-ml` GitHub repo](https://github.com/maxsitt/insect-detect-ml) to
classify all cropped insect images in your `data` folder and write the
classification results to a new .csv file and/or append them to the merged
metadata .csv files.

- Copy the `predict_mod.py` script to `YOLOv5-cls\yolov5-master\classify`.
- Copy your `data` folder, saved from the Raspberry Pi's SD card (e.g. with the
  [DiskInternals LinuxReader](../software/localsetup.md#diskinternals-linuxreader))
  into the `YOLOv5-cls` folder. Please make sure that **only cropped detections**
  are present, if you additionally saved full HQ frames (with `-raw` or
  `-overlay`), save them to a different directory and delete them before
  running the classification script.
- Download the provided YOLOv5-cls classification model (ONNX format) from the
  [`insect-detect-ml` GitHub repo](https://github.com/maxsitt/insect-detect-ml)
  or use your own model and copy it into the `YOLOv5-cls` folder.

Open the Terminal (PowerShell) in VS Code, navigate to the `YOLOv5-cls` folder
and start the classification script by running:

``` powershell
python.exe yolov5-master\classify\predict_mod.py --name data_classified --weights yolov5s-cls_128.onnx --source data/**/ --img 128 --concat-csv
```

![YOLOv5 run classification](assets/images/yolov5_classify_command.png){ width="800" }

!!! info "Optional arguments"

    - `--concat-csv` to concatenate metadata .csv files and append
      classification results to new columns
    - `--new-csv` to create a new .csv file with classification results, e.g.
      if no metadata .csv files are available
    - `--sort-top1` to sort classified images to folders with predicted top1
      class as folder name and do not write results on to image as text (which
      is the default configuration)
    - `--save-txt` to save the classification results to individual .txt files
      for each image

All metadata .csv files in the `data` folder will be concatenated,
classification results (top3 classes + probabilities) written to new columns
and saved as single `metadata_classified_{timestamp}.csv` file in the
`YOLOv5-cls` folder. If you used `--sort-top1` as optional argument, you will
find the sorted images in the directory
`yolov5-master\runs\predict-cls\data_classified\top1_classes`. This allows for
a quick identification of edge cases (e.g. insect taxa that were classified to
the wrong class) and these images can be used to
[retrain your classification model](../modeltraining/yolov5.md).

The classified metadata .csv file still contains multiple rows for each tracked
insect (= track_ID). In the [last processing step](analysis.md), we will use a
Python script to automatically analyze the classified metadata .csv file and
calculate the respective class with the overall highest probability for each
tracked insect. This step will yield the final .csv file in which each row
corresponds to an individual tracked insect.
