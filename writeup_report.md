# **Advanced Lane Finding** 

---

## **Advanced Lane Finding Project Goals/Summary**

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

[image1]: ./output_images/calibrate_1.jpg "Image - hold chessboard with corner marked"
[image2]: ./output_images/raw_undistort_1.jpg "Image - Raw image and calibrated images"
[image3]: ./output_images/unwarp_sobel_1.jpg "Image - unwarped and sobel threshold applied"
[image4]: ./output_images/final_1.jpg "Image - Lines drawn based on histogram and final image with lane marking"

---
### Files Submitted & Code Quality

#### 1. Submission includes all required files and can be used to run the simulator in autonomous mode

Project includes the following files:
* **lanefindpipe.html** containing the script to advanced lane finding
* **white.mp4** containg snapshot of results
* **writeup_report.md**  summarizing the results
* **calibrate.html** camera calibration
* **calibrate.p** pickle file with calibration parameters

#### 2. Submission includes functional code

Lane finding code submitted as html file for reference. Code is split into two sections.

calibrate.html - Code that calibrates the camera based on the test images provided. This one time run and all calibrated parameters are stored in pickle file.
lanefindpipe.html - Code that find the lane and marks on the images. 

#### 3. Submission code is usable and readable

Submitted code meets all goals defined and steps explained below. All functions requirements defined as functions mentione dbelow

These functions are called in pre-processing step before training the model.

**Camera Calebration**

**calibrate_camera** : Funcation find corners in all 20 images provided. Most of the images provided has corner(x,y) = (9,6). however, there are few images were different.
Therefore to get corners on all images, array of corner(x,y)s was defined and got corners marked in all images.

After identifying corners, calibration parameters were pickled to file calibrate.p

![Image - hold chessboard with corner marked][image1]

**Lane Finding Pipeline**

**getCalibrationParam** : Function read calibrate.p pickle file and use these parameters to undistort images from the camera. 

**undistort** : function to undistort image based on above camera calibration parameters.

![Image - Raw image and calibrated images][image2]


**corner_unwarp** : Function primarily applies cv2 prospect transformation that will allow to capture lanes very clearly.

**sobel thresholding functions** : There were multiple functions defined to achieve sobel thresholding like sobel gradient threshold, sobel magniture threshold and sobel direction threshold.
These threshold were used jointly to apply thresholding on the images

![Image - Image - unwarped and sobel threshold applied][image3]

**find_histogram_lane** : function used to find histogram that will capture high frequency location based on lane markings. These will be used as base point to start building
rectangular boxes that will be stacked up from bottom to top of the image in y-axis direction. Later, based on pixels within the rectangular boxes will be used to draw lanes using polyfit

**draw_org_img** : function used for drawing final markings on the lane and writing Radius of curvature and car position on the image.

![Lines drawn based on histogram and final image with lane marking][image4]

**calc_curvature** : Used for finding for finding line curvature.

**img_process** : Function that stritches all above functions together.

**Line Class** : This class was built to keep track certain data parameters as required by project.



---

### Techniques Strategies


#### 1. Sobel Threshold identification

Multiple options tried for thresholding and after experimenting gradientx threshold was set to (120,255) and gradienty was also set to (120,255).
graident magnitude threshold set to (25,255) that will capture minimal magnitude change starting from 25. After sobel thresholding to segregate
area of interest, masking technique used to all unrelated parts of image other than LANE markings.

#### 2. Lane finding:

After image is pre-processed based on prospect translation, sobel threshold filtering and masking, image processed thru function - find_histogram_lane. This function identify
heighest frequency in pixel in bottom part of the image and this pixel location becomes the starting part of building lane markings. Based on this starting points, recatangular boxes
built vertically up based on pre-defined measurements. Later based on pixel intensity, pixels were processed with polyfit function.

#### 3. unwarp - prospective tranlsation:

Considering center of image as width/2 and height/2, points were identified on image that were used for prospective translation.

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 300,500       |  75,0         | 
| 900,500       | 1075,0        |
| 300,680       | 300,680       |
| 900,680       | 775, 720      |

#### 4. Stritching all together

Function process_img striches all together by calling pipeline in sequence

1. Before calling pipelines of processes, camera calibration parameters read from pickle file.
2. image read and undisorted.
3. undistorted image's corners unwarped using prospective translation.
4. sobel thresholding applied on unwarped image.
5. Image sent to histogram based lane identification process.
6. Finally image process and lane markings layed over actual image.


In final automation run, lane finding pipeline applied on video - project_video.mp4.