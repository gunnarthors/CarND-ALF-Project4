#**Advanced Lane Finding Project** #

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

[image1]: ./output_images/chessboard_corners.png "ChessboardCorners"
[image2]: ./output_images/distortedToUndistortedCB.png "UndistortedChessboardImages"
[image3]: ./output_images/distortedToUndistortedIMG.png "UndistortedExampleImages"
[image4]: ./output_images/thresholdtool.png "Threshold tool"
[image5]: ./output_images/plottedPoints.png "Source points"
[image6]: ./output_images/clrandgradApplied.png "Threshold applied and warped"
[image7]: ./output_images/histogram.png "Histogram"
[image8]: ./output_images/testresult.png "Result image"


##Camera Calibration##

The code for this step is contained in the second code cell of the IPython notebook located in "./CarND-AdvancedLaneFinding-P4.ipynb". Class name CalibrateCamera

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

![alt text][image1]

I then keep the output `objpoints` and `imgpoints` in the calibration object. To compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  The object has a function undistortImage which uses cv2.undistort function and returns undistorted image. Here below is example with undistorted chessboard images and test images.

![alt text][image2]

![alt text][image3]


##Image processing##

Third code cell of the IPython notebook located in "./CarND-AdvancedLaneFinding-P4.ipynb" is where I apply thresholds to create a binary image. I use gradient threshold, magnitude of gradient, direction of the gradient and color thresholding where we manipulate the S channel after changing the image into HLS color space. Output is binary image which I use.

It took time to find the right threshold parameters but a tool I found in the Udacity forum came in handy where I used slider with auto update on test image to see the result.

![alt text][image4]


Fourth code cell of the IPython notebook located in "./CarND-AdvancedLaneFinding-P4.ipynb" includes a warp() function which I use for perspective transform. The function takes in an image, source (src) points and destination (dst) points. I hardcoded the source points by marking in points I felt were in middle of the right and left lane line on test image. I then calculated that with the center of the image and found out acceptable source points (see image below). Destination points are also hardcoded and just from what I felt was good bird's eye view by trial-ing.

![alt text][image5]

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 249, 683      | 320, 720      | 
| 590, 450      | 320, 0        |
| 690, 450      | 960, 0        |
| 1031, 683     | 960, 720      |


Result after image processing and warping to bird'seye view.

![alt text][image6]


##Finding lanes##
Fifth code cell of the IPython notebook located in "./CarND-AdvancedLaneFinding-P4.ipynb" is where we search for lane lines.
Here I used the method of taking a histogram of the warped binary image.
With this histogram I am adding up the pixel values along each column in the image. In my thresholded binary image, pixels are either 0 or 1, so the two most prominent peaks in this histogram will be good indicators of the x-position of the base of the lane lines. I can use that as a starting point for where to search for the lines. From that point, I can use a sliding window, placed around the line centers, to find and follow the lines up to the top of the frame.
I then extract the line pixels and fit a second order polynomial using np.polyfit with the extracted pixels.
When this is done we add the information to the line object to keep a history. Next frame we don't neccisary have to go through the sliding window search again but instead use existing sliding window details and search in a margin around the previous line. If we are not sure that is a good line we go again into new sliding window search. How I decide when to go into sliding window search is by calculating the slope of left and right line and if the slope is within 25% of each other i'm satisfied else we go into sliding window search. I also check if radius of curvature on the left and right line are similar. 

If we are satisfied with the line it goes into history. If history is less than 4 we only use current lane as result but if more than 4 to max 12 we use the average of those lines to draw back into the plane as result. This is done to make the result more smooth.

And back into fourth code cell of the IPython notebook located in "./CarND-AdvancedLaneFinding-P4.ipynb" there we calculate the radius of curvature for the lanes. Function find_radius_of_curvature()
We hardcode values for meters per pixel.
ym_per_pix = 30/720 # meters per pixel in y dimension
xm_per_pix = 3.7/700 # meters per pixel in x dimension

There we also calculate lane offset from center of the camera. There we use same pixels per meter value. I set each line with distance from center of the camera and then in function offset_from_mid() I calculate the difference between left and right lanes. This information are then drawed to the result image.

Result on test image
![alt text][image7]


##Pipeline##

Sixth code cell of the IPython notebook located in "./CarND-AdvancedLaneFinding-P4.ipynb" is where the pipeline is. I have the same pipeline for single images and videos as it only needs an image to work. I have single left and single right line objects to work with through the process. I use them globally within the pipeline but pass the objects to the functions where the objects are updated with data.
Pipeline take in image.
* Undistort the original image
* Get binary image from the undistorted image
* Warps the binary image
* Finds lines in the warped binary image
* draws the lines into result image
* unwarps the result image with src as dst and dst as src
* combines the result image with the original undistorted image
* add info text to new image
* combines info image to the result image
* return result.

Here's a [link to my video](https://youtu.be/QJVTNWFOu0Y)


---

##Discussion##

I feel like the result are acceptable but not as good as I want it to be. I think it is mostly in the threshold parameters where I can tweak it to a much better result. In the project video it goes well through the first sunlight on the road but after the second it behaves a bit off but is quick to get back on right track. The car should not crash that wall there :)
 

