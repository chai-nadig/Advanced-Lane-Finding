## Advanced Lane Finding

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

[undistorted_calibration1]: ./output_images/calibration1.jpg "undistorted_calibration1.jpg"
[calibration1]: ./camera_cal/calibration1.jpg "calibration1.jpg"
[test2]: ./test_images/test2.jpg "test2.jpg"
[test2_undistorted]: ./output_images/test2_undistorted.jpg "test2_undistorted.jpg"
[test2_binary]: ./output_images/test2_binary.jpg "test2_binary.jpg"
[test2_warped]: ./output_images/test2_warped.jpg "test2_warped.jpg"
[test2_linepixels]: ./output_images/test2_linepixels.jpg "test2_linepixels.jpg"
[test2_linepixels_fit_overplotted]: ./output_images/test2_linepixels_fit_overplotted.jpg "test2_linepixels_fit_overplotted.jpg"
[test2_lane_detected]: ./output_images/test2.jpg "test2_lane_detected.jpg"
[image6]: ./examples/example_output.jpg "Output"
[video1]: ./project_video.mp4 "Video"

### Camera Calibration

1. The code for this step is contained in the 3rd and 4th code cell of the IPython notebook.
2. The object points are the same for each calibration image.
3. The image points are gotten by the `find_corners` function.
4. After the calibration, image from the camera are undistorted using `undistort_image` in the 6th code cell.

| **calibration1.jpg** | **undistorted_calibration1.jpg** |
|:--------------------:|:--------------------------------:|
|![alt text][calibration1] | ![alt text][undistorted_calibration1]


### Pipeline (single images)

#### 1. Distortion corrected image

* Here's an example test image undistorted after calibration.

| **test2.jpg** | **test2_undistorted.jpg** |
|:--------------------:|:--------------------------------:|
|![alt text][test2] | ![alt text][test2_undistorted]

#### 2. Color transforms and gradients thresholding

1. For color thresholding, images were converted from RGB space to HLS space
2. The S-channel was extracted and min and max threshold of 170 and 255 was applied.
3. For gradient thresholding, images were converted to grayscale and sobel operator was applied along the x-axis.
4. All gradients with a min and max threshold of 20 and 100 were retained.
5. The two thresholded images were then combined to render one binary image.
6. Code for this is contained in the 7th code cell.

| **test2_undistorted.jpg** | **test2_binary.jpg** |
|:--------------------:|:--------------------------------:|
|![alt text][test2_undistorted] | ![alt text][test2_binary]


#### 3. Perspective transform

1. This is the mapping between source and destination points used to perform perspective transform

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 230, 720      | 350, 720      | 
| 1050, 720     | 940, 720      |
| 700, 450      | 1100, 0       |
| 580, 450      | 230, 0        |

2. The code for perspective transform is in code cell 8.
3. The result of perspective transform on the original image and the binary thresholded image look like this -  

| **test2.jpg** | **test2_warped.jpg** | **test2_linepixels.jpg** |
|:-----------------:|:------------------------:|:----------------:|
|![alt text][test2] | ![alt text][test2_warped]|![alt text][test2_linepixels]


#### 4. Identify lane-line pixels and fit a polynomial

1. The lane pixels in the binary warped image `test2_linepixels.jpg` were identified using a histogram and moving window algorithm.
2. The code for this is in the 11th code cell under the `find_lane_pixels` function.
3. To optimize on line detection, there's also a `find_lane_pixels_prior` function which detects lines in a binary warped image around an line that's already detected. This code is in the 12th code cell.
4. The following image visualizes how the line detection looks. The detected line pixels are colored in red and blue.
5. The code to draw the lane line pixels is in code cell 13 under `draw_lane_lines`.

| **test2_linepixels.jpg** | **test2_linepixels_fit_overplotted.jpg** |
|:--------------------:|:--------------------------------:|
|![alt text][test2_linepixels] | ![alt text][test2_linepixels_fit_overplotted]

#### 5. Radius of curvature and position of vehicle

1. The radius of curvature and the position of the vehicle is calculated in the 18th code cell.
2. The radius of curvature is calculated individually for each of the lane lines.
3. The average of the two radii is the radius of curvature at the center of the vehicle.
4. We get the radius in real world space in **meters** as the line pixels are converted from image space to real world.
5. The convertion rate is as follows

| **axis** | **real world distance**| **image space reference**|
|:--------:|:--------:|:--------------------------------------:|
| x-axis | 3.7 meters | width between the base points of each of the lane lines |
| y-axis | 30 meters  | approximate distance of the lane as seen in the bird's eye warped image of the lane |



#### 6. Lane detection in unwarped image

1. The lane detected in the warped image is then unwarped and projected onto the real image.
2. This is done in the `unwarp_lane_lines_to_frame` function in code cell 15.
3. The final result looks like this - 

| **test2_undistorted.jpg** | **test2_lane_detected.jpg** |
|:--------------------:|:--------------------------------:|
|![alt text][test2_undistorted] | ![alt text][test2_lane_detected]


---

### Pipeline (video)

Here's a [link to my video result](./output_videos/project_video.mp4)

---

### Discussion

1. This pipeline's line detection via gradients and color thresholding is noisy
2. In the challenge video, I see that this pipeline detects lines in the middle of the lane due to different colored asphalt on the road.
3. This can be avoided by expecting the base of the max bins while histogramming to be at a minimum distance.
4. The line detection also fails under very bright light conditions. The lane lines are noisy when the car drives in a patch of road which is under bright sunlight.
5. To handle those error, I would have to use smoothing of the line from previously calculated line pixels.
