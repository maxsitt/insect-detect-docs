# Image classification model training

## YOLOv5-cls

[YOLOv5](https://github.com/ultralytics/yolov5){target=_blank} is probably the
easiest to use computer vision framework, including models for
[object detection](https://github.com/ultralytics/yolov5/wiki/Train-Custom-Data){target=_blank},
[image classification](https://github.com/ultralytics/yolov5/pull/8956){target=_blank} and
[instance segmentation](https://github.com/ultralytics/yolov5/releases/v7.0){target=_blank}.

You can use the provided [Google Colab](https://colab.research.google.com/){target=_blank}
notebook for custom training of a:

- [**YOLOv5 image classification model**](https://colab.research.google.com/github/maxsitt/insect-detect-ml/blob/main/notebooks/YOLOv5_classification_training.ipynb){target=_blank}
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
