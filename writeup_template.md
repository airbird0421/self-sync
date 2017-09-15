## Writeup Template

### You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

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

[image1]: ./examples/undistort_output.png "Undistorted"
[image2]: ./test_images/test1.jpg "Road Transformed"
[image3]: ./examples/binary_combo_example.jpg "Binary Example"
[image4]: ./examples/warped_straight_lines.jpg "Warp Example"
[image5]: ./examples/color_fit_lines.jpg "Fit Visual"
[image6]: ./examples/example_output.jpg "Output"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "./examples/example.ipynb" (or in lines # through # of the file called `some_file.py`).  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image. I use function `cv2.xxxx` to detect the chessboard corners in an image. `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection. For all the images provided in this project in folder "./camera_cal", except the first 3 images, all the other 17 images have their corners detected successfully. So I just use the first 3 images as the test images to verify the camera calibration result.  

I used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this pipeline, I will start with a distortion-corrected image as blow:
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps at lines # through # in `another_file.py`).  Here's my output for this step. More specifically, I used gradient thresholds of X-direction, which can detect white lines better. I used S-channel and H-channel to do the color thresholding.

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warper()`, which appears in the 3rd code cell of the IPython notebook.  The `warper()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner: open the undistored image, which contains the straight lane lines, in an image viewer software, which can show the mouse position in pixel. Move the mouse to get coordinates of totally 4 points on the left and right lane lines such that those 4 points corresponds to a triangle in the transformed image. 

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 585, 460      | 280, 0 | 
| 203, 720      | 1000, 0  |
| 1127, 720     | 1000, 720 |
| 695, 460      | 280, 720 |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I used the sliding window search methord to identify the pixels that potentially belong to the lane lines. Basically, first determine the base position of the lane lines by summing the columns on the bottom half of the warped image, and find out the two peak positions, which serve as the base positions of left and right lane lines. Secondly, sliding the window vertically, and find out all pixels in those windows, adjust the window center based on the mean of the X-value of the pixels. Thirdly, with all potential pixels identified, I fit my lane lines with a 2nd order polynomial kinda like this:

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in lines # through # in my code in `my_other_file.py`. The radius calculation is based on a formula, using the coefficients got from step 4 above. The distance between the lane center and the image center is used to calculate the position of the vehicle with respect to lane center. I used an approximate 3.7m as the lane width to conver the pixel distance to real world distance.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in lines # through # in my code in `yet_another_file.py` in the function `map_lane()`.  Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further. 

The camera calibration and perspective transformation steps are kind of standard. The threshoding step is where difference can be made. At the beginning, I just used the S-channel thresholding, which can detect yellow lines perfectly, and X-direction gradient threshoding, which can detect white lines better. I combined these two together as the basis for further processing. But I noticed that though S-channel can detect yellow lines perfectly, but it can't eliminate the shadows on the road, which gave me some trouble on the shadowed area. Then I used H-channel thresholding, which can show the yellow lines, and also effectively eliminate the shadows from the binary result. Combining them together, the shadow can be depressed well.

Another issue I hit is that at the beginning, when I define the perspective transformation, the y-direction distance between the points I selected is a little bit smaller. The result is that on the warped image, fewer part of the lane lines are represented, which just decreases the accuracy of the polynomial fit and radius calculation. I just took the points again, with as farther Y-distance as possible, which effectively improved the stability and accuracy of the pipeline.

The third thing is that the video clip API feeds the pipeline with RGB image, but pipeline expects BGR since I used cv2 when I made the pipeline ready. I found this when I tried to debug some video frames that gave me trouble.

With those pointed above, the pipeline can generate an acceptible video result. I did try to make it better, using techniques like smoothing. I just tried to use the average fit parameters in the past several iterations, and also tried to use the most recent good fit result when I detected something potentially bad, but didn't see obvious improvements. So I just removed those code.

One potential issue with the method to calculate the radius is that we are using too short of a road sample to estimate the radius. For example, the Y-direction distance in real worl mapping to the warped image is about 30m where we do the polynormial fit, as mentioned in the introduction part. With an 1Km radius, 30m is only about 1/200 of the circle, or less than 2 degrees. With so small a sample, we can't expect much accuracy for the calculation, especially with dashed lines in both sides. 

The other issue with this method is the when doing perspective transformation, the farther away the object, the bigger impact it has on the warped image. For example, a very short dot far away will become a much longer line in the warped image, but the lines near the car will be compressed. But the farther away, the more noises. This potentially brings much uncertainty to the result.

Dashed lines seem more challenging because it may present a situation, where only few pixels for the line were captured, plus the noises, the calculation may become much inaccurate. One way I can think of to deal with it, is that when we detect few pixels in one side, but much much more pixels in the other side, we can just rely on one side with more pixels, and add an offset to it as the result of the side with much less pixels. The other way is that, use the pixels in the previous several iterations, to fit the polynomial. but I'm not sure how accurate it is since the car is moving, the position is chaning. With an 120KM/hour speed, suppose the camera frame rate is about 30/s, then distance between each frame is about 1m. Maybe it can work.









