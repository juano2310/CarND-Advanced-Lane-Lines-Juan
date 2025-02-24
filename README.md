## Advanced Lane Finding Project
[![Udacity - Self-Driving Car NanoDegree](https://s3.amazonaws.com/udacity-sdc/github/shield-carnd.svg)](http://www.udacity.com/drive)

The Project
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

---

[//]: # (Image References)

[image1]: ./examples/gopro.png "Undistorted"
[image2]: ./examples/undistorted.png "Example image undistorted"
[image3]: ./examples/binary.png "Binary Example"
[image4]: ./examples/top_down.png "Warp Example"
[image5]: ./examples/fitted.png "Fit Visual"
[image6]: ./examples/curvature.png "Curvature"
[image7]: ./examples/warp_back.png "Warp the detected lane boundaries back"
[image8]: ./examples/output_curvature.png "Output curvature"
[image9]: ./examples/formula.png "Equation"
[video1]: ./project_video_result.mp4 "Video"

## Implementation

On this writeup I will describe step by step all the pipeline used on the main IPython notebook located in `./Advanced Lane Finding Project.ipynb`.

### 1 - Camera Calibration

The code for this step is contained in the first code cell of the IPython notebook located in `./Undistor Images.ipynb` and also as part of the main IPython notebook located in `./Advanced Lane Finding Project.ipynb`.  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.

It is important to properly set the numbers of corners for `objp`. In this case the printed chessboard has 6*9 corners.

`objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image using the function cv2.findChessboardCorners() on all the images also specifying the number of corners. `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  

Finally, I applied this distortion correction to the test image using the `cv2.undistort()` function.

On the "Undistor Images" notebook I applied the camera calibration and undistort pipeline to my own images captured with GoPro Hero 4 and obtained this result:

![alt text][image1]

### 2 - Apply a distortion correction to raw images

Using the the values obtained by camera calibration function defined in the previous step I applied `cv2.undistort()` to an example image and this is the result:

![alt text][image2]

### 3 - Use color transforms, gradients, etc., to create a thresholded binary image

On the function `toBinary()` I used a combination of color and gradient thresholds to generate a binary image. The output is shown below. The final image is a combination of binary thresholding the R channel (RGB) and binary thresholding the result of applying the Sobel operator in the x direction on the undistorted image.

![alt text][image3]

### 4 - Apply a perspective transform to rectify binary image ("birds-eye view")

The code for my perspective transform includes a function called `warper()` which takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```python

offset = 100 # offset for dst points

# Source points
src = np.float32([[[ 610,  450]],
		[[ 680,  450]],
		[[ img_size[0]-300,  680]],
		[[ 380,  680]]])

# Result points        
dst = np.float32([[offset, 0],
		[img_size[0]-offset, 0],
		[img_size[0]-offset, img_size[1]],
		[offset, img_size[1]]])
```

This resulted in the following source and destination points:

| Source        | Destination   |
|:-------------:|:-------------:|
| 610, 450      | 100, 0        |
| 680, 450      | 1180, 0       |
| 980, 680      | 1180, 720     |
| 380, 680      | 100, 720      |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

In the example bellow I applied perspective transform to the binary image from step 3.

![alt text][image4]

### 5 - Detect lane pixels and fit to find the lane boundary

For this task I applied a convolution `find_window_centroids()`, which maximized the number of "hot" pixels in each window.

A convolution is the summation of the product of two separate signals, in our case the window template and the vertical slice of the pixel image.

You slide your window template across the image from left to right and any overlapping values are summed together, creating the convolved signal. The peak of the convolved signal is where there was the highest overlap of pixels and the most likely position for the lane marker.

Finally I combined the left and right points over the previous image.

![alt text][image5]

### 6 - Determine the curvature of the lane and vehicle position with respect to center

Using the left and right points found with the `find_window_centroids()` function I was able to define `curvature()` which returned the center of the image as well as the left and right Radius of curvature using the following equation:
![alt text][image9]

Here are the results plotted to an image:

![alt text][image6]

### 7 - Warp the detected lane boundaries back onto the original image

Once I had a good measurement of the line positions in warped space, it was time to project the measurement back down onto the road. Using the arrays called ploty, left_fitx and right_fitx obtained after fitting the lines with a polynomial of the binary image, I projected those lines onto the undistorted image.

![alt text][image7]

### 8 - Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position

For this I used text(), from the matplotlib librery, to write the numerical estimation of lane curvature and vehicle position on top of the image.

![alt text][image8]

---

### Pipeline (video)

The pipeline basically calls all the previously defined functions and a quick sanity check.

In addition I make sure that the lanes have a valid separation. If this fails I draw the previous estimation.

Also, for the video, I used cv2.putText() to write on top of the image instead of text(), from the matplotlib librery, used on the plotted example.

Here's a [link to my video result](./project_video_result.mp4) to the final video output.

---

### Discussion

For this project, I used a step by step approach which helped me understand each part of the pipeline. This was very helpful when things didn't go exactly as expected. Debugging was a lot easier and concepts became clearer. At the end I put together a pipeline by simply calling the functions I implemented along the way.

Ideas to improve this project:
* 1 - Average the lines to reduce sudden changes.
* 2 - Add birds-eye perspective to the final video.
* 3 - Test with more videos and images to find corner cases and improve the pipeline.
* 4 - Increase the "offset" of the destination points on the perspective transform function.
