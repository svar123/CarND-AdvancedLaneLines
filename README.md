

## **Advanced Lane Finding Project**

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

[image1]: ./output_images/undistch.JPG "Undistorted"
[image2]: ./output_images/undistrd.JPG "Road Transformed"
[image3]: ./output_images/colorth.JPG "Color"
[image4]: ./output_images/warped.JPG "Warped"
[image5]: ./output_images/rect.JPG "Rectangles"
[image6]: ./output_images/finalimage.JPG "Final"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the code cells 3,4,27 of the jupyter notebook located in "AdvancedLaneFinding.ipynb".  

I started by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function. I applied this distortion correction to the checkerboard image and the test images using the `distort_corr` function which calls the `cv2.undistort()` function to undistort the images.

Checkerboard Image:

![alt text][image1]



### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I applied the distortion correction to one of the test images as shown below. First, I read in an image and then called the function `distort_corr` which used the matrix and distortion coeffts from calibration to undistort the image.

![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image. 

I used a combination of color thresholds to generate a binary image (thresholding steps in code cell 7 of the jupyter notebook. The function is called `color_threshold()`). I used thresholding for r-channel, g-channel (RGB), s-channel & l-channel (HSL) and v-channel (HSV). I experimented with using gradient(Sobelx and Sobely) but it introduced a lot of noise. So I decided to go with just color thresholding. Here is an example of the color binary image.

![alt text][image3]


#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image. 

Then I performed a perspective transform on this binary color output (perspective transform steps in code cell 8, function name is `warp()`).

I hard-coded the source and destination points by testing on the images.

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 570, 470      | 340, 0        | 
| 220, 720      | 340, 720      |
| 1120, 720     | 980, 720      |
| 730, 470      | 980, 0        |


Here are the warped images.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

The function `fit_line()` in code cell 21 locates the lane lines using the sliding windows and fits a polynomial. First I drew a histogram using the lower half of the warped image which gave me the starting point for the left and right lanes. Then I used sliding window placed around the line centers to find and follow the lines to the top of the frame as shown in this example.

![alt text][image5]


#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this code cell 13 `radius_curvature()`, code cell 14 `avg_rad_cur()` and code cell 15 `center_offset()`. The radius of curvature was calculated using the formula:
R = ((1 + (2Ay+B) ** 2 ) ** 3/2)/ abs(2A) where A and B are coeffts of the line. 
To calculate the position of the vehicle with respect to center, I first calculated the position of the camera (which is center of image). Then assuming the height of the image to be 720, I took the average of left [719] and right [719] predictions to find the lane center. The position would then be the absolute difference between the two.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in code cell 25 in the function `final_image()`.  Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_output.mp4)



### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The most difficult problem I faced was to find the right color and gradient thresholds. After many trials, I got the color threshold to work but the gradient thresholds did not work. It introduced a lot of noise in the image. So I used only the color thresholds for my pipeline.

The pipeline could likely fail if the lanes have a very sharp turn or if any lane line is obscured by very dark shadows.

To make it more robust, I would like to experiment more with getting the gradient magnitude and  direction thresholds to work and test it on the challenge video.

