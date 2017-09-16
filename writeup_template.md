# Behaviorial Cloning Project

[![Udacity - Self-Driving Car NanoDegree](https://s3.amazonaws.com/udacity-sdc/github/shield-carnd.svg)](http://www.udacity.com/drive)

Overview
---

In this project, deep neural networks and convolutional neural networks are developed in Keras to learn driving behavior using front camera images. The model will output a steering angle to an autonomous vehicle.

Udacity has provided a simulator where you can steer a car around a track for data collection. I have used image data and steering angles to train a neural network and then use this model to drive the car autonomously around the track.

Files description
---
The submission includes five files: 
* model.py (script used to create and train the model to predict steering angles)
* drive.py (script to drive the car in Udacity simulator using the trained model)
* model.h5 (the trained Keras model)
* writeup.md (report file)
* video.mp4 (a video recording of your vehicle driving autonomously around the track for at least one full lap)

Solution
---
Network:

I started with the LeNet architecture, but soon converted to the Nvidia's architecture which is the state of the art network for this purpose, [End to End Learning for Self-Driving Cars](http://images.nvidia.com/content/tegra/automotive/images/2016/solutions/pdf/end-to-end-dl-using-px.pdf).

The model is proved to be effective in the previous works. They start with importing a YUV image and normalizing the image. Then the model follows three CNN layers to extract the features from the input image. Then the output is flattened and fed into four fully connected layers and output a signle value which is the steering angle. 

I also added Dropout layers after the fully connected layers to avoid overfitting.

Data collection
---

The Udacity simulator provides a training section that captures and outputs the images from three cameras at the center, left and right side of the car and the streeing angle. I started with caputuring my own training data, however because I keyboard to drive the car in the tracks, the trained model did not drive the car smoothly. So I eventually used a data set provided by Udacity to train the model. 

Udacity's driving simulator provides two different test tracks, and all sample data was collected from track 1. As mentioned, the training data includes images from left, center and right and cameras on the front panel of the car (A sample of these traning images are shown bellow). 

Data Augmentation and Preprocessing
---
In the training data, there is only one steering angle provided for these left/center/right images. As the first method to augment the training data, the left and right images are also used with an offset (I chose 0.25) added and substracted from the steering angle. As the second augmentation method, for each training image, I filped it around the y axis and used the negative of the steering angle to the model. 

The input to the model as stated in the paper, should be a 66x200 pixels image with YUV layers. Howverm the original images from training have 160x320 size in RGB. Therefore, I crop the top 40 pixels and bottom 20 pixels of the input image which gives 100x320 pizels. I then resize the image to the desired 66x200 pixels. I eventually convert the image to YUV layers using a CV2 function. This processed image is fed to the netwerk for training.


### Network architecture

The Nvidia architecter model, as shown in the image, consist of 9 layers including a normalization layer, 5 convolution layers, and 3 fully connected layers. The image is fed with size 66x200x3 and the in the normalization layer the data are mapped into -1.0 to 1.0 range. The first convolution layer has 5x5 kernel and outputs 24@31x98. The second and third convolution layers have 5x5 kernel resulting in 36@14x47 and 48@5x22 respectively. The forth and fifth convolution layer have 3x3 kernel that output 64@3x20 and 64@1x18 respectively. Then the outputs are flatten resulting in 1164 nodes that are fed into three fully connected layers with 100, 50, 10 nodes and output into 1 node which is the steering angle. I also used dropout layers between the fully connected layers to avoid overfitting. The wights of the model are trained by minimizing the mean squared error between the output and the training steering angle using 'Adam' optimizer. The details of the model are shown in the following image. 

## Training   


##  Auto drive evaluation




### `drive.py`

Usage of `drive.py` requires you have saved the trained model as an h5 file, i.e. `model.h5`. See the [Keras documentation](https://keras.io/getting-started/faq/#how-can-i-save-a-keras-model) for how to create this file using the following command:
```sh
model.save(filepath)
```

Once the model has been saved, it can be used with drive.py using this command:

```sh
python drive.py model.h5
```

The above command will load the trained model and use the model to make predictions on individual images in real-time and send the predicted angle back to the server via a websocket connection.

Note: There is known local system's setting issue with replacing "," with "." when using drive.py. When this happens it can make predicted steering values clipped to max/min values. If this occurs, a known fix for this is to add "export LANG=en_US.utf8" to the bashrc file.

#### Saving a video of the autonomous agent

```sh
python drive.py model.h5 run1
```

The fourth argument, `run1`, is the directory in which to save the images seen by the agent. If the directory already exists, it'll be overwritten.

```sh
ls run1

[2017-01-09 16:10:23 EST]  12KiB 2017_01_09_21_10_23_424.jpg
[2017-01-09 16:10:23 EST]  12KiB 2017_01_09_21_10_23_451.jpg
[2017-01-09 16:10:23 EST]  12KiB 2017_01_09_21_10_23_477.jpg
[2017-01-09 16:10:23 EST]  12KiB 2017_01_09_21_10_23_528.jpg
[2017-01-09 16:10:23 EST]  12KiB 2017_01_09_21_10_23_573.jpg
[2017-01-09 16:10:23 EST]  12KiB 2017_01_09_21_10_23_618.jpg
[2017-01-09 16:10:23 EST]  12KiB 2017_01_09_21_10_23_697.jpg
[2017-01-09 16:10:23 EST]  12KiB 2017_01_09_21_10_23_723.jpg
[2017-01-09 16:10:23 EST]  12KiB 2017_01_09_21_10_23_749.jpg
[2017-01-09 16:10:23 EST]  12KiB 2017_01_09_21_10_23_817.jpg
...
```

The image file name is a timestamp of when the image was seen. This information is used by `video.py` to create a chronological video of the agent driving.

### `video.py`

```sh
python video.py run1
```

Creates a video based on images found in the `run1` directory. The name of the video will be the name of the directory followed by `'.mp4'`, so, in this case the video will be `run1.mp4`.

Optionally, one can specify the FPS (frames per second) of the video:

```sh
python video.py run1 --fps 48
```

Will run the video at 48 FPS. The default FPS is 60.

#### Why create a video

1. It's been noted the simulator might perform differently based on the hardware. So if your model drives succesfully on your machine it might not on another machine (your reviewer). Saving a video is a solid backup in case this happens.
2. You could slightly alter the code in `drive.py` and/or `video.py` to create a video of what your model sees after the image is processed (may be helpful for debugging).

## How to write a README
A well written README file can enhance your project and portfolio.  Develop your abilities to create professional README files by completing [this free course](https://www.udacity.com/course/writing-readmes--ud777).
