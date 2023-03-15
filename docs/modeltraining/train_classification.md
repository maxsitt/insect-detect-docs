# Image classification model training

The recommended [processing pipeline](../deployment/detection.md#processing-pipeline){target=_blank}
uses a [YOLOv5n](../index.md#detection-models){target=_blank} detection model
with only one generic class ("insect"), as the low input resolution, which is
necessary for the model to run fast enough to track flying insects, would result
in a low classification accuracy in most cases. The detections are then cropped
from HQ frames with a high enough resolution to enable good classification results
in a subsequent step on your local PC.

Check out the [classification instructions](../deployment/classification.md){target=_blank}
for more information on how to deploy the classification model and the
[automated analysis](../deployment/analysis.md){target=_blank} for post-processing
of the classification results.

---

## YOLOv5-cls

Since release [v6.2](https://github.com/ultralytics/yolov5/releases/v6.2){target=_blank},
classification model training and deployment is supported by
[YOLOv5](https://github.com/ultralytics/yolov5){target=_blank}.

To train your own image classification model, you can select between YOLOv5-cls,
ResNet and EfficientNet
[classification models](https://github.com/ultralytics/yolov5#classification){target=_blank},
pretrained on the ImageNet-1k dataset. A YOLOv5s-cls model is provided in the
[`insect-detect-ml`](https://github.com/maxsitt/insect-detect-ml){target=_blank}
GitHub repo, trained on the
[Insect_Detect_classification](https://universe.roboflow.com/maximilian-sittinger/insect_detect_classification){target=_blank}
dataset. When exported to ONNX format, this model runs very fast even on a
standard CPU, while still providing a good accuracy. Choose a bigger model
to achieve potentially higher classification accuracy and if speed is not
so important or your hardware (e.g. GPU) allows it.

You can use the provided [Google Colab](https://colab.research.google.com/){target=_blank}
notebook for custom training of a:

- [**YOLOv5 image classification model**](https://colab.research.google.com/github/maxsitt/insect-detect-ml/blob/main/notebooks/YOLOv5_classification_training.ipynb){target=_blank} &nbsp;
  [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/maxsitt/insect-detect-ml/blob/main/notebooks/YOLOv5_classification_training.ipynb){target=_blank}
  > Includes [export](https://github.com/ultralytics/yolov5/issues/251){target=_blank}
    to [ONNX](https://onnx.ai/){target=_blank} format for faster CPU inference on your PC.

Check the [introduction](https://colab.research.google.com/){target=_blank} and
[features overview](https://colab.research.google.com/notebooks/basic_features_overview.ipynb){target=_blank},
if this is your first time using a Colab notebook.

After opening the notebook, log in to your Google account and under the **File**
menu choose **Save a copy in Drive**. Now you can change the code and adapt the
training procedure to your use case if desired. Before connecting to a Google
Colab cloud instance, make sure to check that **GPU** is selected as Hardware
accelerator under **Runtime --> Change runtime type**.
