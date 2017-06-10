## Writeup/Readme for Advanced Lane Finding Project


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

[image1]: ./pictures_for_writeup/Undistort_Example.png "Undistorted"
[image2]: ./pictures_for_writeup/Undistort_Example2.png "Road Transformed"
[image3]: ./pictures_for_writeup/Combined_binary.png "Binary Example"
[image4]: ./pictures_for_writeup/Warped.png "Warp Example"
[image5]: ./pictures_for_writeup/Binary_warp.png "Binary_warp"
[image6]: ./pictures_for_writeup/Binary_warped.png "Binary_warped"
[image7]: ./pictures_for_writeup/hist.png "hist"
[image8]: ./pictures_for_writeup/finding_points.png "finding points"
[image9]: ./pictures_for_writeup/Fit.png "Fit lines"
[image9]: ./pictures_for_writeup/Final.png "Fit lines"

[video1]: ./white_out.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!
The Python notebook contains two parts. First is for data exploration and the other part is for defining functions for Pipeline function. In this Writeup, I mainly use the first part, which is data exploration to explain the process I did the project. However, the second part is cleaner.

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the second code cell of the IPython notebook located in "Advanced_Lane_Detection-Camera Calibration and Data Exploration.ipynb".

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of HLS color, gradient thresholds(x and y direction separately and magnitude) to generate a binary image.

I define some helper functions for helping picking threshhold and visualize therm (they are defined in code section 6, 7 and 8).

Using those functions, I pick my threshholds within code section 9, 10, 11, 12, 13, 14 (including Sobel X and Y threshholds picking, Gradient Magnititude threshhold picking, Gradient direction threshhold picking and HLS threshhold picking)

Then I combined the result in code section 16.
Here's an example of my output for this step.  (note: this is actually from one of the test images)

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform (exploration) is in code section 17, 18 and 19. (You can find defined function in `warp()` function in second part of the notebook)

I first defined 4 source points from straight test picutre and defined the 4 distination points. Using those two sets of points and function `cv2.getPerspectiveTransform(src, dst)` to find warp matrix M and inverse warp matrix Minv. Then I use `cv2.warpPerspective` to do perspective transform

I chose the hardcode the source and destination points in the following manner:

```python
src = np.float32([[753.5, 490.5],
                  [532.5, 490.5],
                  [256.5, 674.5],
                  [1051.5, 674.5]])


dst = np.float32([[930, 300],
                  [260, 300],
                  [260,700],
                  [930, 700],
                  ])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 753.5, 490.5      | 930, 300      | 
| 532.5, 490.5     | 260, 300      |
| 256.5, 674.5     | 260,700      |
| 1051.5, 674.5      | 930, 700        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]
![alt text][image5]
![alt text][image6]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

1. Finding line starting point (warped binary picture). Using warped binary picture's lower half's nonzero points distribution. (code section 20 and 21)
![alt text][image7]

2. Finding line points using sliding window search. (code section 21)
![alt text][image8]

3. Fit lines using points found before. (code section 21)
![alt text][image9]

For defined function, please look at the second part of the notebook.

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this using the fitted polynomial function and apply curvature function and also the assumption of 30/720 meters per pixel in y dimension and 3.7/700 meters per pixel in x dimension. You can find this function in code section 29 in the notebook.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I used the `Pipeline()` to get his step (code section 32).  Here is an example of my result on a test image:

![alt text][image10]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./white_out.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The biggest problem is detecting lines from various situations. I spend quite some time to test the binary threshholds to get the best result of first finding lane lines. However, it is still problematic when I try the chanllenge videos, not even menthion the situation that when it is raining or snowing. I want to spend more time on this.

Another problem is warp/unwarp picture and calculating related curvature and postion. Since now I don't have data to calibrate meter per pixle.