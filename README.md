# pi-opencv
This is OpenCV 3.4.1 build for Raspberry Pi (Raspbian Jessie 2017-07-05).

Source codes are from https://github.com/opencv/opencv

## Known problems
During the build some xml files from opencv were not include into the package.

https://github.com/opencv/opencv/tree/master/data/haarcascades

I was not able to resolve that problem.

So please download the missed files from their origin or from this repo:

https://github.com/tprlab/pi-opencv/tree/master/opencv-extra/haarcascades

## DNN
I built OpenCV primarily to use its DNN module for images detection with Tensorflow.

So far Tensorflow-provided network graphs are normally going ahead of OpenCV development, the latest Tensorflow graph versions are not compatible with this build.

I used SSD Mobilenet from here:

http://download.tensorflow.org/models/object_detection/ssd_mobilenet_v1_coco_11_06_2017.tar.gz

To use OpenCV-DNN a couple more files required:
* [mscoco_label_map](https://github.com/tprlab/pi-opencv/blob/master/opencv-extra/mscoco_label_map.pbtxt)
* [ssd_mobilenet_v1_coco](https://github.com/tprlab/pi-opencv/blob/master/opencv-extra/ssd_mobilenet_v1_coco.pbtxt)

## Installation

Just like a regular deb package:

```
sudo dpkg -i opencv_3_4-raspian_jessie.deb
```

## Usage


Python sample:


```
import cv2 as cv
import tf_labels
import sys

DNN_PATH = "---path-to:ssd_mobilenet_v1_coco_11_06_2017/frozen_inference_graph.pb"
DNN_TXT_PATH = "--path-to:ssd_mobilenet_v1_coco.pbtxt"
LABELS_PATH = "--path-to:mscoco_label_map.pbtxt"

tf_labels.initLabels(PATH_TO_LABELS)
cvNet = cv.dnn.readNetFromTensorflow(pb_path, pb_txt)

img = cv.imread(sys.argv[1])
rows = img.shape[0]
cols = img.shape[1]
cvNet.setInput(cv.dnn.blobFromImage(img, 1.0/127.5, (300, 300), (127.5, 127.5, 127.5), swapRB=True, crop=False))
cvOut = cvNet.forward()

for detection in cvOut[0,0,:,:]:
    score = float(detection[2])
    if score > 0.25:
        left = int(detection[3] * cols)
        top = int(detection[4] * rows)
        right = int(detection[5] * cols)
        bottom = int(detection[6] * rows)
        label = tf_labels.getLabel(int(detection[1]))
        print(label, score, left, top, right, bottom)
        text_color = (23, 230, 210)
        cv.rectangle(img, (left, top), (right, bottom), text_color, thickness=2)
        cv.putText(img, label, (left, top), cv.FONT_HERSHEY_SIMPLEX, 1, text_color, 2)

cv.imshow('img', img)
cv.waitKey()
```

The module tf_labels and its dependencies can be downloaded from here:

https://github.com/tprlab/pi-opencv/blob/master/samples/dnn

This is a wrapper to read the labels for detected classes.

It is based on slightly modified sources of Tensorflow (to get rid the dependence of Tensorflow itself).

The original sources are here:

https://github.com/tensorflow/models/blob/master/research/object_detection/utils/label_map_util.py



