## Advanced Lane Finding

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

[image1]: ./examples/calibration.png "Calibration"
[image2]: ./examples/undistorted.png "Undistorted"
[image3]: ./examples/binary.png "binary outputs"
[image4]: ./examples/warped.png "warped"
[image5]: ./examples/polyfit.png "Fit Visual"
[image6]: ./examples/pipeline.png "Output"
[video1]: ./project_output.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points
###Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
###Writeup / README

####1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!
###Camera Calibration

####1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.


The code for this step is contained in the first code cell of the IPython notebook located in "p4.ipynb".
I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image. Thus, objp is just a replicated array of coordinates, and objpoints will be appended with a copy of it every time I successfully detect all chessboard corners in a test image. imgpoints will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.
I then used the output objpoints and imgpoints to compute the camera calibration and distortion coefficients using the cv2.calibrateCamera() function. I applied this distortion correction to the test image using the cv2.undistort() function and obtained this result:

![alt text][image1]

###Pipeline (single images)

####1. Provide an example of a distortion-corrected image.
As the calibration is completed, I use the image points and object points to process and generate an undistorted image:
![alt text][image2]
####2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

Cell 3 lists all the various methods used to generate X & Y Sobel gradient thresholds, Magnitude and Directional thresholds, and the HLS L & S channel color models. The outputs for all these methods are shown after the cell "6. Pipeline for Test Images".
After looking at the various outputs, and after tuning kernal sizes, I was able to obtain a better binary image by combining the outputs from the L and S channel models. Later, I apply a region mask to generate an binary image with only the field of interest (Road surface).

![alt text][image3]

####3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `corners_unwrap()`, which appears in the 4th code cell of the IPython notebook.  The `corners_unwrap()` function takes as inputs an image (`img`).   I took a guesstimate of the source pairs for the input image and the estimated destination points for the transformed image:

```
src = np.float32([(570,460),
                (700,460),
                (250,680),
                (1050,680)])
  dst = np.float32([(450,0),
                (w-450,0),
                (450,h),
                (w-450,h)])

```
This resulted in the following source and destination points:

| Source        | Destination   |
|:-------------:|:-------------:|
| 570, 460      | 450, 0        |
| 700, 460      | w-450, 720      |
| 250, 680     | 960, h      |
| 1050, 680      | w-450, h        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

####4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

After generating a transformed binary image, the image is passed to the "slidingwindow" method. This method first generates a histogram of the white pixels in the binary image. As instructed in the lesson, the midpoint is used to separate the left and right collections of points to identify the lanes. This points are passed to a polyfit function to generate a 2nd order equation. I understand that once the starting pixels have been established, we can use the previous points to generate the rest of 2nd order equation. This is an enhancement that I plan to make in the near future. For sanity checks, I store the left and right lane points in global variables to be referenced again when points are not detected in a particular frame.:

![alt text][image5]

####5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

As suggested in the lesson, I reference the calculation for the radius of curvature from the following link: http://www.intmath.com/applications-differentiation/8-radius-curvature.php
The position is calculated by taking the average of left and right halves of the boundaries in the image and subtracted from the mid of the image. Sanity checks are in place to make sure that lane points are available to generate curvature. If not, previous frame values are used.

####6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.


The 'pipeline' function located in code cell 7 takes './test_images/test3.jpg' to generate binary images from the various methods shown below:

![alt text][image6]

---

###Pipeline (video)

####1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video.mp4)

---

###Discussion

####1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

As noticed in the video, the pipeline seems to do a decent job over curves, straight lines, and surface changes. Its only at the near end of the video with a combination of surface changes and shadows that the pipeline shows some jitters. I believe these issues can be addressed by incorporating better sanity checks such as maintaining a minimum distance between the two lane lines, using previous best fits to compensate for bad data, and also applying smoothening. I also believe the pipeline would raise issues when there are lane markings present in the middle of the lane (Such as "Straight only" arrows, HOV signs). I will continue to work on the pipeline to implement these optimizations.
