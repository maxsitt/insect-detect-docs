# Annotation

To "teach" CNN models **which** objects to detect **where** in an image
(detection/localization) and **what** these objects are (classification), you
will first have to annotate/label the objects of interest manually to generate
a training dataset.

---

## Annotation tools

Various open and closed source tools working offline on your PC or via an
online platform in your browser are available. These tools have different
pricing plans, but most of them can be used for free with usage limits or for
education/academia. All of them will give you the basic functionalities of
annotating images for object detection (bounding boxes) or classification
(single image label). Some are easy to use, others are more complex and can
provide additional functions such as AutoML ("one-click model training") or
model assisted labeling.

Some of these annotation tools are:

- [Roboflow](https://roboflow.com/)
- [CVAT](https://www.cvat.ai/)
- [Label Studio](https://labelstud.io/)
- [Supervisely](https://supervise.ly/)
- [labelme](https://github.com/wkentaro/labelme)
- [SuperAnnotate](https://www.superannotate.com/)
- [Hasty](https://hasty.ai/)
- [V7 Labs](https://www.v7labs.com/)
- [Labelbox](https://labelbox.com/)

---

## Roboflow

Because of its free [public plan](https://roboflow.com/pricing), direct
connection to a repository for [open source datasets](https://roboflow.com/universe)
and ease of use with many helpful functions, we chose Roboflow to annotate our
images. The Roboflow [Python package](https://docs.roboflow.com/python) makes
it easy to automatically upload annotated datasets e.g. when using a Google
Colab notebook as training environment.

If you are new to image annotation and dataset management, it is recommended to
study the [Roboflow documentation](https://docs.roboflow.com/), especially for
the [Annotate tool](https://docs.roboflow.com/annotate). Also take a look at
[best practices for image labeling](https://blog.roboflow.com/tips-for-how-to-label-images/),
as a dataset with optimally labeled images can significantly increase model
performance and accuracy.

The [dataset health check](https://docs.roboflow.com/dataset-health-check) can
give you a quick overview of your current dataset and show possible problems,
e.g. regarding class imbalance. With different
[image preprocessing](https://docs.roboflow.com/image-transformations/image-preprocessing)
steps you can easily modify your dataset in multiple ways (e.g. resize, modify
classes).
[Image augmentations](https://docs.roboflow.com/image-transformations/image-augmentation)
can diversify your dataset, e.g. by applying image rotation, exposure or blur.
Multiple dataset can be generated in this way from the same annotated source
images. These datasets can then be [exported](https://docs.roboflow.com/exporting-data)
in all common formats, either as .zip file or as download link that can be used
to upload the dataset e.g. into a Google Colab model training notebook.

<figure markdown>
  ![Roboflow Annotate](assets/images/roboflow_annotate.jpg){ width="800" }
  <figcaption>Roboflow Annotate, a simple Browser-based GUI for annotating
              images, e.g. for object detection (bounding boxes)</figcaption>
</figure>
