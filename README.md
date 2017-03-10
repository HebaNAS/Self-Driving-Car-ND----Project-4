# Self-Driving-Car-ND----Project-4
Advanced Lane Finding
---------------------
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

[image1]: ./output_images/corners13.jpg "Detecting Corners on a Chessboard"
[image2]: ./test_images/test6.jpg "Road Transformed"
[image3]: ./output_images/undist5.png "Undistorted Example"
[image4]: ./output_images/unmod_threshold5.png "Binary, Thresholded Example"
[image5]: ./output_images/warped5.png "Warped Example"
[image6]: ./output_images/c_5.png "Traced Lane Lines Example"
[image7]: ./output_images/img5.png "Final Output Example"
[video1]: ./project_video_output.mp4 "Video"

---
###Camera Calibration
The code for this is found in the first two cells in the jupyter notebook.

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result:

I saved the output for all images with detected corners in the output folder as `./output_images/corners*.jpg`.

![alt text][image1]

---
###Pipeline (single images)

####1. Provide an example of a distortion-corrected image.
To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]

Code can be found in `cell 5`.

I used the camera calibration matrix saved in `calibration.p` to perform corrections on images. I used `cv2.undistort` to perform the undistortions.

I saved the output for all images with detected corners in the output folder as `./output_images/undist*.png`.

Here's an example of my output for this step.

![alt text][image3]

####2. Color transforms and gradients or other methods to create a thresholded binary image.
Code can be found in `cell 6`

First I applied a mask to the undistorted image to get an area of interest, which is the road only, and masked the upper half of the image and scenery. I used several test and trials to get the trapizoid containing the region of the image I wanted. This is represented in the vertices array which containes the coordinates of the trapizoid.

I defined 3 helper functions, `abs_sobel_thresh()` that calculates directional gradient by applying the Sobel operator, `mag_thresh()` that calculates the threshold magnitude, and `dir_threshold()` that calculates the direction of the threshold.

I used a combination of color and gradient thresholds to generate the binary image.

I created a function `combined_thresh()` to apply thresholds. First I applied Gaussian blur, then I converted the images to HLS and separated the channels to get the `S` channel and then I grayscaled the image. I applied the Sobel operator on the `S` channel by running the `abs_sobel_thresh()` function. I used a min threshold of 10 and a max of 255 on the `x` orientation, and a min of 60 and a max of 255 on the `y` orientation. Then I caculated the magnitude and the direction of the gradient by running `mag_thresh()` and `dir_threshold()`, respectively. At the end I combined the threshold information. The values chosen for thresholding were based on multiple different trials until I got satisfactory results. The output is returned as a binary image.

I saved the output for all images with detected corners in the output folder as `./output_images/unmod_threshold*.png`.

Here's an example of my output for this step.

![alt text][image4]

####3. Perspective transform.
Code can be found in `cell 57`

The code for my perspective transform includes a function called `perspective_transform()`.  The function takes as inputs an image (`img`).
I hardcoded the values of src and dst points. These values were added based on some trials and errors.

I calculated `src` and `dst` points as follows

```
src = np.float32([[200, 720], [1100, 720], [595, 450], [685, 450]])
dst = np.float32([[300, 720], [980, 720], [300, 0], [980, 0]])

```
This resulted in the following source and destination points:

| Source        | Destination   |
|:-------------:|:-------------:|
| 200, 720      | 300, 720        |
| 1100, 720      | 980, 720      |
| 595, 450     | 300, 0      |
| 685, 450      | 980, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

I saved the output for all images with detected corners in the output folder as `./output_images/warped*.png`.

Here's an example of my output for this step.

![alt text][image5]

####4. Identifying lane-line pixels and fit their positions with a polynomial.
Code can be found in `cell 13`

I created a function called `find_lanes()`, it is using histogram peaks in the lane lines in many sliding windows on the image. It accepts a binary image as input, which is an output of the undistortion, warping, thresholding pipeline. By taking a histogram of the image along small window sizes we should see some peaks in pixel values which are indicative of the lane lines. Once we find where a lane line starts in the first sliding window at the bottom of our image, we can use that to find how the lane line continous as we move to the second sliding window. I created `leftx_current` and `rightx_current` to hold the positions of lane lines starting points to be updated for each window. I hardcoded a `margin` value around the found positions to be considered as our lane lines. I also hard coded a `nwindows` value that divides the image height into slices for smaller windows. I then looped through the windows and detected possible lanes through histogram peaks in each window, adding indices values to `left_lane_inds` and `right_lane_inds` arrays.
 After finding the pixels of each lane, we can fit a polynomial to these pixels. This is done in fucntion `fit_lines()` which takes the outputs of `find_lanes()` function and returns `x` and `y` values for plotting the lane lines.

I saved the output for all images with detected corners in the output folder as `./output_images/c_*.png`.

Here's an example of my output for this step.

![alt text][image6]

####5. Calculating the radius of curvature of the lane and the position of the vehicle with respect to center.

Code can be found in `cell 14`

I created a function called `find_curvature()`, which takes the outputs of `find_lanes()` and returns the left and right curvatures in meters.
The function converts the `x` and `y` pixel coordinates of the lane lines to world space coordinates. As per US regulations, the lane is about 30 meters long and 3.7 meters wide. Using the image's height and width we can compute meters per pixel in both `x` and `y` dimensions. By fitting a polynomial to our computed metric coordinate values, we can determine the equation of each lane line in world space coordinate. I calculated the radius of curvature for each lane line using the equation provided in the classroom.

Another function I created was `car_offset()`, it returns the position of the car with respect to the center in meters. We can assume that the camera is mounted at the center of the car therefore the midpoint of the image is in fact the center of the vehicle. I calculated the difference between the leftmost point of the left curve and the rightmost point of the right lane and then the offset is calculated by subtracting half the width of the image from that difference. I converted the pixel space to worlds space by multiplying the result by meters-per-pixel in `x` dimension.

####6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

Result can be found in `cell 48`

!I saved the output for all images with detected corners in the output folder as `./output_images/img*.png`.

Here's an example of my output for this step.

![alt text][image7]

---

###Pipeline (video)

####1. Link to the final video output.

Here's a [link to my video result](./project_video_ouput.mp4)

---

###Discussion

####1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I found that my implementation for the pipeline works well on all images, works fairly well on the video, it becomes wobbly, and has trouble finding the exact curvature for far lines. It mostly happens when the frame contains unfavorable lighting conditions or some shadows on the road. I think this can be solved if I did some more tweaking for color thresholding.
