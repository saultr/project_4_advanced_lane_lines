## Advance Lane Lines Project

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

[image1]: ./img/undistorted.png "Undistorted"
[image2]: ./img/undistorted2.png "Undistorted"
[image3]: ./img/undistorted_img.png "Road Transformed"
[image4]: ./img/undistorted_img2.png "Road Transformed"
[image5]: ./img/thresholding.png "Thresholding process"
[image6]: ./img/perspective1.png "Region of interest"
[image7]: ./img/perspective2.png "Transformed image to birds-eye view"
[image8]: ./img/perspective3.png "Binary transforemed image"
[image9]: ./img/warped_straight_lines.jpg "Warp Example"
[image10]: ./img/find_line.png "Find lines"
[image11]: ./img/curvature.jpg "Radius curvature calculation"
[image12]: ./img/str_lane.png "Lane overlay example in a straight"
[image13]: ./img/corner_lane.png "Lane overlay example in a corner"
[image14]: ./img/sliding_window.png "Lane overlay example in a corner"
[image15]: ./img/region.png "Lane overlay example in a corner"

[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in `advance_lane_lines.ipynb`.  
I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

![alt text][image2]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this ones:
![alt text][image3]

![alt text][image4]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of Sobel gradient thresholding in x direction and S color thresholds to generate a binary image (thresholding steps at lines 70 through 94 in `advance_lane_lines.ipynb` in Color Transforms and Gradients section).  Here's an example of my output for this step.  

![alt text][image5]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warper()`, which appears in lines 27 through 8 in the file `advance_lane_lines.ipynb` in Perspective transform section.  The `warper()` function takes as inputs an image (`img`), as well as source (`src_vertices`) and destination (`dst_vertices`) points.  The best values found for this transformation have been:


```python
# Define the region
region_interest_vertices =  np.array([[580,460],
                                     [705,460],
                                     [1110,720],
                                     [220,720]])
# For destination points, I'm choosing: x-> 200, 1080  y-> 0, 720
region_destination_vertices =  np.array([[200, 0],
                                        [1080, 0], 
                                        [1080, 720], 
                                        [200, 720]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 580, 460      | 200, 0        | 
| 705, 460      | 1080, 0       |
| 1110, 720     | 1080, 720     |
| 220, 720      | 200, 720      |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image6]

![alt text][image7]

![alt text][image8]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

The process to identify lane-lines pixels is performed by `find_lines()` function (lines 98 through 225 in `advance_lane_lines.ipynb` - Find Lines section). It has two different methods depending on previous frames.

* Sliding window search method. If no previous line has been detected, an histogram of the bottom part of the image is done. Max peak points in the histogram are picked and all the points detected in that window region are taken. It also will be used as center point for the next upper window. The process will continue until arrive to the top of the image. All points detected are stored into a vector.  

![alt text][image14]
        
* Local region method. In this case, assuming that a polynomial fit has been deteced in the previous frame, that information will be used to define the region to look for valid points. Only pixels near the previous polynomial fit will be search on. It will speed up calculation time. In case that this method fails for one frame in left or right line, it will be automatically switched to the previous method and a full search will be performed in this new frame.

![alt text][image15]

The condition I have used to determine if a valid polynomial is detected is based on the density of points (min number of points). For future versions more validation methods will be introduced.
    
Below it is shown the final result for a given frame:
    
![alt text][image10]    
    


#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in lines 84 through 92 in `measure_radius_of_curvature()` in my code in `advance_lane_lines.ipynb` - Find Lines section. 

The following equation for radius of curvature has been used:

Rcurve =(1+(2Ay+B)^2)^3/2)/∣2A∣

Pixels to meters conversion has done with the following factors:
```python
ym_per_pix = 30/720 # meters per pixel in y dimension
xm_per_pix = 3.7/700 # meters per pixel in x dimension
```
![alt text][image11] 

I calculated the position of each line respect the center in lines 211 through 214 inside `find_lines()` in `advance_lane_lines.ipynb` - Find Lines section. Last pixel of the polynomial curve corresponding to the bottom of the image is picked (left_fitx[-1] and right_fitx[-1]). The calculated car offset position is the average distance between both lines measurements to the center of the image and it is done in line 256 of `overlay_lane()` function. 


#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in lines 240 through 264 in my code in `advance_lane_lines.ipynb` - Find Lines section, in the function `overlay_lane()`.  Here is an example of my result on a test image:

![alt text][image12]
![alt text][image13]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./output_videos/project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?
The problems I encountered were almost exclusively due to lighting conditions, shadows, discoloration, etc. It wasn't difficult to dial in threshold parameters to get the pipeline to perform well on the original project video. Other colors like the B channel of the LAB colorspace could be tested to improve it.
Also some problems isolating the correct left line for the the challege video.
The width of the lane is different in each video and the system is confused with wall. I've considered a few possible approaches for making my algorithm more robust in this aspect by weighting the histogram results for the inner pixels and and dicarting the outer ones (in case of a wall very close to the line will chose the line). Also a dynamic region of interest implementation would help.
I hope to review some of these strategies in the future.