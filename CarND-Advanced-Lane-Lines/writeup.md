## Writeup 


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

[image1]: ./output_images/undist_cal_img.png "Undistorted calibration image"
[image2]: ./output_images/undistort_img.png "Undistorted test image"
[image3]: ./output_images/warped_img.png "Warped road test image"
[image4]: ./output_images/sobel_gradx_img.png "Applied sobel operator gradient on x"
[image5]: ./output_images/sobel_grady_img.png "Applied sobel operator gradient on y"
[image6]: ./output_images/mag_of_grad_img.png "Magnitude of the gradient"
[image7]: ./output_images/dir_of_grad_img.png "Direction of the gradient"
[image8]: ./output_images/hls_select_img.png "H-channel of the HLS color space"
[image9]: ./output_images/combined_threshs_img.png "Combined thresholds"
[image10]: ./output_images/warped_binary_img.png "Warped binary image"
[image11]: ./output_images/hist_img.png "Histogram of the binary image"
[image12]: ./output_images/sliding_win_img.png "Binary image with sliding windows"
[image13]: ./output_images/search_around_poly_img.png "Search around poly"
[image14]: ./output_images/drawing_img.png "Fit visual"
[image15]: ./output_images/display_info_img.png "Display curvature and offset on visual"
[video1]: ./project_video_output.mp4 "Video pipeline output"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README


### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the second code cell of the IPython notebook named "Advanced_Lane_Finding.ipynb"

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undist()` function and obtained this result: 

![Undistorted calibration image][image1]


### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to the "test1.jpg" image found in the "./test_images/" folder. After using the undistort() function, the undistorted test image results like this:
![Undistorted test image][image2]

The test image is converted from RGB to grayscale and using the camera calibration and distortion coefficients obtained above in the Camera Calibration procedure, the cv2.undist() function is used.


#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

Code cells 6-11 show how I used gradient and color spaces to create a thresholded binary image.

First, I used the Sobel operator to find the gradient on the x and y axis:
![Applied sobel operator gradient on x] [image4]
![Applied sobel operator gradient on y] [image5]

Second, I applied a threshold to the overall magnitued of the gradient, in both x and y.combination of color and gradient thresholds to generate a binary image (thresholding steps at lines # through # in `another_file.py`).  Here's an example of my output for this step.  
![Magnitude of the gradient] [image6]

Third, I computed the direction, or orientation, of the gradient:
![Direction of the gradient] [image7]

Fourth, I used the HLS color space and applied a threshold to the S-Channel: 
![H-channel of the HLS color space] [image8]

Finally, I combined all the above selection thresholds: 
![Combined thresholds] [image9]


#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warp()`, which appears in the fifth code cell of the IPython notebook.  The `warp()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose to hardcode the source and destination points by eyeballing the measurements:
        
    src=np.float32(
            [[280,  700],  # Bottom left
             [595,  460],  # Top left
             [725,  460],  # Top right
             [1125, 700]]) # Bottom right  

    dst=np.float32(
            [[250,  720],  # Bottom left
             [250,    0],  # Top left
             [1065,   0],  # Top right
             [1065, 720]]) # Bottom right

The "test1.jpg" was tested using the warp() function and the result after applying the perspective transform can be seen in this image:
![Warped road test image][image3]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I took a histogram of the binary image to locate the lane lines:
![Histogram of the binary image][image11]

Then through sliding windows I found the left and right lane pixels:
![Binary image with sliding windows][image12]

Having extracted the left and right line pixel positions, I fit my lane lines with a 2nd order polynomial.  A polygon was then drawn to show the selection window: 
![Search around poly][image13]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I implemented the radius of curvature in the function called "measure_curvature_real()".  The position of the vehicle with respect to the center was implemented in the function "lane_center_offset()".

To calculate the radius of curvature I first find the lane pixels and then fit a second order polynomial to each.  I then plugged the coefficients of the second order polynomial on the Radius of Curvature equation provided in the lesson.

To get real values a conversion from pixel to meters was used.

For the position of the vehicle with respect to the center, the middle horizontal position of the frame was used to substract the actual car position from it. The car position with respect to the lane was found by adding calculated left and right x polynomial values and dividing by 2.


#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in the function `drawing()`.  Here is an example of my result on a test image:

![Fit visual][image14]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

The video outout form the pipeline is here:
![video pipeline output](video1)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I used all the same techniques used in the lessons for my project.  Basically, the camera was camera was calibrated.  The frames/images were then undistorted.  The HLS color space and other gradients techniques such as Sobel were used to generate a binary image.  A perescpective view of the image was used to calculate the radius of curvature.  The image was then unwarped to its original shape. 

The pipeline might fail with different road videos.  The reason is different shades of the pavement.  The thresholds would need to be more robust to work with different road lighting, shadows, road conditions, wet road, etc.  
