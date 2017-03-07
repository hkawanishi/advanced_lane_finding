##**Advanced Lane Finding Project**

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

[image1]: ./camera_cal/calibration1.jpg "Camera Imaged to be Calibrated"
[image2]: ./output_images/calibration_camera_images/calibration_with_corners_1.jpg' "Chessboard with Corners"
[image3]: ./output_images/binary_images/binary_image_1.jpg "Binary Example"
[image4]: ./output_images/warped_images/warped_image_1.jpg "Warp Example"
[image5]: ./output_images/images_with_sliding_windows/windows_image_0.jpg "Fit Visual"
[image6]: ./output_images/line_images/road_image_0.jpg "Road Image with Lines"
[image7]: ./output_images/results_with_curv_info/info_image_1.jpg "Curvature and Position Info"
[video1]: ./project_output.mp4 "Output Video"

###Python file: project_hkawanishi_4.ipynb


###Camera Calibration

The code for this step is contained in the first, second, and third cells.  

I started by preparing object points for the chessboard size 9x6. I used glob statement to go through all calibrate*jpg files in camera_cal directory.  The example of calibrate*.jpg is as follows:  
![Camera Imaged to be Calibrated][image1]
For each image, I used findChessBoardCorners to find corners and if the corners are found, stored in imgpoints array.  I rendered the detected chessboard corners using cv2.drawChessboardCorners. The example of the resulting image is:
![Chessboard with Corners][image2]

The camera undistortion was done using cv2.calibrateCamera function which outputs destination coeffcient and camera matrix which was used to transform 3d object points to 2d image points.  Those coefficients and a distorted image became inputs cv2.undistort function, and it returns an undistorted image.  In cell 4, I did a quick test using 'camera_cal/calibration1.jpg' file.  These functions are also used in cell 9 when I used still images to do some investigating.  And also used in cell 11 for a pipeline function which will be discussed later.

###Use color transforms, gradients, etc., to create a thresholded binary image

Color transforms are done in cell 6.  This is one of the places I spent a lot of time since there are lots of "knobs" (parameters) to try out.  In the end, I decided to use sobel x and y operators (abs_sobel_thresh functions in cell 6) and S-Channel in HLS color space (color_thresh function). I tried the magnitude and the direction of the gradient and applied the threshhold but they did not seem to improve detecting lines.  One of the images with color/gradient transformation is:
![Binary Example][image3]

###Apply a perspective transform to rectify binary image

The perspective transformation is done in cell 7. `src` is the source points and `destination` is the destination points for the transformation.  The four corners points (x, y) I used had an order (1) top left, (2) bottom left, (2) bottom right, and (4) top right.  I used the parameters for the source points, and I spend a lot of time tweaking these points.  Unfortunatly, I didn't find the "best" settings.  I did all these settings manually.  Once these source/destination points are determined, I used cv2.getPerspectiveTransform function to do perspective transform.  I used the same function but flipped source and destination points to get inverse image which will be used later.  And then, I used cv2.warpPerspective to get the warped image.  The example of the warped image is as follows:
![Warp Example][image4]

###Detect lane pixels and fit to find the lane boundary
To detect lane pixels, I used the sliding window search method which is done in cell 8.  
I first took a histogram along all the columns of the bottom half of the image.  This helps to decide where the lines are.  Then from the results of the histogram, determined the starting points at the bottom and using the sliding windows to obtain the non-zero pixel locations.  After some investigations, I decided to use 18 windows veritically with width of 30.  Here, I also added some validations in terms of the lanes: (1) if there is no right lane information, use left lane data, and (2) make sure the lanes are kept within a certain horizontal distances.  
The results are visualized and the example is:
![Fit Visual][image5]

###Warp the detected lane boundaries back onto the original image
Cell 8 also contains the cv2.warpPerspective which uses Minv obtained in cell 7 and changes back to the original image.  Using hte cv2.fillPoly functions, the lines for the lanes and the between those lanes are drawn.  The example image is:
!["Road Image with Lines"][image6]

###Determine the curvature of the lane and vehicle position with respect to center 

This is also done in cell 8.  I used variable called xm_per_pix and ym_per_pix which are a meter per pixel in x- and y- direction, respectively.  Those numbers are determine by looking at the length of the car and the lane width.  Using these variables, the polynomial for the lane equations are refitted.  And then using the curvature equations, left and right curvatures were obtained.  The vehicle position (horizontal direction) was obtained simply by using the positions of the left and right lanes at the bottom of the image and calculate the differences between the center of those points and the center of the image.  

###Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position
I used cv2.putText to add these information (cuvatures and position) to the image.  The example image is:
![Curvature and Position Info][image7]

Note: cell 9 is used to go through all the examples images, and for each image, undistort, color binary transformation, perspective transformation, and lane are found using sliding windows.  

###Pipeline (video)
Once I was able to detect lines using still images (cell 9), I applied all these functions to a video image (project_video.mp4). Cells 11-14 are used to analyze a video.  Cell 10 import necessary functions, cell 11 is the pipeline, cell 12 contains the process_image function, and cell 13 specifies input/output video files.  My video output is:
![Output Video][video1]


###Discussion

####The problems / issues I faced.  Where will my pipeline likely fail?  What could I do to make it more robust? Etc., Etc.

My pipeline doesn't seem to work 

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  

(1) My pipeline doesn't work well in challenge video.  The left and right lanes are "dancing" around in the video.  I think I needed to spend more time tweaking the corner paremeters when doing the perspective transformation.  I needed to invetigate more robost way to determine these four points. I wish I had more time.
(2) I feel like there is a better way to do a color/gradient transformation.  I feel that I had a reasonable output using project_video.mp4, but I didn't do well in the challange video especially when there are some other cars in vicinity.  I need to work on filtering out those objects and implement a better method to detect the line.  

