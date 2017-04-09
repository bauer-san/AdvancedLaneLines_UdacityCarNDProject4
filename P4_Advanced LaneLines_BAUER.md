# **Advanced Lane Finding Project** #
Project 4 of Udacity SDC NanoDegree

---
Geoff Bauer

8 April 2017

URL: https://github.com/bauer-san/AdvancedLaneLines_UdacityCarNDProject4

---
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

[raw_cb]: ./camera_cal/calibration3.jpg "raw"
[found_corners]: ./camera_cal/chessboard_13.jpg "found corners"
[undist_cb]: ./camera_cal/undist_calibration3.jpg "undistorted checkerboard"
[undist]: ./output_images/undist.jpg "undistorted"
[edges]: ./output_images/edges.jpg "edges"
[birdseye]: ./output_images/birdseye.jpg "birdseye"
[lanelines]: ./output_images/lanelines.jpg "lanelines"
[final]: ./output_images/final.jpg "final"


[video1]: ./project_video.mp4 "Video"

---
### Camera Calibration ###
The code for this step is contained in **./P4_CameraCalibration.ipynb**.  

The camera output needs to be calibrated because the image is distorted, radially and tangentially, and the distortion needs to be corrected so that the images can be used for estimating the curvature of the lane.

The calibration process starts by recording images of a chessboard in different orientations and processing them using built-in OpenCV functions.  The first step of the process was to find the corners of the chessboard in the images, which is done using the `cv2.findChessboardCorners()` function.  As the chessboard corners are found, the (x,y) pixel locations of the corners of the chessboard are collected in an array `imgpoints` and a corresponding array of 3-D object points, `objp`, is also collected.  The `imgpoints` and `obj` points, along with information about the image shape are inputs to the `cv2.calibrateCamera`, which calculates the distortion coefficients.  The distortion coefficients are saved to disc in dist_pickle.p for later use in the pipeline.

The distortion coefficients are then used in the `cv2.undistort` function to undistort images.  A sample raw image, corners found image, and undistorted image are shown:
![alt text][raw_cb]
![alt text][found_corners]
![alt text][undist_cb]


### Pipeline (single images) ###
The code for the image processing pipeline is contained in **./P4_Pipeline.ipynb**.  

#### 1. Provide an example of a distortion-corrected image. ####
The distortion coefficients that were determined and save in the previous step are used to undistort a raw test image.
![alt text][undist]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result. ####
I used a combination of gradient direction and magnitude and color thresholds and a region mask to generate a binary image in the `get_edges()` function of the `./P4_Pipeline.ipynb` notebook.  Here's an example of my output for this step.
![alt text][edges]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image. ####
The code for my perspective transform is in a cleverly named function, `perspective_transform()` of the `./P4_Pipeline.ipynb` file.  The `perspective_transform()` function takes as inputs an image (`img`), calculates source (`src`) and destination (`dst`) points and used the cv2.getPerspectiveTransform() and cv2.warpPerspective() functions to transform the image to a birds-eye view as show in this image:
![alt text][birdseye]


#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial? ####
The lane-line pixels are identified in the `find_lane_boundary()` function of the `./P4_Pipeline.ipynb` notebook, which takes the binary birds-eye image and uses a windowing technique.  The windowing technique starts by taking a histogram of the hot pixels and uses the mean position to seed the x position for the first windows.  The algorithm  moves the windows up the image and finds the 'hot' pixels in each adjacent window.  This is shown in this image:
![alt text][lanelines]
The (x, y) position of the hot pixels for left and right lane lines are collected, converted from pixel space to meters, and used to calculate the coefficients to fit a 2nd order polynomial.  This is done using Numpy function `polyfit()` also inside the `find_lane_boundary()` function of the `./P4_Pipeline.ipynb` notebook.

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center. ####
The radius of curvature of the lane is calculated from the polynomial coefficients in the `coefs_to_radius()` function of the `./P4_Pipeline.ipynb` notebook.  The position of the vehicle relative to the center of the lane is calculated in the `estimate_ego_lateral_position()` function of the `./P4_Pipeline.ipynb` notebook.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly. ####
I reversed the image transform to get the bird's-eye lane line projection back to driver perspective.  The reverse transform and image overlay are done in the `inv_perspective_transform()` function of the `./P4_Pipeline.ipynb` notebook.  Here is an example of my result on a test image:
![alt text][final]

---

### Pipeline (video) ###

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!). ####

Here's a [link to my video result](./P4_video_out.mp4)

---

### Discussion ###

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust? ####
The technical aspects of this project were not difficult.  The main problem I experienced was related to 'wiring' all of the provided lessons together...trying to understand how to interface the functions that were provided/discussed/developed in the lessons.  This was not difficult, but it just took some time.

The pipeline worked well with the project_video.mp4 but it does not work as well for the challenge_video.mp4.  One obvious improvement would be to make the ROI mask adapt size and shape depending on the detection of lane lines would make it more robust.  For the tighter curve radii of the harder_challenge_video.mp4, the 2nd order polynomial is not correct, at least not in combination with the current static ROI mask.