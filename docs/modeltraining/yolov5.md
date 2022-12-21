# YOLOv5 Training

[YOLOv5](https://github.com/ultralytics/yolov5) is probably the easiest to use
computer vision framework, including architectures for
[object detection](https://github.com/ultralytics/yolov5/wiki/Train-Custom-Data),
[classification](https://github.com/ultralytics/yolov5/pull/8956) and
[instance segmentation](https://github.com/ultralytics/yolov5/releases/v7.0)
models. You can find a summary of the YOLOv5 detection model architecture
[here](https://github.com/ultralytics/yolov5/issues/6998). It is highly
recommended to read the
[tips for best training results](https://github.com/ultralytics/yolov5/wiki/Tips-for-Best-Training-Results)
if this is your first time training a deep learning model.

You can use the provided [Google Colab](https://colab.research.google.com/)
notebooks for custom training of a

- [**YOLOv5 detection model**](https://colab.research.google.com/github/maxsitt/insect-detect-ml/blob/main/notebooks/YOLOv5_detection_training_OAK_conversion.ipynb)
- [**YOLOv5 classification model**](https://colab.research.google.com/github/maxsitt/insect-detect-ml/blob/main/notebooks/YOLOv5_classification_training.ipynb)

with your own dataset. The notebook for detection model training also includes
all necessary steps to convert your model into the
[.blob format](https://docs.luxonis.com/en/latest/pages/model_conversion) for
on-device inference on the Luxonis OAK cameras. In the notebook for
classification model training, you will
[export](https://github.com/ultralytics/yolov5/issues/251) your trained model
to [ONNX](https://onnx.ai/) format for faster CPU inference on your local PC.
All steps in both notebooks are easy to follow and include interactive forms
that enable you to change the important parameters without having to write any
code.

After opening the notebook, log in to your Google account and under the **File**
menu choose **Save a copy in Drive**. Now you can change the code and adapt the
training procedure to your use case if desired. Before connecting to a Google
Colab cloud instance, make sure to check that **GPU** is selected as Hardware
accelerator under **Runtime --> Change runtime type**.

<figure markdown>
  ![Google Colab YOLOv5 training](assets/images/google_colab_train.png){ width="800" }
  <figcaption>Train your own YOLOv5 object detection model with the provided
              Google Colab notebook without having to write any code</figcaption>
</figure>
