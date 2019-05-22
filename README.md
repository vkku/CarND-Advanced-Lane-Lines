### Note : Due to inconsistency in Udacity Workbench, I'm submitting this project with this request to review the project from  my Github repo : [Github Link](https://github.com/vkku/CarND-Advanced-Lane-Lines). Issue report attached for evidence.

The error I was getting there was : 
```
error: /tmp/build/80754af9/opencv_1512491964794/work/modules/highgui/src/window.cpp:611: error: (-2) The function is not implemented. Rebuild the library with Windows, GTK+ 2.x or Carbon support. If you are on Ubuntu or Debian, install libgtk2.0-dev and pkg-config, then re-run cmake or configure script in function cvShowImage Changing kernel isn't helping
```

[//]: # (Image References)
[distortion_corrected]: output/distortion_Correction.png
[sobel_x]: output/sobel_x.png
[sobel_y]: output/sobel_y.png
[gradient_magnitude]: output/sobel_gradient.png
[gradient_direction]: output/sobel_direction.png
[color_thresholds]: output/reconciled_thresholds.png
[region_masked]: output/roi.png
[perspective_transform]: output/perspective_transform.png
[sliding_windows]: output/sliding_windows.png
[shaded_lanes]: output/shaded.png
[lane_mapping]: output/highlight.png



## Udacity's Self-Driving Car Nanodegree Program
### Project 4 - Advanced Lane Finding   

The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position. 

Let's get started ...  

---
In this project we apply different computer vision algorithms using opencv to perform advanced lane detection . First
we apply on test images  and then the same pipeline is being applied to video stream .

Below is the detail description of all different techniques .

###Camera Calibration 

First, I define "object points", which represent the (x, y, z) coordinates of the chessboard corners in the world. I assume that the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` is appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` is appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

Here are the results of distortion correction on each of the test images (located in the test_images folder):

![alt text][distortion_corrected]

###Pipeline (Images)

 Complete pipeline for the project can be described as a set of following steps :
 * Camera Calibration
 * Distortion Correction
 * Color Transformation and Gradient Calculation
 * Detecting Lanes and fitting boundary
 * Determing Curvature
 * Warping to original image
 * Adding mechanism to save frame data
 
 I have implemented the aforementioned steps (not in order of description)

+ Color Channel HLS & HSV Thresholding : I extracted the S-channel of the original image in HLS format and combine the result with the extracted V-channel of the original image in HSV format.

![alt text][color_thresholds]  

+ I used the sobel X and Y I've learned in previous modules

![alt text][sobel_x]
![alt text][sobel_y] 

The region masking results are shown below. I chose to limit the mask to a region with the following points: 

| Point       | Value                                    | 
|:-----------:|:----------------------------------------:| 
| Upper Left  | (image width x 0.4, image height x 0.65) | 
| Upper Right | (image width x 0.6, image height x 0.65) |
| Lower Right | (image width, image height)              |
| Lower Left  | (0, image height)                        |

![alt text][region_masked]

The code for my perspective transform (bird's eye view) is performed in a function  called transform_perspective (Section 4). The function takes in a thresholded binary image with the source points coinciding with the region masking points.

![alt text][perspective_transform]  

This steps deals with detecting lane pixels and to fit the polynomial in Section 5 of my code. I used sliding_windows and shaded_lanes to get a better result, The result of the step is :

Sliding Window Technique:

![alt text][sliding_windows]

Shaded Lane Technique:
![alt text][shaded_lanes] 

This step deals with the calculation of radius of curvatures. The table contains the resulting values.

| Test Image | Radius of Curvature (Left) | Radius of Curvature (Right) | 
|:----------:|:--------------------------:|:---------------------------:| 
| test1.png  | 2985.467894 meters         | 2850.142018 meters          | 
| test2.png  | 4984.505982 meters         | 12357.329365 meters         |
| test3.png  | 10088.084712 meters        | 2363.421967 meters          |
| test4.png  | 9894.520013 meters         | 2366.846436 meters          |
| test5.png  | 2548.327638 meters         | 6124.849321 meters          |
| test6.png  | 4173.313472 meters         | 45794.832663 meters         |

Another calculation performed was the offset from the lane's center. The calculations are shown in the code cell following the radius of curvature, and yielded the following:

| Test Image | Offset from Center |
|:----------:|:------------------:| 
| test1.png  | 1.020159 meters    |
| test2.png  | 0.901221 meters    |
| test3.png  | 0.935598 meters    |
| test4.png  | 1.123244 meters    |
| test5.png  | 0.930278 meters    |
| test6.png  | 1.070320 meters    |  

As a result the stacked image is :

![alt text][lane_mapping]


### Pipeline (Video)

The final video for the project can be viewed at : 

### P4_video_final.mp4
![Final_Video][P4_video_final.mp4]

### 

### Discussion
This project makes me reminds of some old classical computer vision problems and sets. I enjoyed going back and reorienting things and managing to get back with required precision and parameters. The steps I have undergone includes I've learned throught the course module and explored my myself. I recognize there are some instances my solution won't work, for extra challenging scenarios and extreme conditions like shiny road, which may create HSV image biasing is the thing I will like to work on if extending this project.
