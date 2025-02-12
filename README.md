## Final Project - System Integration

This is the project repo for the final project of the Udacity Self-Driving Car Nanodegree: Programming a Real Self-Driving Car. For more information about the project, see the project introduction [here](https://classroom.udacity.com/nanodegrees/nd013/parts/6047fe34-d93c-4f50-8336-b70ef10cb4b2/modules/e1a23b06-329a-4684-a717-ad476f0d8dff/lessons/462c933d-9f24-42d3-8bdc-a08a5fc866e4/concepts/5ab4b122-83e6-436d-850f-9f4d26627fd9).

<img src="imgs/simulator.gif" width="640" alt="Simulator run" />

### The team

It is a solo submission by: 

- [Viatcheslav Tretyakov](https://github.com/vtretyakov) (viatcheslav.tretyakov@gmail.com)

### Implementation details

#### Software architecture

The following is a system architecture diagram showing the ROS nodes and topics used in the project.

<img src="imgs/final-project-ros-graph-v2.png" width="860" alt="Software architecture" />

#### Traffic lights detection

For detecting traffic lights from a camera feed a pre-trained on the [COCO dataset](http://mscoco.org/) model [ssdlite_mobilenet_v2_coco](http://download.tensorflow.org/models/object_detection/ssdlite_mobilenet_v2_coco_2018_05_09.tar.gz) has been taken from the [Tensorflow detection model zoo](https://github.com/tensorflow/models/blob/master/research/object_detection/g3doc/detection_model_zoo.md). The model was selected based on the processing speed as being the fastest among the latest listed. Although the frozen inference graphs were generated using the [v1.8.0](https://github.com/tensorflow/tensorflow/tree/v1.8.0) release version of Tensorflow and the project required version [v1.3.0](https://github.com/tensorflow/tensorflow/tree/v1.8.0), the model appeared to be compatible with that version.

<img src="imgs/classified_box.jpg" width="480" alt="Detected traffic light" />

#### Traffic light color identification

Once the model finds the traffic lights and provides you the boundary boxes, the next step is to crop the traffic light images from the scene based on those boxes and identify the color. The approach is entirely based on image processing:
1. Convert the image into LAB color space and isolate the L channel. Good support material can be found [here](https://www.learnopencv.com/color-spaces-in-opencv-cpp-python/).

<img src="imgs/LAB_l_crop.png" width="100" alt="LAB color space channel `l`" />

2. Split the traffic light cropped image onto three equal segments - upper, middle, and lower corresponding to read, yellow, and green lights respectively. 

<img src="imgs/red_slice.png" width="100" alt="Upper segment" />
<img src="imgs/yellow_slice.png" width="100" alt="Middle segment" />
<img src="imgs/green_slice.png" width="100" alt="Lower segment" />

3. To identify the color, we need to find out which segment is brighter. Thanks to the LAB color space, L channel gives us exactly that information. All we need to do is to find the sum of all pixels in each of the three segments. The highest score gives us the traffic light color. 

`The light is GREEN`

In the real scenario several filtering methods are applied to give a more reliable estimate. 
1. Among all the lights detected in a frame, the one with the highest confidence score is selected.
2. The respected lights' boundary box is verified based on the aspect ratio. If it doesn't fit into our threshold, that means the image will not be correctly cropped and the color may be wrongly identified.
3. Gamma correction was used to enhance too bright images at every second frame.

<img src="imgs/traffic_light_training.gif" width="480" alt="traffic_light_training.bag" />

<img src="imgs/loop_with_traffic_light.gif" width="480" alt="loop_with_traffic_light.bag" />

<img src="imgs/just_traffic_light.gif" width="480" alt="just_traffic_light.bag" />

#### Traffic lights detection v.2.0

Although the results with the [ssdlite_mobilenet_v2_coco](http://download.tensorflow.org/models/object_detection/ssdlite_mobilenet_v2_coco_2018_05_09.tar.gz) model were quite satisfactory, it didn't perform good enough on the `traffic_light_training.bag` bagfile. To improve the performance, another model was used ([ssd_mobilenet_v1_coco_2017_11_17](https://github.com/vtretyakov/CarND-Capstone/tree/master/ros/src/tl_detector/ssd_mobilenet_v1_coco_2017_11_17)) that was retrained on a dataset [kindly shared](https://drive.google.com/file/d/0B-Eiyn-CUQtxdUZWMkFfQzdObUE/view?usp=sharing) by one of the Nanodegree's alumni. It was a great help finding a project of another student with a GitHub nickname [coldKnight](https://github.com/coldKnight/TrafficLight_Detection-TensorFlowAPI) explaining how to re-train the models. But due to the project's requirement to use TensorFlow 1.3 the steps described could not be directly followed since the research models and the tools required to train the models for TensorFlow 1.3 are kindly removed from the [earlier releases](https://github.com/tensorflow/models/releases). The version supported on the date of this writing was 1.8.0.

There are two ways one could still get the older version of the `tensorflow/models` repo:

1. On [archive.org](https://archive.org/):

```sh
$ wget https://archive.org/download/github.com-tensorflow-models_-_2017-10-05_18-42-08/tensorflow-models_-_2017-10-05_18-42-08.bundle
$ git clone tensorflow-models_-_2017-10-05_18-42-08.bundle -b master
$ mv -r tensorflow-models_-_2017-10-05_18-42-08 ~/tensorflow/models
```
2. By knowing the commit's hash:

```sh
$ cd ~/tensorflow/
$ git clone https://github.com/tensorflow/models.git
$ cd models
$ git checkout edcf29f
```
After that you need to follow a simple installation process [described here](https://github.com/tensorflow/models/blob/master/research/object_detection/g3doc/installation.md). In case od doubt you can always find the instruction inside your folder: `research/object_detection/g3doc/installation.md`.

The last step described there is to test the installation. If it fales, please perform two more simple steps:

```sh
$ python setup.py build
$ python setup.py install
```

At this point the test script should pass.

The result after re-training the model:

<img src="imgs/traffic_light_training_v2.gif" width="480" alt="traffic_light_training.bag" />

### Native Installation

* Be sure that your workstation is running Ubuntu 16.04 Xenial Xerus or Ubuntu 14.04 Trusty Tahir. [Ubuntu downloads can be found here](https://www.ubuntu.com/download/desktop).
* If using a Virtual Machine to install Ubuntu, use the following configuration as minimum:
  * 2 CPU
  * 2 GB system memory
  * 25 GB of free hard drive space

  The Udacity provided virtual machine has ROS and Dataspeed DBW already installed, so you can skip the next two steps if you are using this.

* Follow these instructions to install ROS
  * [ROS Kinetic](http://wiki.ros.org/kinetic/Installation/Ubuntu) if you have Ubuntu 16.04.
  * [ROS Indigo](http://wiki.ros.org/indigo/Installation/Ubuntu) if you have Ubuntu 14.04.
* [Dataspeed DBW](https://bitbucket.org/DataspeedInc/dbw_mkz_ros)
  * Use this option to install the SDK on a workstation that already has ROS installed: [One Line SDK Install (binary)](https://bitbucket.org/DataspeedInc/dbw_mkz_ros/src/81e63fcc335d7b64139d7482017d6a97b405e250/ROS_SETUP.md?fileviewer=file-view-default)
* Download the [Udacity Simulator](https://github.com/udacity/CarND-Capstone/releases).

### Docker Installation
[Install Docker](https://docs.docker.com/engine/installation/)

Build the docker container
```bash
docker build . -t capstone
```

Run the docker file
```bash
docker run -p 4567:4567 -v $PWD:/capstone -v /tmp/log:/root/.ros/ --rm -it capstone
```

### Port Forwarding
To set up port forwarding, please refer to the [instructions from term 2](https://classroom.udacity.com/nanodegrees/nd013/parts/40f38239-66b6-46ec-ae68-03afd8a601c8/modules/0949fca6-b379-42af-a919-ee50aa304e6a/lessons/f758c44c-5e40-4e01-93b5-1a82aa4e044f/concepts/16cf4a78-4fc7-49e1-8621-3450ca938b77)

### Usage

1. Clone the project repository
```bash
git clone https://github.com/udacity/CarND-Capstone.git
```

2. Install python dependencies
```bash
cd CarND-Capstone
pip install -r requirements.txt
```
3. Make and run styx
```bash
cd ros
catkin_make
source devel/setup.sh
roslaunch launch/styx.launch
```
4. Run the simulator

### Real world testing
1. Download [training bag](https://s3-us-west-1.amazonaws.com/udacity-selfdrivingcar/traffic_light_bag_file.zip) that was recorded on the Udacity self-driving car.
2. Unzip the file
```bash
unzip traffic_light_bag_file.zip
```
3. Play the bag file
```bash
rosbag play -l traffic_light_bag_file/traffic_light_training.bag
```
4. Launch your project in site mode
```bash
cd CarND-Capstone/ros
roslaunch launch/site.launch
```
5. Confirm that traffic light detection works on real life images



