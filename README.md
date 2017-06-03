## Advanced Lane Finding
[![Udacity - Self-Driving Car NanoDegree](https://s3.amazonaws.com/udacity-sdc/github/shield-carnd.svg)](http://www.udacity.com/drive)

[//]: # (Image References)
[image1]: ./examples/undistorted_chess.png "Undistorted Chessboard"
[image2]: ./examples/undistorted_lane.png "Undistorted Lane"
[image3]: ./examples/thresholded_dir.png "Directional"
[image4]: ./examples/thresholded_red.png "Red"
[image5]: ./examples/warp_straight_line.png "Warped"
[image6]: ./examples/radius_of_curvature.png "RoC"
[image7]: ./examples/processed_image.png "Processed"
[image8]: ./test_images/test4.jpg "Test 4"

**Advanced Lane Finding Project**

In this project, our task is to write a software pipeline to identify the lane boundaries in a video. This is similar to [project 1](https://github.com/vishoo7/CarND-LaneLines-P1) but now we are building something more robust. In short we want to:
* take into account the radial and tangential distortions that occur to some extent with all cameras with lenses.
* explore other edge detection criteria, such as the magnitude of the gradient or the direction of the gradient
* better handle shadows and color alterations by looking at different color spaces
* user perspective transform to get a sense of the curvature of the lanes

The goals / steps of this project (taken from the template) are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

I would consider this README as a supplement to the [jupyter notebook](./p4.ipynb), which does a very good job of describing my thought process from start to finish.

## [Rubric Points](https://review.udacity.com/#!/rubrics/571/view)
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You are looking at it ;)

---

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

I compute the camera calibration matrix and distortion using chessboard images combined with an expectation of where the corners in the chessboard should be.  `cv2.findChessboardCorners` help us generate the expected corners and `cv2.calibrateCamera` give us the outputs used for calibration. I then used `cv2.undistort` to visualize what an undistorted version of a chessboard looks like.

![alt text][image1]

---

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.
I explored three Sobel thresholding techniques. One in the X direction, one using magnitude, and one with directional information. I used `cv2.Sobel` and then converted into a matrix with 1s where a certain threshold was attained and 0 otherwise. An example of directional thresholding:

![alt text][image3]

As you can see the lines  between .7 and 1.2 radians (40 degrees to 69 degrees) are accentuated. The remaining Sobel thresholding examinations are in the jupyter notebook.

For color transformations I used `cv2.cvtColor` for light and saturation. The red channel can be directly grabbed from the image matrix. An example of the red channel thresholded:

![alt text][image4]


#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

I described a perspective transform using `cv2.getPerspectiveTransform`. For the inputs to this function I eyed where the points in the were on the and where they should be. From the output I used `cv2.warpPerspective` to transform the image. Here's what a straight line looks like after a perspective transformation.

![alt text][image5]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

After undistorting and thresholding the original image into a binary image I created a histogram. From the histogram I use the sliding window technique explained in lesson 33. What this does is divide the pixels to left lane and right lane. From the x and y values from the lanes we can get a fit using `np.polyfit`. This gives us a coefficient vector representing the A, B and C in f(y)=Ay^2 + By + C. Subsequently we can divide the binary image into left and right lane pixels by searching the binary image within a margin of the previous left and right coefficients. This saves us some computation.

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

We get the curvature by using calculus:

![alt text][image6]

The A and B above are taken from the coefficient vector described in the previous set.

To get to from pixel space to meters we convert by multiplying with the following facts...
meters per pixel in y dimension:
ym_per_pix = 30/IMG_HEIGHT

meters per pixel in x dimension:
xm_per_pix = LANE_WIDTH/700


#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.


![alt text][image7]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

[Video](./project_video_processed.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

- The perspective transform was challenging. I transformed an image with straight lines where they appear vertical/parallel, but it was by no means perfect.

- I suspect a lot more thought could have gone into sanity checking. Just throwing out bad curvature lines or lines far from parallel might not be enough.

- I had a problem fitting a polynomial for the right lane in one of the test images:
![alt text][image8]
This was the case because there are not many pixels for the bottommost part of the right lane. This was a problem without context (i.e. a single image), but given a video and a list of recent coefficients this should be manageable. We could use the average of the past 10-20 frames. However, if we had large portions of the road without long/clear lines we could struggle to fit a polynomial.

- Beyond problems from the length of the lane lines we could get into issues if lane lines are different shades, if there's rain, or if it's very dark, just to name a few different conditions. For a production application we would really need to test our color transformations and relevant gradient methods for other conditions. I suspect there are many more spaces/methods that would improve our pipeline.

- Although my pipeline produced a satisfactory processed video, I am unsure of how performant this would be if it were generalized to real-time. The sliding window method, if we had to find lane lines from scratch repeatedly, could require too much processing.

- Going forward I have decided to create classes/functions in separate files then include them into my jupyter notebook accordingly. This separation is cleaner and better for future reuse. I could then focus on writing great code and focus on good presentation in separate contexts.
