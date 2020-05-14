# PeopleCounter
Simple People Counter App designed to work at the Edge

## Prerequisites

You need Intel's OpenVino toolkit down installed on your machine and the OpenCV as well. You can read more about both these.

* [OpenCV](https://opencv.org) - The simple install should look like `pip install opencv-python`.
* [OpenVino toolKit](https://software.intel.com/en-us/openvino-toolkit) - See website for installation depending of your configuration.

In the context of the Udacity Nanodegree project, I have been provided with the following :

* FFMPEG server
* Mosca server
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


