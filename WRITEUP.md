# Project Write-Up

## Explaining Custom Layers

The process behind converting custom layers involves using the tools provided with the model optimizer to extract concerned layers (extension generator - extgen.py). Then implementing the layer for use with concerned hardware (eg. CPU, GPU...), and finally run the conversion by specifying layers implementations.

Some of the potential reasons for handling custom layers are : 
- the use of a layer in the source model which isn't supported by the model optimizer.
- wanting to implement a layer in a different way that what's done by the model optimizer.


## Comparing Model Performance

### ssdlite_mobilenet_v2_coco

Since I planned to test this project on a raspberry pi, I chose to test a light model and downloaded ssdlite_mobilenet_v2_coco from http://download.tensorflow.org/models/object_detection/ssdlite_mobilenet_v2_coco_2018_05_09.tar.gz

I converted it with : 
`python mo_tf.py --input_model frozen_inference_graph.pb
--tensorflow_use_custom_operations_config ssd_v2_support.json
--tensorflow_object_detection_api_pipeline_config ssd_mobilenet_v2_coco.config
--data_type FP16`

With my simple first version of the app, it counted 683 detected persons on the test video with a 0.6 probability threshold, suming all frames detections. For the speed, the "time" command mesured 2m23s.

### ssd_resnet_50_fpn_coco

I chose to test a model with better accuracy, and downloaded the ssd_resnet_50_fpn_coco from http://download.tensorflow.org/models/object_detection/ssd_resnet50_v1_fpn_shared_box_predictor_640x640_coco14_sync_2018_07_03.tar.gz

I converted it the same way as the first model (even if I has some issues because I though to use another json config first) :
`python /opt/intel/openvino/deployment_tools/model_optimizer/mo_tf.py --input_model frozen_inference_graph.pb --tensorflow_use_custom_operations_config ssd_v2_support.json --tensorflow_object_detection_api_pipeline_config pipeline.config --data_type FP16`

On the model zoo page, they gave numbers of 76ms / 35 COCOmAP for ssd_resnet_50_fpn_coco versus 27ms / 22 COCOmAP for ssdlite_mobilenet_v2_coco. So I was expecting a running time 3 times longer than my first test, as a fair price to obtain a far better accuracy.
But running the same simple first version of the app, I stopped the execution after 35 minutes... Big disappointment !

### VGG_ilsvrc_ssd_300

I found a Caffe model here : https://www.deepdetect.com/models/
I modified the deploy.prototxt file to include a correct input header.
Converted it with :
`python /opt/intel/openvino/deployment_tools/model_optimizer/mo_caffe.py --input_model VGG_ilsvrc_ssd_300.caffemodel --input_proto deploy.prototxt --input data,rois --input_shape (1,3,227,227),[1,6,1,1]`

With my simple first version of the app (modified to fit ouputs of this model), it counted 862 detected persons on the test video with a 0.6 probability threshold, suming all frames detections. For the speed, the "time" command mesured 30m32s "real" & 28m53s "user".

So if the accuracy is far better, the performance of this model was not suitable for our needs.

### ssd_mobilenet_v2_coco

Finally, I decided to try this basic ssd instead of the ssdlite version I first tried.

http://download.tensorflow.org/models/object_detection/ssd_mobilenet_v2_coco_2018_03_29.tar.gz

`python /opt/intel/openvino/deployment_tools/model_optimizer/mo_tf.py --input_model frozen_inference_graph.pb --tensorflow_use_custom_operations_config /opt/intel/openvino/deployment_tools/model_optimizer/extensions/front/tf/ssd_v2_support.json --tensorflow_object_detection_api_pipeline_config pipeline.config --reverse_input_channels`

And I ran the same basic script than before : it counted 736 detected persons on the test video with a 0.6 probability threshold, suming all frames detections. For the speed, the "time" command mesured 3m13s. 


### Comparing with the basic use of Tensorflow

