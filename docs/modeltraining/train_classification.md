# Image classification model training

The recommended [processing pipeline](../deployment/detection.md#processing-pipeline){target=_blank}
uses a [detection model](../index.md#detection-models){target=_blank} with only
one generic class ("insect"). The low input resolution enables a high inference
speed, which is necessary to reliably track moving/flying insects. Images of the
detected and tracked insects are cropped from synchronized HQ frames in real time.

By using the provided script for
[automated monitoring](../software/programming.md#automated-monitoring-script){target=_blank},
cropped detections of individual insects are saved as .jpg files and relevant
[metadata](../deployment/detection.md#metadata-csv){target=_blank} is saved to .csv for each
recording interval. The insect images can be classified in a subsequent step on
your local PC, by using a [classification model](../index.md#classification-model){target=_blank}
exported to [ONNX format](https://github.com/ultralytics/yolov5/issues/251){target=_blank}
for faster CPU inference.

Check out the [classification instructions](../deployment/classification.md){target=_blank}
for more info on how to deploy your classification model.

---

## YOLOv5

Since release [v6.2](https://github.com/ultralytics/yolov5/releases/v6.2){target=_blank},
classification model training and deployment is supported by
[YOLOv5](https://github.com/ultralytics/yolov5){target=_blank}.

To train your own image classification model, you can select between YOLOv5-cls,
ResNet and EfficientNet
[classification models](https://github.com/ultralytics/yolov5#classification){target=_blank},
pretrained on the ImageNet-1k dataset. A EfficientNet-B0 model is provided in the
[`insect-detect-ml`](https://github.com/maxsitt/insect-detect-ml){target=_blank}
GitHub repo.

- **YOLOv5 classification model training** &nbsp;
  [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/maxsitt/insect-detect-ml/blob/main/notebooks/YOLOv5_classification_training.ipynb){target=_blank}

    > The notebook for classification model training includes [export](https://github.com/ultralytics/yolov5/issues/251){target=_blank}
      to [ONNX](https://onnx.ai/){target=_blank} format for faster CPU inference.

Check the [introduction](https://colab.research.google.com/){target=_blank} and
[features overview](https://colab.research.google.com/notebooks/basic_features_overview.ipynb){target=_blank},
if this is your first time using a Colab notebook.
