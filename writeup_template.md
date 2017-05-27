##Writeup Template
###You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

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

[image1]: https://github.com/toanhle93/P4/blob/master/example_images/undistort_output.PNG "Undistorted"
[image2]: https://github.com/toanhle93/P4/blob/master/test_images/test1.jpg "Road Transformed"
[image3]: https://github.com/toanhle93/P4/blob/master/example_images/binary_combo.PNG "Binary Mask"
[image4]: https://github.com/toanhle93/P4/blob/master/example_images/warped_test_lines.PNG "Warp Example"
[image5]: https://github.com/toanhle93/P4/blob/master/example_images/color_fit_lines.PNG "Fit Visual"
[image6]: https://github.com/toanhle93/P4/blob/master/example_images/Output.PNG "Output"
[video1]: https://github.com/toanhle93/P4/blob/master/project_video_out.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points
###Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
###Writeup / README

####1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!
###Camera Calibration

####1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first and second cell of the IPython notebook located in "./P4.ipynb"
 
I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection. This is all done within the first cell.

In the second cell, I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

###Pipeline (single images)

####1. Provide an example of a distortion-corrected image.
To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]
####2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of saturation, gradient, magnitude, and directional thresholds along with a mask in order to generate a binary image. This is located in cell number 5. Here's an example of my output for this step.

![alt text][image3]

####3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

I applied the perspective transform to the images in cell number 9 using my function `corners_unwarp()` which is defined in cell number 8. The `corners_unwarp()` function takes in the image and matrix and distortion coefficients. I chose the hardcode the source and destination points in the following manner:


```
src = np.float32(
    [[150+430,460],
    [1150-440,460],
    [1150,720],
    [150,720]])
dst = np.float32(
    [[offset1, offset3], 
    [img_size[0]-offset1, offset3], 
    [img_size[0]-offset1, img_size[1]-offset2], 
    [offset1, img_size[1]-offset2]])

```
This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 570, 460      | 100, 0        | 
| 150, 720      | 100, 720      |
| 1150, 720     | 1180, 720     |
| 710, 460      | 1180, 0       |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

####4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

After applying the binary mask to the perspective transform, I created a histogram using the pixel distribution by scanning through the windows. I then fit my lane lines with a second order polynomial.

![alt text][image5]

####5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I calculated the radius of curvature of the lane and the position of the vehicle with respect to the center in cell 18. I did this by using the code provided by Udacity. I chose a conversion factor for pixels to meters. Using that I fit new polynomials in meters to calculate the radii. In order to find the center, I took the average between the left and right line.

####6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in cell 19 by using functions `fillPoly()`, `polylines()`, and `putText`. I then added this onto the original image using `addWeighted()`. Here is an example of my result on a test image:

![alt text][image6]

---

###Pipeline (video)

####1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video.mp4)

---

###Discussion

####1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I came across problems such as determining which parameters were ideal for the binary mask. This pipeline will likely fail in cases with bad weather such as heavy rain or snow. The test video has a road in very ideal weather conditions with distinct lane lines displayed. I also had problems making a pipeline that would cater to varying conditions.
 
