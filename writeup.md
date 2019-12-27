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

[calibratedIm]: ./output_images/calibration_output/calibrated0.jpg "Calibrated Image Example"
[undistortIm]: ./output_images/undistorted_images/undistorted0.jpg "Undistorted Image Example"
[thresholdIm]: ./output_images/thresholded_images/thresholded0.jpg  "Thresholded Image Example"
[warpedIm]: ./output_images/warped_images/warped0 "Warp Example"
[windowedIm]: ./output_images/slidingsearches/slidingsearch0 "Fit Visual"
[polynomialIm]: ./output_images/windowed_images/window_image0 "Windowed Visual"
[outputIm]: ./output_images/visualized_lanes/visualized_lane0 "Output Image"
[outputVid]: ./project_video_output.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "./examples/example.ipynb" (or in lines # through # of the file called `some_file.py`).  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![Calibration Image Example][undistortIm]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images:

First I collected the `ret`, `mtx`, `dist`, `rvecs`, and `tvecs`, variables from my `calibrate_image()` function in order to undistort my image. I then ran `cv2.undistort` with my variables calculated before and returned the output of my undistorted image. It looks like this:
![Undistorted Image Example][undistortIm]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps at in my main .ipynb file).  Here's an example of my output for this step.

![Thresholded Image Example][thresholdIm]

I had multiple different thresholds for my image that when combined gave me the ideal output threshold. I used a sobelx, sobely, magnitude, direction, and color threshold on my image.

First, I will describe my method to create the sobelx and sobely thresholds. My function to create these is called calc_sobel_thresh. It calculates both of these instead of having two separate functions. It takes in the grayscale image and the two thresholds as inputs. First I start by calculating the sobelx threshold gradient. I use the `cv2.Sobel` function with inputs of the grayscale image, and `cv2.CV64F` follwed by `1, 0` which specifies that I want to take the derivative in the x direction. I then take the absolute value of this `sobelx` and name that `abs_sobelx`. I then get my `scaled_sobelx` by getting the 8 bit version of 255 times my `abs_sobelx` variable divided by the maximum value in `abs_sobelx`. Then I start to actually create the binary threshold. I create an sx_binary image of black pixels and overlay all the pixels from my `scaled_sobelx` image where the pixel values are in between my minimum threshold and maximum threshold. I then repeat this exact step for `0, 1` for the y direction and return both of these.

Second, I calculate the magnitude threshold of the x and y direction together. This function has a much smaller threshold than the sobel x and sobel y separately. First I take the sobelx and sobely in the their respetive directions. Then I get my abs_sobelxy by taking the square root of my sobelx and sobely squared and added to each other. Then I get my picture with the np.unit8 function. I then threshold the image as before by overlaying the two and taking the pixels inbetween my threshold.

Third I take the directional threshold. This threshold has a much larger sobel_kernel than the first two. I start out the same as the other two by calculating my sobelx and sobely. I then find the direction gradient by taking the arc tangent of the absolute values of sobely divided by sobelx. I take this gradient and overlay it to a black image and threshold between 0 and pi/2.

My last threshold is a color threshold. First I take the input image and convert it to the hls colorspace. I then select only the saturation channel since it is the best for finding lane lines. Finally I overlay the s channel on a black image with the pixel values from my s channel that are between my threshold of 170 and 250.

Finally in my main threshold_image method I call all of the above stated methods and combine all of those thresholds. My outputed image is my final thresholded image.


#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

For my birds_eye_transform I checked many sources and tested different combinations of source and destination points for my transform and finally landed on these hard coded points:

```python
    src = np.float32(
        [[220, 720],  # Bottom Left
         [570, 470],  # Top Left
         [725, 470],  # Top Right
         [1110, 720]]) # Bottom Right
        
    dst = np.float32(
        [[320, 720],  # Bottom Left
         [320, 0],  # Top Left
         [920, 0],  # Top Right
         [920, 720]]) # Bottom Right 
```

