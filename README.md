# AI for Cone Detection by Autonomous Car
#### by Ho Kai Siu

## Background
<p align="justify">
This project is part of the research on Autonomous Guiding System for Formula Student Autonomous (the FSA), which is a driverless racing competition for university student. The other four parts of the research are: the ROS simulation, Real-time monitoring system, PCBs designs and STM32 control and the AI for Map Generation, which are the honours projects of other four students. It is hoped our team could combine these five projects into a big system for the autonomous racing car.
</p>

<p align="center">
 <img src="/Pic/BG.png" width="500">
</p>

## Inspiration
<p align="justify">
The research is inspired by Mr Chan Tai Wing’s project in 2017, which shows the potential of AI guiding system on car racing. The “instant cone pair” detection algorithm was adopted, which would be discussed in the next page. Furthermore, our team had discussed with the PolyU racing team before to address their needs on an autonomous racing car. It is wished that this project could provide the framework of the autonomous racing car to the Racing team for participating in the Formula SAE in China.
</p>

## "Instant Cone Pair" Detection (Old)
#### by Mr Chan
<p align="justify">
Concerning the “instant cone pair” detection algorithm, it only focus on the first cone pair in front of the car. The detection would be operated pair by pair, which means that, the car only detect for the next cone pair after passing through the detected pair. The midpoint of the cone pair is calculated and determined for the car to pass through. This approach would give some problems to the system. For example, bottlenecking the car travel velocity since the car drive through the midpoint of the cone pair, which don’t give the shortest travel time. Moreover, the system is unable to handle a 90-degree corner as stated in Chan’s project.
</p>

<p align="center">
 <img src="/Pic/90corner.png" width="500">
</p>

<p align="center">
 <i>90-degree Turning Corner [1]</i>
</p>

## Car Racing
#### *In car racing, travel time is the first and foremost goal for participants to win the game*
#### Need *NEW* cone detection algorithm (This Project)

## To Reduce Travel Time
* In straight Path
  * Power of Engine (**Horsepower**)

* At Corner
  * Cutting Corner (**Racing Line**)
  <p style="text-align: center;">
   <img src:"/Pic/RacingLine.png" width="500">
 
   Racing Line Example \[2]
  </p>

## Project Objectives
To design an artificial intelligence (AI) guiding system for autonomous racing car
* Detect the incoming pattern of the traffic cones
* Determine the apex of the corner
* Give the corresponding velocity and steering angle

  <p align="center">
   <img src:"/Pic/cameraview.png" width="500">
  </p>

## System Design
<p align="center">
 <img src:"/Pic/systemdesign.png" width="500">
</p>

<p align="justify">
First, a stream of video frames are inputted into the system. Two HSV color masking would be operated to retrieve only the yellow and red color component in the frame, which are the cones colors in the left and right borders. Then, the masked frames are passed to the YOLO object detector to detect the cones in the frames. The centre coordinates of the cones are calculated and provided for the cone-to-path connection session to form the borderlines. After that, the apex in the borderlines would be detected. Finally, speed and steering angle for the micro-controller to drive the car across to apex would be sent.
</p>

## Hardware
<p align="justify">
The Nivida Jetson TX2 and a RGB camera were utilised. A Depth Camera, like RealSense D435i was used instead of RGB camera to give additional depth data of cone for more accurate cone-to-path connection
</p>

## Implementation
### HSV Color Masking
##### *with OpenCV*
<p align="justify">
 Two ranges of HSV values were set for retrieving all the yellow and red color components in the frame. 
</p>

(*To further reduce the chance of false-class detection*)


<p align="center">
 <img src:"/Pic/hsvtrackbar.png" width="500">

 *Trackbars for setting Lower (L) and Upper (U) H, S, V values*
</p>



### YOLO Object Detection
##### *YOLOv4*
<p align="center">
 <img src:"/Pic/yolov4.png" width="500">

 MC COCO Object Detection Comparison \[3]
</p>

<p align="justify">
 It was trained with a custom training dataset (300 RGB images of cones with different scales). 
 * 2 classes (*Yellow Cone, Red Cone*)
 * 416x416 network size
 * 64 branch size
 * 32 subdivisions
 * 6000 max_branch
 * 3200, 3600 steps
 * (2 classes + 5) x 3 = 21 filters before each YOLO layer
</p>

<p align="center">
 <img src:"/Pic/loss.jpeg" width="500">
 
 Loss
</p>

<p align="justify">
Since two masked frames are passed to the YOLO, two arrays of centres coordinates (*Yellow Cone, Red Cone*) are calculated and returned for the next procedure.
</p>

<p align="center">
 <img src:"/Pic/detectedcenter.png" width="500">

 Detected Cone’s boundary box with centre points marker
</p>

### Cone-to-Path (Cone Connection) Algorithm
##### *Transform unrelated coordinates of cone centers into related borders*
<p align="justify">
In the view of the car. The first pair of cones (**root cones**) possess the largest boundary box area and y-coordinate. The system filtered the detected cones by an area threshold of 600 and confident level of 0.6. The remaining cone centres were then connected in descending order of their y-coordinates.

Yet, the image would appear to converge at a vanishing point. Overlapping of cones near the point would occur, which degrade the cone connection. Therefore, this project would focus on the **first six cones only**.
</p>

<p align="center">
 <img src:"/Pic/connect6.png" width="500">

 Connections of the First Six Cone
</p>

### Apex Detection
<p align="justify">
 1. Calculate the slopes of the connected paths
    <p align="center">
     <img src:"/Pic/slopef.png" width="500">
    
     Slope formula
    </p>
</p>
