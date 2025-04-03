# Insect Detect Docs - DIY camera trap documentation

<img src="https://raw.githubusercontent.com/maxsitt/insect-detect-docs/main/docs/assets/logo.png" width="540">

[![DOI PLOS ONE](https://img.shields.io/badge/PLOS%20ONE-10.1371%2Fjournal.pone.0295474-BD3094)](https://doi.org/10.1371/journal.pone.0295474)
[![License: CC BY-SA 4.0](https://img.shields.io/badge/License-CC_BY--SA_4.0-lightgrey.svg)](https://creativecommons.org/licenses/by-sa/4.0/)
[![DOI Zenodo](https://zenodo.org/badge/580908850.svg)](https://zenodo.org/badge/latestdoi/580908850)

This repository contains the Markdown source files and assets (images, screenshots) of
the [**Insect Detect Docs**](https://maxsitt.github.io/insect-detect-docs/) 📑 website,
based on [Material for MkDocs](https://github.com/squidfunk/mkdocs-material).

The `PDF_templates` folder contains
[drilling templates](https://github.com/maxsitt/insect-detect-docs/tree/main/PDF_templates/drilling_templates/version_2024)
that can be used while [building](https://maxsitt.github.io/insect-detect-docs/hardware/2024_buildinstructions_enclosures/)
the DIY camera trap.

The Insect Detect DIY camera trap system is composed of low-cost off-the-shelf hardware components
([Raspberry Pi Zero 2 W](https://www.raspberrypi.com/products/raspberry-pi-zero-2-w/),
[Luxonis OAK-1](https://docs.luxonis.com/hardware/products/OAK-1),
[Witty Pi 4 L3V7](https://www.uugear.com/product/witty-pi-4-l3v7/) or
[PiJuice Zero pHAT](https://uk.pi-supply.com/products/pijuice-zero)), combined with
open source software and can be easily assembled and set up with the
[provided instructions](https://maxsitt.github.io/insect-detect-docs/).

<img src="https://raw.githubusercontent.com/maxsitt/insect-detect-docs/main/docs/hardware/assets/images/2024_mount_camtrap_platform.jpg" width="400">

---

## Hardware assembly

In the [**Hardware**](https://maxsitt.github.io/insect-detect-docs/hardware/)
section of the documentaton website, you will find a list with all required
[components](https://maxsitt.github.io/insect-detect-docs/hardware/2024_components/)
and detailed step-by-step
[instructions](https://maxsitt.github.io/insect-detect-docs/hardware/2024_buildinstructions_enclosures/)
on how to build and assemble the DIY camera trap system. Only
some standard tools are necessary, which are listed in the Hardware
[overview](https://maxsitt.github.io/insect-detect-docs/hardware/2024_buildinstructions_overview/).

<img src="https://raw.githubusercontent.com/maxsitt/insect-detect-docs/main/docs/hardware/assets/images/2024_enclosures_connected.jpg" width="500">

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
for testing and deploying the camera trap. More information about the specific
scripts and how to run them can be found in
[Usage](https://maxsitt.github.io/insect-detect-docs/software/usage/).

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
from on-device [detection](https://maxsitt.github.io/insect-detect-docs/deployment/detection/) and
tracking, to [classification](https://maxsitt.github.io/insect-detect-docs/deployment/classification/)
of the cropped insect images on your local PC and subsequent metadata
[post-processing](https://maxsitt.github.io/insect-detect-docs/deployment/post-processing/) of the combined results.
The Python script for [classification](https://github.com/maxsitt/yolov5/blob/master/classify/predict.py)
of the captured insect images is available in the custom [`yolov5`](https://github.com/maxsitt/yolov5) fork.
A Python script for metadata [post-processing](https://github.com/maxsitt/insect-detect-ml/blob/main/process_metadata.py)
is available in the [`insect-detect-ml`](https://github.com/maxsitt/insect-detect-ml) GitHub repo.

<img src="https://raw.githubusercontent.com/maxsitt/insect-detect-docs/main/docs/deployment/assets/images/hq_sync_pipeline.png" width="800">

---

## License

This repository is licensed under the terms of the Creative Commons Attribution-ShareAlike 4.0
International License ([CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/)).

## Citation

If you use resources from this repository, please cite our paper:

```
Sittinger M, Uhler J, Pink M, Herz A (2024) Insect detect: An open-source DIY camera trap for automated insect monitoring. PLOS ONE 19(4): e0295474. https://doi.org/10.1371/journal.pone.0295474
```
