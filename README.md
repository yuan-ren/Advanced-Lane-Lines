# Advanced Lane Finding Project
----------------------

In this project, computer vision techniques are used to detect lines from camera images. Detailed goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

[//]: # (Image References)

[image1]: ./images/undistort_output.png "Undistorted"
[image2]: ./images/undistort_output2.png "Road Transformed"
[image3]: ./images/binary_combo_example.png "Binary Example"
[image4]: ./images/warped_example.png "Warp Example"
[image5]: ./images/sliding_window_search.png "Sliding Window Search"
[image6]: ./images/inv_perspective.png "Output"
[video1]: ./project_video_output.mp4 "Video"

---

### Camera Calibration

#### 1. Compute camera matrix and distortion coefficients are compute:

The code for this step is contained in the second code cell of the IPython notebook located in "./Project.ipynb".  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. An example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]

#### 2. Use color transforms, gradients or other methods to create a thresholded binary image.  

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps in the fourth code cell of the IPython notebook).  Here's an example of my output for this step. 

![alt text][image3]

#### 3. Perspective transform and an example of a transformed image.

The code for my perspective transform in contained in the fifth code cell of the IPython notebook.  The source and destination points I chose are listed below:


| Source        | Destination   | 
|:-------------:|:-------------:| 
| 220, 720      | 320, 720        | 
| 1110, 720      | 920, 720      |
| 570, 470     | 320, 1      |
| 722, 470      | 920, 1        |

I verified that my perspective transform was working as expected by drawing the source and destination points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Identify lane-line pixels and fit their positions with a polynomial

I performed a sliding window search to identify lane-line pixels. To start with, I used histogram to find positions of 2 lanes, then used a moving slide window to find lane positions based on previous location. I have used 10 windows of width 100 pixels.
The x & y coordinates of non zeros pixels are found, a polynomial is fit for these coordinates and the lane lines are drawn. 

![alt text][image5]

Sliding window searching was only used in the first frame of the video. Because lane positions in the next frame should not move too far, I searched the new postion based on previous results. This was implemented in function `search_lane_in_margin` in the 7th code cell of IPython notebook.

#### 5. Calculate the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in `get_curvature_and_offset` function in the 8th code cell of IPython notebook. The radius of curvature is computed according to the formula and method described in the classroom material. Polynomial fit was calculated in pixels. But because the curvature has to be calculated in real world matrics, I have to use a pixel to meter transformation and recompute the fit again.

The mean of the lane pixels closest to the car gives us the center of the lane. The center of the image gives us the position of the car. The difference between the 2 is the offset from the center.

#### 6. An example image of result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in the function `inverse_transform()` in the 9th code cell of IPython notebook.  Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Link to the final video output.  

Here's a [link to my video result](./project_video_output.mp4)

---

### Discussion

#### 1. Problems / issues faced in this project. 

In the final pipeline (10th code cell in IPython notebook), I used two additional techiques to make the pipeline more robust. First one is to smooth curvature and offset by taking average of previous 10 frames. This is to get rid of possible jumps in calculation results.

Second one is to detect bad frames. There are some frames where lanes cannot be detected correctly. Bad frames are identified if calculated curvatures of left and right lanes are too different. In case of bad frame, we use calculated results from last frame. 
