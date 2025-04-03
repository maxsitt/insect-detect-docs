# Object detection model training

[YOLO](https://pjreddie.com/darknet/yolo/){target=_blank} (You Only Look Once)
([Redmon et al., 2016](https://doi.org/10.48550/arXiv.1506.02640){target=_blank})
is the first object detection model which combines object detection (bounding
box prediction) and classification (associated class probabilities) into a
single neural network. This new approach makes the model very fast and enables
its use for real-time inference, even on resource-constrained hardware.
After the first release, several new YOLO versions improved the initial model
and formed the [YOLO family](https://blog.roboflow.com/guide-to-yolo-models/){target=_blank}.

Check out the provided [Python scripts](https://github.com/maxsitt/insect-detect){target=_blank}
and the recommended
[processing pipeline](../deployment/detection.md#processing-pipeline){target=_blank}
for more information on how to deploy your custom trained detection model.

---

## YOLOv5

**YOLOv5n** is the most recommended model for the DIY camera trap at the moment.

[YOLOv5](https://github.com/ultralytics/yolov5){target=_blank}
([Jocher, 2020](https://doi.org/10.5281/zenodo.3908559){target=_blank})
is probably the most popular YOLO version, including models for
[object detection](https://github.com/ultralytics/yolov5#pretrained-checkpoints){target=_blank}
([architecture summary](https://github.com/ultralytics/yolov5/issues/6998){target=_blank}),
[image classification](https://github.com/ultralytics/yolov5#classification){target=_blank} and
[instance segmentation](https://github.com/ultralytics/yolov5#segmentation){target=_blank}.

It is highly recommended to read the
[tips for best training results](https://github.com/ultralytics/yolov5/wiki/Tips-for-Best-Training-Results){target=_blank}
if this is your first time training a detection model. Most of these tips are
also relevant if you choose a different YOLO version.

- **YOLOv5 detection model training** &nbsp;
  [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/maxsitt/insect-detect-ml/blob/main/notebooks/YOLOv5_detection_training.ipynb){target=_blank}

    > The PyTorch model weights can be converted to [.blob](https://docs.luxonis.com/en/latest/pages/model_conversion/){target=_blank}
      format at [tools.luxonis.com](https://tools.luxonis.com/){target=_blank} for on-device inference
      with the [Luxonis OAK](https://docs.luxonis.com/projects/hardware/en/latest/){target=_blank} devices.

Check the [introduction](https://colab.research.google.com/){target=_blank} and
[features overview](https://colab.research.google.com/notebooks/basic_features_overview.ipynb){target=_blank}
if this is your first time using a Colab notebook.

<figure markdown>
  ![Google Colab YOLOv5 training](assets/images/google_colab_train.png){ width="800" }
  <figcaption>Train your own YOLOv5 object detection model with the provided
              Google Colab notebook without having to write any code</figcaption>
</figure>

---

## YOLOv6

[YOLOv6](https://github.com/meituan/YOLOv6){target=_blank}
([Li et al., 2022](https://doi.org/10.48550/arXiv.2209.02976){target=_blank})
is specifically tailored for industrial applications and implements several
new features, including a efficient decoupled head, anchor-free detection and
optimized loss functions which can increase model performance.

- **YOLOv6 detection model training** &nbsp;
  [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/maxsitt/insect-detect-ml/blob/main/notebooks/YOLOv6_detection_training.ipynb){target=_blank}

    > The PyTorch model weights can be converted to [.blob](https://docs.luxonis.com/en/latest/pages/model_conversion/){target=_blank}
      format at [tools.luxonis.com](https://tools.luxonis.com/){target=_blank} for on-device inference
      with the [Luxonis OAK](https://docs.luxonis.com/projects/hardware/en/latest/){target=_blank} devices.

---

## YOLOv7

[YOLOv7](https://github.com/WongKinYiu/yolov7){target=_blank}
([Wang et al., 2022](https://doi.org/10.48550/arXiv.2207.02696){target=_blank})
implements new features, e.g. Extended-ELAN (E-ELAN) to enhance the learning
ability of the network, improved model scaling techniques and re-parameterized
convolution which can further increase model performance.

- **YOLOv7 detection model training** &nbsp;
  [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/maxsitt/insect-detect-ml/blob/main/notebooks/YOLOv7_detection_training.ipynb){target=_blank}

    > The PyTorch model weights can be converted to [.blob](https://docs.luxonis.com/en/latest/pages/model_conversion/){target=_blank}
      format at [tools.luxonis.com](https://tools.luxonis.com/){target=_blank} for on-device inference
      with the [Luxonis OAK](https://docs.luxonis.com/projects/hardware/en/latest/){target=_blank} devices.

---

## YOLOv8

[YOLOv8](https://github.com/ultralytics/ultralytics){target=_blank} by
[Ultralytics](https://ultralytics.com/){target=_blank}, the developers behind
YOLOv5, claims to reach the current state-of-the-art performance by
integrating new features, including anchor-free detection and a new backbone
network and loss function. A big difference to other YOLO models are the
developer-friendly options, such as the integrated
[command-line interface](https://docs.ultralytics.com/usage/cli/){target=_blank} (CLI)
and [Python package](https://docs.ultralytics.com/usage/python/){target=_blank}.

- **YOLOv8 detection model training** &nbsp;
  [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/maxsitt/insect-detect-ml/blob/main/notebooks/YOLOv8_detection_training.ipynb){target=_blank}

    > The PyTorch model weights can be converted to [.blob](https://docs.luxonis.com/en/latest/pages/model_conversion/){target=_blank}
      format at [tools.luxonis.com](https://tools.luxonis.com/){target=_blank} for on-device inference
      with the [Luxonis OAK](https://docs.luxonis.com/projects/hardware/en/latest/){target=_blank} devices.
