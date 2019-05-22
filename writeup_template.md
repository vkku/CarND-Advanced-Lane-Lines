## Writeup Template

### You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

### Note : Due to inconsistency in Udacity Workbench, I'm submitting this project with this request to review the project from  my Github repo : [Github Link](https://github.com/vkku/CarND-Advanced-Lane-Lines). Issue report attached for evidence.

![Udacity_Workbech_Error][resources\projectSubmitError.jpg]
The error I was getting there was : 
```
error: /tmp/build/80754af9/opencv_1512491964794/work/modules/highgui/src/window.cpp:611: error: (-2) The function is not implemented. Rebuild the library with Windows, GTK+ 2.x or Carbon support. If you are on Ubuntu or Debian, install libgtk2.0-dev and pkg-config, then re-run cmake or configure script in function cvShowImage Changing kernel isn't helping
```

---

**Advanced Lane Finding Project**

The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

[//]: # (Image References)

[image1]: output/Camera_Calibration.png "Camera Calibrated"
[image2]: output/Undistorted.png "Undistorted"
[image3]: output/reconciled_thresholds.png "Thresholding"
[image4]: output/perspective_transform.ong "Perspective Transformation"
[image5]: output/sliding_windows.png "Sliding Windows"
[image6]: output/shaded.png "Shaded Lanes"
[image7]: resources/curvature_formula.jpg "Formula for finding curvature"
[image8]: output/annodated_img.png "Annodated Image"

# Please Review my project At : [Github](https://github.com/vkku/CarND-Advanced-Lane-Lines)


## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "Project2_Submission.ipynb"

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.
The chessboard used is of 8 * 6 dimension.

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![Camera_Calibration][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

I used the coefficients extracted above to undistort image.
![Undistorted][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used various methods for region isolation that included :
* Sobel_X
* Sobel_Y
* Gradient Magnitude
* Sobel Direction
* Color Threshold

I further used a region of interest to specific points in image :
|      Point    |                 Value                      |
| ------------- | ------------------------------------------ |
| Upper Left    |  (image width x 0.4, image height x 0.65)  |
| Upper Right   | (image width x 0.6, image height x 0.65)   |
| Lower Right   | (image width, image height)                |
| Lower Left    | (0, image height)                          |

I used specifically the following filter to narrow down the image :
```
 binary_output[(binary_x == 1) & (binary_y == 1) & (mag == 1) | (color == 1) | (mag == 1) & (direction == 1)] = 1
        
```
![reconciled_thresholds][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `transform_perspective()`.  The function takes as inputs an image (`img`) and returns the warped_output image  I chose the hardcode the source and destination points in the following manner:

```python
src = np.array([[(width*0.4, height*0.65),
                        (width*0.6, height*0.65),
                        (width, height),
                        (0, height)]], 
                        dtype=np.float32)
    dst = np.array([[0,0], 
                    [img.shape[1], 0], 
                    [img.shape[1], img.shape[0]],
                    [0, img.shape[0]]],
                    dtype = 'float32')
```

![perspective_transform][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I created a histogram, carved slots out of my image found the midpoint and lane starting points and fit my lane lines with a 2nd order polynomial kinda like this in method sliding_windows() :
```
    left_fitx = left_fit[0]*ploty**2 + left_fit[1]*ploty + left_fit[2]
    right_fitx = right_fit[0]*ploty**2 + right_fit[1]*ploty + right_fit[2]
```

I prefered sliding window technique
![sliding_windows][image5]
![shaded][image6]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I calculated curvature of the lane in roc_in_pixels() I calculated them according to the formula :

![formula][image7]

as :

```

    left_curverad = ((1 + (2*left_fit[0]*y_eval + left_fit[1])**2)**1.5) / np.absolute(2*left_fit[0])
    right_curverad = ((1 + (2*right_fit[0]*y_eval + right_fit[1])**2)**1.5) / np.absolute(2*right_fit[0])
    
```
I calculated vehicle positioning in offset() by calculating difference of polygon and image centre and multipying them with measured values in real world.


#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I used lane_highlight() to fill polygon between left and right lane pixels and annotate() to scribe results on image.


![annodated_img][image8]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](P4_video_final.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

* Based on the time taken to compile video, some performance enhancements can be done on the domain of calculation. Thereby reducing redundant calculations.

* More robust region thresholding and gradient calculation can be done by varied calculations as Laplacian edge detectors.

* While moving to next window more parameters can be preserved as to have a robust overall process.

Moreover, I faces issues 


