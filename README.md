# PeopleCounter

Simple People Counter App designed to work at the Edge.

Given a video file, it add a box around the detected persons, count the different people as they are detected, keep track of the average presence time and publish these stats to a MQTT server.

This is my work for the "Deploy a People Counter App at the Edge" project of the Udacity "Intel® Edge AI for IoT Developers Nanodegree" Program.

## Prerequisites

You need Intel's OpenVino toolkit down installed on your machine and the OpenCV as well. You can read more about both these.

* [OpenCV](https://opencv.org) - The simple install should look like `pip install opencv-python`.
* [OpenVino toolKit](https://software.intel.com/en-us/openvino-toolkit) - See website for installation depending of your configuration.

In the context of the Udacity Nanodegree project, I have been provided with the following :

* FFMPEG server (https://trac.ffmpeg.org/wiki/ffserver)
* Mosca server (https://github.com/moscajs/mosca)
* GUI webservice

If you want to use this project on your own web service, you'll have to adapt the publishing part.

## Usage

Example command line to start analyzing a video file :
```
python main.py -i resources/Pedestrian_Detect_2_1_1.mp4 -m models/ssdlite_mobilenet_v2_coco.xml -l /opt/intel/openvino/deployment_tools/inference_engine/lib/intel64/libcpu_extension_sse4.so -d CPU -pt 0.6 | ffmpeg -v warning -f rawvideo -pixel_format bgr24 -video_size 768x432 -framerate 24 -i - http://0.0.0.0:3004/fac.ffm
```

This project is also capable of analyzing an image file :
```
python main.py -i resources/snap1.png -m models/ssdlite_mobilenet_v2_coco.xml -l /opt/intel/openvino/deployment_tools/inference_engine/lib/intel64/libcpu_extension_sse4.so -d CPU -pt 0.6
```
In this case, it will output a simple image file named `single_image.png` like this :
![](resources/single_image.png?raw=true)

## Models used

This script has been tested with various models, and is best used with an OpenVino intermediate representation (IR). It also supports Tensorflow frozen models, but with poor results in terms of accuracy and inference time.

Two IR models has been included in this repository, converted from the Tensorflow models from the [model zoo](https://github.com/tensorflow/models/blob/master/research/object_detection/g3doc/detection_model_zoo.md) :
* [ssd_mobilenet_v2_coco.xml](models/ssd_mobilenet_v2_coco.xml)
* [ssdlite_mobilenet_v2_coco.xml](models/ssdlite_mobilenet_v2_coco.xml)

More informations on converting a TensorFlow model to OpenVino IR can be found [here](https://docs.openvinotoolkit.org/latest/_docs_MO_DG_prepare_model_convert_model_Convert_Model_From_TensorFlow.html)

## Licence

These scripts are my work for the "Deploy a People Counter App at the Edge" project of the Udacity "Intel® Edge AI for IoT Developers Nanodegree" Program and are based on Intel Corporation base scripts provided for the project.

They should not be used to complete your own project. Plagiarism is a violation of the Udacity Honor Code. Udacity has zero tolerance for plagiarized work submitted in any Nanodegree program.

