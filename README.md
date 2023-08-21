# Insect Detect Docs - DIY camera trap documentation

<img src="https://raw.githubusercontent.com/maxsitt/insect-detect-docs/main/docs/assets/logo.png" width="500">

[![DOI](https://zenodo.org/badge/580908850.svg)](https://zenodo.org/badge/latestdoi/580908850)
[![License: CC BY-SA 4.0](https://img.shields.io/badge/License-CC_BY--SA_4.0-lightgrey.svg)](https://creativecommons.org/licenses/by-sa/4.0/)

This repository contains the Markdown source files and assets (images, screenshots) of
the [**Insect Detect Docs**](https://maxsitt.github.io/insect-detect-docs/) 📑 website,
based on [Material for MkDocs](https://github.com/squidfunk/mkdocs-material).

The `PDF_templates` folder contains
[drilling templates](https://github.com/maxsitt/insect-detect-docs/tree/main/PDF_templates/drilling_templates)
that can be used while [building](https://maxsitt.github.io/insect-detect-docs/hardware/buildinstructions_enclosure/)
the DIY camera trap and templates for the small and big
[flower platform](https://github.com/maxsitt/insect-detect-docs/tree/main/PDF_templates/flower_platform)
that is used as visual attractant and background for the automated insect monitoring.

The Insect Detect DIY camera trap system is composed of low-cost off-the-shelf hardware components
([Raspberry Pi Zero 2 W](https://www.raspberrypi.com/products/raspberry-pi-zero-2-w/),
[Luxonis OAK-1](https://docs.luxonis.com/projects/hardware/en/latest/pages/BW1093.html),
[PiJuice Zero pHAT](https://uk.pi-supply.com/products/pijuice-zero)), combined with
open source software and can be easily assembled and set up with the
[provided instructions](https://maxsitt.github.io/insect-detect-docs/).

<img src="https://raw.githubusercontent.com/maxsitt/insect-detect-docs/main/docs/hardware/assets/images/insectdetect_diy_cameratrap.jpg" width="400">

---

## Hardware assembly

In the [**Hardware**](https://maxsitt.github.io/insect-detect-docs/hardware/)
section of the documentaton website, you will find a list with all required
[components](https://maxsitt.github.io/insect-detect-docs/hardware/components/)
and detailed step-by-step
[instructions](https://maxsitt.github.io/insect-detect-docs/hardware/buildinstructions_enclosure/)
on how to build and assemble the DIY camera trap system. Only
some standard tools are necessary, which are listed in the Hardware
[overview](https://maxsitt.github.io/insect-detect-docs/hardware/buildinstructions_overview/).

<img src="https://raw.githubusercontent.com/maxsitt/insect-detect-docs/main/docs/hardware/assets/images/full_setup_overview.jpg" width="500">

---

## Software setup

In the [**Software**](https://maxsitt.github.io/insect-detect-docs/software/)
section of the documentaton website, all steps to get the camera trap up and
running are explained. You will start with installing the necessary software
to your [local PC](https://maxsitt.github.io/insect-detect-docs/software/localsetup/),
to communicate with the Raspberry Pi Zero 2 W. The next steps will guide you
through the [Raspberry Pi configuration](https://maxsitt.github.io/insect-detect-docs/software/pisetup/),
after which everything is ready to use the Python scripts from the
[`insect-detect`](https://github.com/maxsitt/insect-detect) GitHub repo
for testing and deploying the camera trap. Details on various options to
adapt the scripts to different use cases can be found in the
[Programming](https://maxsitt.github.io/insect-detect-docs/software/programming/)
part of the Software section.

<img src="https://raw.githubusercontent.com/maxsitt/insect-detect-docs/main/docs/software/assets/images/vscode_raspberry_ssh_x11.gif" width="800">

---

## Model training

The **Model Training** section will show you tools to
[annotate](https://maxsitt.github.io/insect-detect-docs/modeltraining/annotation/)
your own images and use these to train your custom YOLOv5, YOLOv6, YOLOv7 or YOLOv8
[object detection model](https://maxsitt.github.io/insect-detect-docs/modeltraining/train_detection/)
that can be deployed on the OAK-1 camera.

To classify the cropped insect images, you can train a custom YOLOv5-cls
[image classification model](https://maxsitt.github.io/insect-detect-docs/modeltraining/train_classification/)
in the next step that can be run on your local PC (no GPU necessary). All of the
model training can be done in [Google Colab](https://colab.research.google.com/),
where you will have access to a free cloud GPU for fast training without special
hardware requirements. The model training notebooks are available in the
[`insect-detect-ml`](https://github.com/maxsitt/insect-detect-ml) GitHub repo.

<img src="https://raw.githubusercontent.com/maxsitt/insect-detect-docs/main/docs/modeltraining/assets/images/roboflow_annotate.jpg" width="800">

---

## Deployment

The **Deployment** section will give you details on each step of the processing pipeline,
from [on-device detection](https://maxsitt.github.io/insect-detect-docs/deployment/detection/) and
tracking, to [classification](https://maxsitt.github.io/insect-detect-docs/deployment/classification/)
of the cropped insect images on your local PC and subsequent automated post-processing and
[analysis](https://maxsitt.github.io/insect-detect-docs/deployment/analysis/) of the combined results.
The Python script for [classification](https://github.com/maxsitt/yolov5/blob/master/classify/predict.py)
of the captured insect images is available in the [custom YOLOv5](https://github.com/maxsitt/yolov5) fork.
Python scripts for post-processing and [analysis](https://github.com/maxsitt/insect-detect-ml/blob/main/csv_analysis.py)
of the results are available in the [`insect-detect-ml`](https://github.com/maxsitt/insect-detect-ml) GitHub repo.

<img src="https://raw.githubusercontent.com/maxsitt/insect-detect-docs/main/docs/deployment/assets/images/hq_frame_sync_1080p.gif" width="800">

---

## License

The documentation website and its content is licensed under the Creative Commons Attribution-ShareAlike 4.0
International License ([CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/)).

## Citation

You can cite this repository as:

```
Sittinger, M. (2023). Insect Detect Docs - Documentation website for the
Insect Detect DIY camera trap system (v1.1). Zenodo. https://doi.org/10.5281/zenodo.7503371
```
