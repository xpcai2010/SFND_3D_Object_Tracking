# SFND 3D Object Tracking

[//]: # (Image References)
[image0]: ./summary/images/veryshort.gif "project"
[image1]: ./summary/images/evaluation1.png "ev1"
[image2]: ./summary/images/TTC_1.png "ttc1"
[image3]: ./summary/images/evaluation2.png "ev2"
[image4]: ./summary/images/metric.png "metric"
[image5]: ./summary/images/TTC_2.png "ttc2"
[image6]: ./summary/images/evaluation3.png "ev3"

Welcome to the final project of the camera course. By completing all the lessons, you now have a solid understanding of keypoint detectors, descriptors, and methods to match them between successive images. Also, you know how to detect objects in an image using the YOLO deep-learning framework. And finally, you know how to associate regions in a camera image with Lidar points in 3D space. Let's take a look at our program schematic to see what we already have accomplished and what's still missing.

<img src="images/course_code_structure.png" width="779" height="414" />

In this final project, you will implement the missing parts in the schematic. To do this, you will complete four major tasks: 
1. First, you will develop a way to match 3D objects over time by using keypoint correspondences. 
2. Second, you will compute the TTC based on Lidar measurements. 
3. You will then proceed to do the same using the camera, which requires to first associate keypoint matches to regions of interest and then to compute the TTC based on those matches. 
4. And lastly, you will conduct various tests with the framework. Your goal is to identify the most suitable detector/descriptor combination for TTC estimation and also to search for problems that can lead to faulty measurements by the camera or Lidar sensor. In the last course of this Nanodegree, you will learn about the Kalman filter, which is a great way to combine the two independent TTC measurements into an improved version which is much more reliable than a single sensor alone can be. But before we think about such things, let us focus on your final project in the camera course. 

## Dependencies for Running Locally
* cmake >= 2.8
  * All OSes: [click here for installation instructions](https://cmake.org/install/)
* make >= 4.1 (Linux, Mac), 3.81 (Windows)
  * Linux: make is installed by default on most Linux distros
  * Mac: [install Xcode command line tools to get make](https://developer.apple.com/xcode/features/)
  * Windows: [Click here for installation instructions](http://gnuwin32.sourceforge.net/packages/make.htm)
* Git LFS
  * Weight files are handled using [LFS](https://git-lfs.github.com/)
* OpenCV >= 4.1
  * This must be compiled from source using the `-D OPENCV_ENABLE_NONFREE=ON` cmake flag for testing the SIFT and SURF detectors.
  * The OpenCV 4.1.0 source code can be found [here](https://github.com/opencv/opencv/tree/4.1.0)
* gcc/g++ >= 5.4
  * Linux: gcc / g++ is installed by default on most Linux distros
  * Mac: same deal as make - [install Xcode command line tools](https://developer.apple.com/xcode/features/)
  * Windows: recommend using [MinGW](http://www.mingw.org/)

## Basic Build Instructions

1. Clone this repo.
2. Make a build directory in the top level project directory: `mkdir build && cd build`
3. Compile: `cmake .. && make`
4. Run it: `./3D_object_tracking`.


## Writeup - Final Report

![alt text][image0]

## Task FP.0
### _Provide a Writeup / README that includes all the rubric points and how you addressed each one_


## Task FP.1 Match 3D Objects
### _Implement the method "matchBoundingBoxes", which takes as input both the previous and the current data frames and provides as output the ids of the matched regions of interest (i.e. the boxID property). Matches must be the ones with the highest number of keypoint correspondences._

### The function of `void matchBoundingBoxes(std::vector<cv::DMatch> &matches, std::map<int, int> &bbBestMatches, DataFrame &prevFrame, DataFrame &currFrame)` is implenmented in *<camFusion_Student.cpp>*. 

## Task FP.2 Compute Lidar-based TTC

### _Compute the time-to-collision in second for all matched 3D objects using only Lidar measurements from the matched bounding boxes between current and previous frame._

### The function of `void computeTTCLidar(std::vector<LidarPoint> &lidarPointsPrev, std::vector<LidarPoint> &lidarPointsCurr, double frameRate, double &TTC)` is implenmented in *<camFusion_Student.cpp>*. 

## Task FP.3 Associate Keypoint Correspondences with Bounding Boxes
### _Prepare the TTC computation based on camera measurements by associating keypoint correspondences to the bounding boxes which enclose them. All matches which satisfy this condition must be added to a vector in the respective bounding box._

### The function of `void clusterKptMatchesWithROI(BoundingBox &boundingBox, std::vector<cv::KeyPoint> &kptsPrev, std::vector<cv::KeyPoint> &kptsCurr, std::vector<cv::DMatch> &kptMatches)` is implenmented in *<camFusion_Student.cpp>*. 


## Task FP.4 Compute Camera-based TTC

### _Compute the time-to-collision in second for all matched 3D objects using only keypoint correspondences from the matched bounding boxes between current and previous frame._

### The function of `void computeTTCCamera(std::vector<cv::KeyPoint> &kptsPrev, std::vector<cv::KeyPoint> &kptsCurr, std::vector<cv::DMatch> kptMatches, double frameRate, double &TTC, cv::Mat *visImg)` is implenmented in *<camFusion_Student.cpp>*. 

## Task FP.5 Performance Evaluation 1
### _Find examples where the TTC estimate of the Lidar sensor does not seem plausible. Describe your observations and provide a sound argumentation why you think this happened._


### Evaluation - Implausible Values

### Below are the results of Lidar and Camera TTC from all the provided KITTI data. The detector type is _SHITOMASI_, the descriptor type is _BRISK_. I also used _MAT_BF_ as matcher type. 

![alt text][image1]

- As you can see, there are negative or Infinitive TTC values. It starts from frame 53. It's because the car comes to a stop. 
- At frame 43 and 49, the lidar TTC shows negative values while camera TTC doesn't. It could be the road is uneven, we may need to readjust the range of lidar height dynamically. At the moment, it's a fixed range: _minZ = -1.5, maxZ = -0.9_. 


### The graph below plots the first 40 frames of TTC from Lidar and Camera's estimation. There are some large deviation between Lidar and Camera's TTC. 

![alt text][image2]

- For the outliers of lidar TTC, the error could be that we consider only one closest point on the rear bumper of the vehicle to calculate TTC.
- For the outliers of camera TTC, the error could be related to the detector and/or descriptor type that we choose.   

### The improvements that we can adopt to improve the TTC accuracy are
- Fuse lidar data with camera data and use Kalman Filter to calculate TTC.
- For the Lidar TTC, we need to consider dynamically adjust the range of Lidar detection range that we use for TTC calculation.
- Also for the Lidar TTC, we may need to consider multiple points instead of the closest point on the rear bumper of the vehicle to calcuate TTC.
- For the Camera TTC, we need to select a better detector and/or descriptor type that we use for TTC calculation.
  
## Task FP.6 Performance Evaluation 2
### _Run several detector / descriptor combinations and look at the differences in TTC estimation. Find out which methods perform best and also include several examples where camera-based TTC estimation is way off. As with Lidar, describe your observations again and also look into potential reasons._


### I ran all the combinations of detector and descriptor with **matcherType = "MAT_BF"** and **matcher_SelectorType = "SEL_KNN"**. I created a metric to evaluate each combination of detector and descriptor by comparing TTC calculated from camera to the TTC calculated from lidar. The metric formula is listed as below. Here I use the first 40 frames. So N = 40

![alt text][image4]

### The metric for all combinations of detector and descriptor is listed as below.  

#### note: 
- please be aware that AKAZE descriptor works only with AKAZE detector and ORB descripotor doesn't work with SIFT detector.
- Harris detector is not able to estimate the camera TTC based on the provided image data. 

![alt text][image3]

### The summary spreadsheet _`<summary of Camera TTC estimators.xlsx>`_ is located ./summary/

### As we can see, the top3 combinations of detector and descriptor providing best Camera TTC are 
- **SIFT(detector) + SIFT(descriptor)**
- **SIFT(detector) + BRIEF(descriptor)**
- **SIFT(detector) + FREAK(descriptor)** 

### The graph below is the **best camera TTC** calcuated from SIFT detector with SIFT descriptor. It provides the best camera TTC estimation, closest to the lidar TTC.

![alt text][image5]

### The frame by frame result of  SIFT detector with SIFT descriptor is also listed below
![alt text][image6]
