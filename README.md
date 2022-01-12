# masking-model-mobile-app

The code in this repository is adapted from the code published by Dennis Ippel in this [repository](https://github.com/MasDennis/SegmentationAndOcclusion), and outlined in this [Medium article](https://rozengain.medium.com/using-coreml-in-arkit-for-object-segmentation-and-occlusion-988a6c7e18dd).

## Product Overview
The goal of the *masking-model-mobile-app* is to run image processing models on the live video captured by an iPhone camera. This project was started with the ultimate goal of being able to run the masking model (found [here](https://github.com/talemhealthanalytics/MaskRCNN)) on live video captured on the user's iPhone. 

## Technical Overview
![App flowchart](https://github.com/talemhealthanalytics/masking-model-mobile-app/blob/main/Flowchart.png?raw=true "App Flowchart")
This app uses Swift and Apple CoreML tools to run a user-defined machine learning model on device. By using the **vision** library, the app is able to interface the video collected on the device, running the model in real-time. 

Two models are used in this code, the first is used for object detection and the second is used for segmentation. The object detection is done using a *YOLOv3Int8LUT* model, and the second one should *ideally* be our masking model. **This is the only remaining piece of the puzzle that would complete this project, and the remaining blocker is highlighted below in the blockers section**. For the time being, a *DeepLabV3Int8Image* model is used for segmentation instead. As a result, the app segments entire cars rather than just the damaged region of interest.

First, the view is loaded and a request to load the camera feed is sent. If successful, the app will then call `objectDetectionRequest` on the video feed, acting on each frame collected from the video. This function accesses the *YOLOv3Int8LUT* and is functional in it's current state.

Next, if an object is found, `segmentationRequest` is called on the detected object, segmenting the detected vehicle and occluding it from the outputted video to the user (This causes a reduction in FPS and causes the app to become bloated around 30FPS). If no object is found, the object detection request is continued. This function currently accesses the *DeepLabV3Int8Image*, and must be replaced with our masking model to be considered complete.

## Developer instructions
- First, the latest version of XCode must be installed, which can be done by following [these instructions](https://mac.install.guide/commandlinetools/3.html). 
- Once this repository is pulled, open Xcode and select "Open an existing project".
- Navigate to the directory in which the `.Xcodeproj` can be found, and select this file. This will open the Xcode workspace where all development will be done.
- To load the app onto an iPhone, connect the mobile device to your computer and select it from the dropdown menu at the top pane of the editor, and press the play button on the left hand side of the editor. 
- Note that simulators will not work with this model, as it must access the device's camera which is not possible for simulated devices.s

## Blockers
The main blocker to the completion of this app is plugging in our masking model instead of the *DeepLabV3Int8Image* model. As our model is a segmentation model which would segment the damaged region of interest of a car rather than the car entirely, which is what the app currently does. 

The blocker is caused by the architecture of our masking model, which prevents us from transforming the model from a TensorFlow model to an MLModel. The following approaches were tried:
- First, we tried converting the model from its `.h5` representation to an MLModel. However, this failed as the model is written in TensorFlow 1 rather than TensorFlow 2.
- Second, we tried converting the model from its `.pb` and `.graph` file to an MLModel, as is recommended for TensorFlow 1 models, however these formats were never available with the masking model, and attempts to create them have been postponed until a valid use case that would prioritize the masking model is found.
