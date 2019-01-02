<div align="center">
  <img src="https://www.tensorflow.org/images/tf_logo_transp.png"><br><br>
</div>

-----------------


**This** is a complete procedure makeing every step done, which starts with taking your own photos on [Raspberry Pi 3 board](https://www.raspberrypi.org) and ends with your face recognized by the board :-) 
Of course, it is implemented with [Tensorflow Lite](https://www.tensorflow.org/lite/). Following the steps guided by the project, developers interested in [TfLite](https://www.tensorflow.org/lite/) can create your integral story armed with the deep-learning technique on [the Pi](https://www.raspberrypi.org).

**1,** Following [Raspberry Pi Software Guide](https://www.raspberrypi.org/learning/software-guide/quickstart), install **Rapibian** onto a  Raspberry Pi 3 board. Then
```shell
$sudo apt install build-essential
```
Follow [TensorFlow Raspberry Pi Examples](https://github.com/tensorflow/tensorflow/tree/master/tensorflow/contrib/pi_examples), a camera can be installed onto the Pi. Verify if the camera works:
```shell
$raspistill -v
```

**2,** On the Pi, create a folder **"camera"** and a sub-folder (**"camera/hxz"** on my case). Take photos of your faces:
```shell
$cd hxz
$raspistill -v -o hua1_%d.jpg -w -512 -h -512 -t 60000 -tl 400 -e jpg -q 100
```
It will generate about 150 photos at **"hxz"**. You may adjust flags -t (timeout) and -tl (time lapse) to take more photos and do fast.

**3,** Type following commands on the terminal of your computer, to run a [docker container](https://docs.docker.com/get-started/part2/). 
```shell 
$sudo docker container start <container name>
$sudo docker container attach <container name>
```
On my case, Ubuntu 18.04 runs for both host and container.

**4,** Enter the container,
```shell
#apt update
#apt upgrade
```

**5,** Input the following commands in the container, to have [TensorFlow For Poets](https://codelabs.developers.google.com/codelabs/tensorflow-for-poets/#0) ready in the container:
```shell
#apt install python-pip
#pip install --upgrade "tensorflow==1.12.*"
#apt install git
#git clone https://github.com/googlecodelabs/tensorflow-for-poets-2
#cd tensorflow-for-poets-2
#apt install curl
#curl http://download.tensorflow.org/example_images/flower_photos.tgz \
    | tar xz -C tf_files
```

**6,** ***Copy*** the folder **"hxz"** from Pi into the sub-folder **"tf_files/flower_photos"** of the container. Then at folder **"tensorflow-for-poets-2"**, input the following commands:
```shell
#IMAGE_SIZE=224
#MODEL_SIZE=0.50
#ARCHITECTURE="mobilenet_${MODEL_SIZE}_${IMAGE_SIZE}"
#python -m scripts.retrain   --bottleneck_dir=tf_files/bottlenecks   --how_many_training_steps=500   --model_dir=tf_files/models/   --summaries_dir=tf_files/training_summaries/"${ARCHITECTURE}"   --output_graph=tf_files/retrained_graph.pb   --output_labels=tf_files/retrained_labels.txt   --architecture="${ARCHITECTURE}"   --image_dir=tf_files/flower_photos
#tflite_convert   --graph_def_file=tf_files/retrained_graph.pb   --output_file=tf_files/optimized_graph.lite   --input_format=TENSORFLOW_GRAPHDEF   --output_format=TFLITE   --input_shape=1,${IMAGE_SIZE},${IMAGE_SIZE},3   --input_array=input   --output_array=final_result   --inference_type=FLOAT   --input_data_type=FLOAT
```
You will find 2 files created at folder **"tf_files"**: **"optimized_graph.lite"** and **"retrained_labels.txt"**

**7,** Follow [tensorflow/tensorflow
 R1.12 #24194](https://github.com/tensorflow/tensorflow/pull/24194), you could get a Linux executable **"camera" **, /tensorflow/tensorflow/contrib/lite/tools/make/gen/rpi_armv7l/bin/camera. 
 Copy the executable from the container to the folder **"camera"** on Pi, along with **"optimized_graph.lite"** and **"retrained_labels.txt"**. 
Run app **"camera"** at folder **"camera"**:
```shell
$./camera -f 10 -m optimized_graph.lite -l retrained_labels.txt
```
The "confidence" will be reported based on the every frame caught right now and the ~150 photos taken on step **2** :-) 
Running
```shell
$./camera --help
```
can display usages for each flag. When running the app, you could change the flags according to your purpose. E.g., increase -f to catch more frames, then you could get enough time to move your opsition on the front of camera, to see how "confidence" will change.