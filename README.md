## Advanced Lane Finding Project

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

[image1]: ./output_images/undistort_output.png "Undistorted"
[image2]: ./test_images/test1.jpg "Road Transformed"
[image3]: ./output_images/binary_combo.png "Binary"
[image4]: ./output_images/warped_straight_lines1.png "Warp Example"
[image5]: ./output_images/warped_straight_lines2.png "Warp Example 2"
[image6]: ./output_images/histogram.png "Histogram"
[image7]: ./output_images/sliding_windows.png "Sliding Windows"
[image8]: ./output_images/prior.png "Search from Prior"
[image9]: ./output_images/color_fit_lines.jpg "Polyfit"

[image100]: ./output_images/singe_image_output.png "Output"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf. 

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "./AdvancedLanesLines.ipynb".

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (in the 5th code cell of the IPython notebook `AdvancedLanesLines.ipynb`).  Here's an example of my output for this step.

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `birds_eye()`, in the 6th code cell of the IPython notebook AdvancedLanesLines.ipynb).  The `birds_eye()` function takes as input an image (`img`) and return the warped image along with the transformation M that was used.  I chose the hardcoded the source and destination points in the following manner:

```python
src = np.float32(
        [[200, height],  #bottom left
         [width - 200, height], #bottom right
         [685, 450], #top right
         [594, 450]]) #top left
    # define 4 destination points
    dst = np.float32(
        [[200, height],  #bottom left
         [width - 200, height], #bottom right
         [width - 200, 0], #top right
         [200, 0]]) #top left
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 200, 720      | 200, 720        | 
| 1080, 720      | 1080, 720      |
| 685, 450     | 1080, 0      |
| 594, 450      | 200, 0        |

I verified that my perspective transform was working as expected by taking the two straight line images `test_images/straight_lines1.jpg` and `test_images/straight_lines2.jpg` and converting them to birds eye view, making sure that their lanes appear parallel.

![alt text][image4]
![alt text][image5]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

a. In the 7th code cell of the IPython notebook, we define a function to find the histogram peaks of the bottom half of the image. Having a spike in the histogram means that this is where most vertical line points are concentrated so this is probably the base of the line towards the car.

![alt text][image6]

b. In the 9th code cell of the IPython notebook we define a set of functions that will help us with line detections. Starting with the positions detected from the histogram we follow the lines using the sliding window technique (`sliding_windows_search()`). This technique is used on the first frame of a video. Within this step we perform a sanity check to make sure that recongized lane width is reasonable.

![alt text][image7]

c. After we have a set of points we can polyfit a polynominal using `fit_polynomial()` which returns us the polynomial of each lane. Within this step we perform a sanity check to make sure that the second degree parameters of both fits are not too different.

![alt text][image9]

d. On consequent frames we can use a margin around the previously detected lines in order to speed up the tracking process. Instead of searching around the whole image we only search within a margin around the previously detected lines using `search_around_poly()`. In order to do assist us in this, we defined in the 8th code cell of the IPython notebook, a class called `Line` which maintains the characteristics of each line detection.

![alt text][image8]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

Function `measure_curvature()` of the 9th code cell, uses the mathematical formula described [here](https://www.intmath.com/applications-differentiation/8-radius-curvature.php), to calculate the curvature of the lanes at their maximum y which is the bottom of the screen. The curvature is initially measured in pixel units and then converted to meters.

The position of the vehicle is calculated using `center_offset()`. This function finds the center of the image `frame_width / 2` and from that substracts the x value of the central point between the lanes `(right_fitx - left_fitx)/2 + left_fitx`. Again the value is converted from pixels to meters.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

The 10th code cell I implement the final pipeline for a single image in the function `process_image()`.  Here is an example of my result on a test image:

![alt text][image100]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./output_videos/project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

What worked:
  - Sanity check and smoothing worked nicely to improve the end result
  - I had to narrow down the margin when searching around lines of previous frames
What could potentially not work and can be improved:
  - In general, I went for a simpler approach, as I wanted to see the real power of the pipeline. I didn't make use of all the properties included in the Lane class. I did really sublte smoothing taking only into account the last 2 successfully recognized frames.
  - Sometimes one of the two lanes is not detected due to lack of enough points. The non found lane could be deducted from its found counterpart, given that lanes are always parallel and with the same distance between them. The distance can also be dynamically calculated from the previous frames.
  - The previous scenario is particularly evident when the curvature is too high and on of the lanes is "lost" to the outer side of the image. The previously described technique would probably work for this scenario as well
