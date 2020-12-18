# tf2-pose-estimation

A (WIP) port of [tf-pose-estimation](https://github.com/ildoonet/tf-pose-estimation) with support for Tensorflow 2.

## NOTE: this project is still WIP, and might even never be finished! My personal knowledge about Tensorflow is almost inexistent but I am willing to try and learn.

## The idea

As the original project seems to not be very actively maintained anymore (though it is still impressive), I was willing to try and port it to TF2 in order to be able to use it with the latest up-to-date features of Tensorflow.

The main issue faced by the project is the use of `tf.contrib`, which has been [sunset (read: removed)](https://github.com/tensorflow/community/blob/master/rfcs/20180907-contrib-sunset.md) when Tensorflow 2 came out. Fortunately, after a first glance it looked to me that most of these could be replaced with libs such as [`tf_slim`](https://github.com/google-research/tf-slim) for example, so I wanted to give it a shot.

I started by running the `tf_upgrade_v2` script, fully aware that some issues would require manual fixing, but at least that gave me a first base of ported code, that is in this repo.

The full list of what apparently needs to be done is in [report.txt](report.txt), which is the report file of the script. Let this act as a TODO for now :)

---

Original README below:

# tf-pose-estimation

'Openpose', human pose estimation algorithm, have been implemented using Tensorflow. It also provides several variants that have some changes to the network structure for **real-time processing on the CPU or low-power embedded devices.**

**You can even run this on your macbook with a descent FPS!**

Original Repo(Caffe) : https://github.com/CMU-Perceptual-Computing-Lab/openpose

| CMU's Original Model</br> on Macbook Pro 15" | Mobilenet-thin </br>on Macbook Pro 15"                     | Mobilenet-thin</br>on Jetson TX2                   |
|:---------------------------------------------|:-----------------------------------------------------------|:---------------------------------------------------|
| ![cmu-model](/etcs/openpose_macbook_cmu.gif) | ![mb-model-macbook](/etcs/openpose_macbook_mobilenet3.gif) | ![mb-model-tx2](/etcs/openpose_tx2_mobilenet3.gif) |
| **~0.6 FPS**                                 | **~4.2 FPS** @ 368x368                                     | **~10 FPS** @ 368x368                              |
| 2.8GHz Quad-core i7                          | 2.8GHz Quad-core i7                                        | Jetson TX2 Embedded Board                          |

Implemented features are listed here : [features](./etcs/feature.md)

## Important Updates

- 2019.3.12 Add new models using mobilenet-v2 architecture. See : [experiments.md](./etcs/experiments.md)
- 2018.5.21 Post-processing part is implemented in c++. It is required compiling the part. See: https://github.com/ildoonet/tf-pose-estimation/tree/master/src/pafprocess
- 2018.2.7 Arguments in run.py script changed. Support dynamic input size.

## Install

### Dependencies

You need dependencies below.

- python3
- tensorflow 1.4.1+
- opencv3, protobuf, python3-tk
- slidingwindow
  - https://github.com/adamrehn/slidingwindow
  - I copied from the above git repo to modify few things.

### Pre-Install Jetson case

```bash
$ sudo apt-get install libllvm-7-ocaml-dev libllvm7 llvm-7 llvm-7-dev llvm-7-doc llvm-7-examples llvm-7-runtime
$ export LLVM_CONFIG=/usr/bin/llvm-config-7 
```

### Install

Clone the repo and install 3rd-party libraries.

```bash
$ git clone https://www.github.com/ildoonet/tf-pose-estimation
$ cd tf-pose-estimation
$ pip3 install -r requirements.txt
```

Build c++ library for post processing. See : https://github.com/ildoonet/tf-pose-estimation/tree/master/tf_pose/pafprocess
```
$ cd tf_pose/pafprocess
$ swig -python -c++ pafprocess.i && python3 setup.py build_ext --inplace
```

### Package Install

Alternatively, you can install this repo as a shared package using pip.

```bash
$ git clone https://www.github.com/ildoonet/tf-pose-estimation
$ cd tf-pose-estimation
$ python setup.py install  # Or, `pip install -e .`
```

## Models & Performances

See [experiments.md](./etc/experiments.md)

### Download Tensorflow Graph File(pb file)

Before running demo, you should download graph files. You can deploy this graph on your mobile or other platforms.

- cmu (trained in 656x368)
- mobilenet_thin (trained in 432x368)
- mobilenet_v2_large (trained in 432x368)
- mobilenet_v2_small (trained in 432x368)

CMU's model graphs are too large for git, so I uploaded them on an external cloud. You should download them if you want to use cmu's original model. Download scripts are provided in the model folder.

```
$ cd models/graph/cmu
$ bash download.sh
```

## Demo

### Test Inference

You can test the inference feature with a single image.

```
$ python run.py --model=mobilenet_thin --resize=432x368 --image=./images/p1.jpg
```

The image flag MUST be relative to the src folder with no "~", i.e:
```
--image ../../Desktop
```

Then you will see the screen as below with pafmap, heatmap, result and etc.

![inferent_result](./etcs/inference_result2.png)

### Realtime Webcam

```
$ python run_webcam.py --model=mobilenet_thin --resize=432x368 --camera=0
```

Apply TensoRT 

```
$ python run_webcam.py --model=mobilenet_thin --resize=432x368 --camera=0 --tensorrt=True
```

Then you will see the realtime webcam screen with estimated poses as below. This [Realtime Result](./etcs/openpose_macbook13_mobilenet2.gif) was recored on macbook pro 13" with 3.1Ghz Dual-Core CPU.

## Python Usage

This pose estimator provides simple python classes that you can use in your applications.

See [run.py](run.py) or [run_webcam.py](run_webcam.py) as references.

```python
e = TfPoseEstimator(get_graph_path(args.model), target_size=(w, h))
humans = e.inference(image)
image = TfPoseEstimator.draw_humans(image, humans, imgcopy=False)
```

If you installed it as a package,

```python
import tf_pose
coco_style = tf_pose.infer(image_path)
```

## ROS Support

See : [etcs/ros.md](./etcs/ros.md)

## Training

See : [etcs/training.md](./etcs/training.md)

## References

See : [etcs/reference.md](./etcs/reference.md)