I then got my regular perspective transform and inverted perspective transform with the `cv2.getPerspectiveTransform` function. Then I found my `warped_image` with the `cv2.warpPerspective` function. I then returned all of these values as they are necessary later on in my pipeline.

![Warped Image][warpedIm]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I then created a histogram of my warped image. To make sure my lanes were most accurate I cut my image in half and use the bottom half. I then summed all the values of the pixels across that bottom half. That gave me a histogram that shows me where the lanes are most likely to start at.

This histogram was a great starting point for my sliding windows and identifying where the lane lines most likely are. I started by setting the amount of windows and margins I would use for them. Then I found all the nonzero pixels in the image on the left and right side of my image. After creating a few more variables I started to step through each window in my variable `nwindows`. I first started by creating some window boundaries for both my right and left window using my margin variable. After that I used cv2.rectangle to create a rectangle using those boundaries. I then identified all the white pixels in my picture that were within the current window and appended it to my `left_lane_inds and right_lane_inds` lists. If the amount of pixels I found for this window was greater than my `minpix` variable then I recentered my window on the mean position of the pixels. I then extracte the left and right x and y pixel positions and returned those and the output image with the windows appended. Below you can see the windowed image output from this function.

![Windowed Image][windowedIm]

A windowed image was not good enough though. I needed a function that found a polynomial line for the lane lines. My `fit_polynomial` function started by using my find_lane_pixels function to find the necessary variables. I then fit a second order polynomial useing np.polyfit and the variables to fit a polynomial to each line. I then plot points for each line and made colors for each then plotted them.

![Polynomial Image][polynomialIm]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

Even this was not good enough though and I had to created a different polynomial that searched around only around the polynomial when deciding what pixels belonged. This function is called `search_around_poly`. I start with a process very similar to my `find_lane_lines` and `fit_polynomial` functions. But created a margin to search around to find the next section of the polynomial. I then set the area to search and outputted all the pixels to the left_lane_inds and right_lane_inds lists. Then I extracted the pixels and fit a polynomial.

I then calculated the curvature of each lane using my `get_curvature_radius` function. This function first declares two variables called `ym_per_pix` and `xm_per_pix` for the meters per pixel in both the x and y direction. I then ran this calculation `((1 + (2*fit_cr[0]*y_eval*ym_per_pix + fit_cr[1])**2)**1.5) / np.absolute(2*fit_cr[0])` on my polynomial and returned that as my curvature.

Then in my original `search_around_poly` function I determined the car's offset from the center of the lane. I determined that the `lane_center` is halfway between the image and the car's position was halfway between its current `leftx` and `rightx`.  I then calculated the offset by subtracting the center of the lane from the car's position and converted that to meters. I then used the cv2.putText function to overlay all this information on my `detected_lanes` output image.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

This step was also implemented in my main .ipynb file in the `visualize_lane` function. This function took in many variables including the `binary_warped` image, `original_image`, `left_fitx`, `right_fitx`, `ploty`, and `Minv`. I started by creating a image to draw the lines onto. Then after casting my x and y points into a usable format I proceeded to use `cv2.fillPoly` and my x and y points to create a polynomial to be overlayed on the lane line. I then warped the perspective back to the original image's perspective and combined them using `cv2.addweighted`. This gave me the result below:

![Output Image][outputIm]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Creating the pipeline for my video was fairly simple as all I had to do was run all the functions on the video. I created a `find_lanes` function to do all of this automatically.

Here's a [link to my video result][outputVid]

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.

I ran into many issues throughout the project of having the wrong image size for the transformation I was trying to run on it. Whether I was in the wrong color space or my binary image outputs were wrong. I was able to solve these problems, but it took a bit of digging.

Also getting my pipeline to work for a video was a bit challenging. I had to look a lot up to get my video working.

I believe my pipeline does not work as well when it is very sunny and the color of the road is a light color. It does not detect the lanes super well. I believe this could be improved by altering the image before doing some of my thresholds. Possibly this would create a larger contrast between the lane lines and the road. 