To compare models before and after conversion to Intermediate Representations, I rewrote the inference.py and its Network class to a Tensorflow version inference_tf.py based on Tensorflow tutorials (https://github.com/tensorflow/models/blob/master/research/object_detection/object_detection_tutorial.ipynb)
to compare the performance on the same video file, with the same 0.6 threshold. As a starter with Tensorflow, I just hope I did things right.

I added several checks to main.py to automate the detection of a "".pb" model that triggers the use of the NetworkTf class.

- With the ssdlite_mobilenet_v2_coco model, I obtained a total count of detections of 602, in 3 minutes.
- With the ssd_mobilenet_v2_coco_2018_03_29, I obtained a total count of detections of 567, in 4m12s.

Pre- and post-conversion results :

- With the ssd_mobilenet_v2_coco_2018_03_29 :
Total count of persons : 8 vs 6
Average presence time : 18s vs 22s
Total inference time : 224s vs 43s
Inference time by frame : 161ms vs 31ms

- With the ssdlite_mobilenet_v2_coco :
Total count of persons : 7 vs 6
Average presence time : 19s vs 22s
Total inference time : 146s vs 44s
Inference time by frame : 104ms vs 31ms

This means in the two tested models a worse detection on a longer time before the conversion : that's a very good point in favor of Intermediate Representations !

Other numbers mesured :

The difference between model accuracy (total number of detections) pre- and post-conversion was :
- 602 vs 683 (12% better) ssdlite_mobilenet_v2_coco
- 567 vs 736 (23% better) for ssd_mobilenet_v2_coco

The size of the model pre- and post-conversion was :
- 19M vs 8.7M for ssdlite_mobilenet_v2_coco
- 67M vs 65M  for ssd_mobilenet_v2_coco

The total running time of the script with model pre- and post-conversion was :
- 3m vs 2m23s (21% faster) for ssdlite_mobilenet_v2_coco
- 4m12s vs 3m13s (23% faster) for ssd_mobilenet_v2_coco

The CPU usage :
I used the system command `ps -eocomm,pcpu | grep python | egrep -v '(0.0)|(%CPU)'` to check the CPU usage during script execution.
- 70% vs 54% CPU usage (23% less) with ssd_mobilenet_v2_coco
- 66% vs 39% CPU usage (41% less) with ssdlite_mobilenet_v2_coco


### Comparing cloud services with edge

We've seen that in terms of performance, using IR at the edge is working fine.
But there's other sides to look at :
- Network consumption, in terms of bandwith, are going to be much less than if we use cloud ML services.
- After a time (because edge hardware isn't free), edge computing is also money saving. For example, I found that using Watson Machine Learning Cloud Standard costs USD 0.50 /1,000 predictions.


## Assess Model Use Cases

Some of the potential use cases of the people counter app are :
- To know how many persons comes to a specific place, like a break room.
- To know how many persons are entering or exiting a place like a movie theater.

Each of these use cases would be useful because :
- Employer can make a bigger room if it's too crowded, or encourage people from a specific floor to go to another break room.
- In case of ermergency (like a fire), the authorities should know as fast as possible if there's still people inside the place.


## Assess Effects on End User Needs

Lighting, model accuracy, and camera focal length/image size have different effects on a deployed edge model.

In order to test the potential effects of each of these, I've run two sets of experiences. Videos and screenshots at 15s of the video are in the resources folder for reference.

I first tried to lower the brightness of the example video with ffmpeg to "-0.3" and "-0.6" settings.
With the ssd_mobilenet_v2_coco model, I observed a very tiny difference of accuracy for the 30% darker video, but a significant 15% loss (632 vs 736 detections) for the 60% darker. However, even a human could have difficulties spotting peoples in the 60% darker video, so I consider this result as a rather good performance.

I also tried to lower by a factor of 2 the resolution of the video, from a 768px width to 384px.
With the same ssd_mobilenet_v2_coco model, I observed a 12% accuracy loss (651 detections). However, the video file size was shrunk from 3.2M to 1.2M (-62%). So in cases of low bandwith situations, this could also be considered as an acceptable loss.
