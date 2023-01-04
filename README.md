# Insect Detect Docs

<img src="https://raw.githubusercontent.com/maxsitt/insect-detect-docs/main/docs/assets/logo.png" width="500">

[![DOI](https://zenodo.org/badge/580908850.svg)](https://zenodo.org/badge/latestdoi/580908850)
[![License badge](https://img.shields.io/badge/license-CC%20BY--SA%204.0-red)](https://creativecommons.org/licenses/by-sa/4.0/)

This repository contains the Markdown source files and assets (images, screenshots)
of the [Insect Detect Docs](https://maxsitt.github.io/insect-detect-docs/) website.
In the `PDF_templates` folder you will find all drilling templates, that can be used
while [building](https://maxsitt.github.io/insect-detect-docs/hardware/buildinstructions_enclosure/)
the DIY camera trap. Also templates for the small and big flower platform that is
used as attractant and background for the insect detection & recording can be found
in this folder.

The camera trap system is composed of low-cost off-the-shelf hardware components
([Raspberry Pi Zero 2 W](https://www.raspberrypi.com/products/raspberry-pi-zero-2-w/),
[Luxonis OAK-1](https://docs.luxonis.com/projects/hardware/en/latest/pages/BW1093.html),
[PiJuice Zero pHAT](https://uk.pi-supply.com/products/pijuice-zero)), combined with
open source software and can be easily assembled and set up with the
[provided instructions](https://maxsitt.github.io/insect-detect-docs/).

<img src="https://raw.githubusercontent.com/maxsitt/insect-detect-docs/main/docs/hardware/assets/images/insectdetect_diy_cameratrap.jpg" width="400">

## Hardware assembly

In the [**Hardware**](https://maxsitt.github.io/insect-detect-docs/hardware/)
section of the documentaton website, you will find a list with all required
[components](https://maxsitt.github.io/insect-detect-docs/hardware/components/)
and detailed [instructions](https://maxsitt.github.io/insect-detect-docs/hardware/buildinstructions_enclosure/)
on how to build and assemble the camera trap system. Only some standard
tools are necessary, which are listed in the Hardware
[overview](https://maxsitt.github.io/insect-detect-docs/hardware/buildinstructions_overview/).

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

## Model training

The **Model Training** section will show you tools to
[annotate](https://maxsitt.github.io/insect-detect-docs/modeltraining/annotation/)
your own images and use these to [train](https://maxsitt.github.io/insect-detect-docs/modeltraining/yolov5/)
your custom YOLOv5 object detection model that can be deployed on the OAK-1 camera.
To classify the cropped insect images, you can also train a custom classification model
in the next step that can be run on your local PC (no GPU necessary). All of the
model training can be done in Google Colab, where you will have access to a free
cloud GPU for fast training. The model training notebooks are available in the
[`insect-detect-ml`](https://github.com/maxsitt/insect-detect-ml) GitHub repo.

## Deployment

The **Deployment** section will give you details on each step of the processing pipeline,
from [on-device detection](https://maxsitt.github.io/insect-detect-docs/deployment/detection/) and
tracking, to [classification](https://maxsitt.github.io/insect-detect-docs/deployment/classification/)
of the cropped insect images on your local PC and subsequent automated
[analysis](https://maxsitt.github.io/insect-detect-docs/deployment/analysis/).
The Python scripts for classification and analysis are available in the
[`insect-detect-ml`](https://github.com/maxsitt/insect-detect-ml) GitHub repo.

## License

The documentation website and its content is licensed under the Creative Commons Attribution-ShareAlike 4.0
International License ([CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/)).

## Citation

You can cite this repository as:

```
Sittinger, M. (2023). Insect Detect Docs - Documentation website for the
Insect Detect DIY camera trap system. Zenodo. https://doi.org/10.5281/zenodo.7503371
```
