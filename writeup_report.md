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

[image0]: ./output_images/uncalibrate_0.jpg "Image - hold chessboard with corner marked"
[image8]: ./output_images/calibrate_undistort_1.jpg "Image - hold chessboard undistorted 1"
[image9]: ./output_images/calibrate_undistort_2.jpg "Image - hold chessboard undistorted 2"
[image10]: ./output_images/calibrate_undistort_3.jpg "Image - hold chessboard undistorted 3"
[image1]: ./output_images/calibrate_1.jpg "Image - hold chessboard with corner marked"
[image2]: ./output_images/new_original_1.jpg "Image - Raw image and calibrated images"
[image3]: ./output_images/new_undistorted_1.jpg "Image - Raw image and calibrated images"
[image4]: ./output_images/new_warped_1.jpg "Image - unwarped and sobel threshold applied"
[image5]: ./output_images/new_sobel_1.jpg "Image - unwarped and sobel threshold applied"
[image6]: ./output_images/new_histogram_1.jpg "Image - Lines drawn based on histogram for lane marking"
[image7]: ./output_images/new_lane_mark_1.jpg "Image - final image with lane marking"
[video1]: ./white.mp4 "Video - Lines drawn on the images"

---
### Files Submitted & Code Quality

#### 1. Submission includes all required files and can be used to run the simulator in autonomous mode

Project includes the following files:
* **lanefindpipe.html** containing the script to advanced lane finding
* **white.mp4** containg snapshot of results
* **writeup_report.md**  summarizing the results
* **calibrate.py** camera calibration
* **calibrate.p** pickle file with calibration parameters

### Camera Calibration

I started processing all test images provided for calibration. Most of the images provided has corner(x,y) = (9,6). however, there are few images were different.
Therefore, not all images were getting corners identified. As there were only 20 images that needed for calibration, i manually counted
corners in each image and found that images(calibration1.jpg, calibration4.jpg and calibration5.jpg) got different corners. I have created array of length 20 to hold
corners(x,y) and specified values that match the images.  This can be found in calibrate.py code (between line#47 to line#50).

After calibration. data is store dinto file **calibrate.p**.

Code file name: **calibrate.py**

I have also created test code to verify the output of calibration(code file: calibrate.py, between line#74 and line#108). This test code also prints test image out to file 
calibrate_1.jpg.

![Image - hold chessboard with corner marked][image1]

![Image - undistored image based on calibration1][image8]

![Image - undistored image based on calibration2][image9]

![Image - undistored image based on calibration3][image10]

### Lane finding Pipeline

Most of the tasks required in lane finding pipeline was split into multiple function and details provided below.
Code file: **lanefindpipe.html**

#### Distortion corrected image

Undistortion of image was achieved using two function mentioned below.

**getCalibrationParam** : Function read calibrate.p pickle file and use these parameters to undistort images from the camera.  
**undistort** : function to undistort image based on above camera calibration parameters.

Below is display of raw image and undistorted image.

![Image - Raw image][image2]


![Image - undistorted calibrated images][image3]


#### Perspective Transformation.

Perspective transformation achieved in below function. From camera, it seems like lane is narrowing down from closest point of camera to far point. To draw identify lanes it is necessary to have 
lanes to be parallel. Therefore, perspective transformation is applied where straight line will remain straight even after transformation.As lanes are narrowing down from closest camera point to far point, lane need be streched more in far point. 
This will make it look like parallel point. To achieve this four points were identified on image and perspective transformation was applied.

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 560,450       | 100,0        | 
| 690,450       | 1000,0        |
| 250,680       | 300,720       |
| 1050,680       | 1000, 720      |


**corner_unwarp** : Function primarily applies cv2 prospect transformation that will allow to capture lanes very clearly.

![Image - Image - unwarped][image4]

#### Color Thresholding

Sobel gradient thresholding was applied based on threshold parameter provided below. Most of the test images found were having lanes as while, Yellow and Blue,
Below gradient threshold between 50 and 255 should be able to identify these colored lanes. Also, magnitude threshold was defined as 10-255, so that minor magnitude
changes below 10 will be ignore and this will avoid identifying shades on road as color changes.

```python
    ksize = 3
    # Apply each of the thresholding functions
    gradx = abs_sobel_thresh(s_hls_channel, orient='x', sobel_kernel=ksize, thresh=(50, 255))
    grady = abs_sobel_thresh(s_hls_channel, orient='y', sobel_kernel=ksize, thresh=(120, 255))
    mag_binary = mag_thresh(s_hls_channel, sobel_kernel=ksize, mag_thresh=(10, 255))
    dir_binary = dir_threshold(s_hls_channel, sobel_kernel=ksize, thresh=(0,np.pi/2))
```

Each function mentioned above does specific task required for sobel thresholding

**abs_sobel_thresh** - calculates X ro Y direction gradient.
**mag_threshold** - calculates sobel magnitude changes
**dir_binary** - calculates directional changes


After sobel thresholding, color thresholding was applied for YELLOW and WHITE color this thresholding helped in identifying lane beter.
Also, L channel in HLS scheme was put in with lower threshold to handle shadows on the road. Area of interest has been identified that is most probable pixels that will capture lanes and all other pixels were masked

```python
   image_hls = cv2.cvtColor(warped, cv2.COLOR_RGB2HLS)
    h_hls_channel = image_hls[:,:,0]
    l_hls_channel = image_hls[:,:,1]
    s_hls_channel = image_hls[:,:,2]
    
     #Threshold on yellow and white color mask
    lower = np.uint8([  0, 200,   0])
    upper = np.uint8([255, 255, 255])
    white_mask = cv2.inRange(image_hls, lower, upper)
    # yellow color mask
    lower = np.uint8([ 10,   0, 100])
    upper = np.uint8([ 40, 255, 255])
    yellow_mask = cv2.inRange(image_hls, lower, upper)
    # combine the mask
    mask_wy = cv2.bitwise_or(white_mask, yellow_mask)

    #filter on light channel to handle shadows on road.    
    l_hls_channel[l_hls_channel > 100] = 1
    
    combined = np.zeros_like(dir_binary)
    combined[((gradx == 1) & (grady == 1)) | ((mag_binary == 1) & (dir_binary == 1)) | mask_wy == 255 | (l_hls_channel == 1)  ] = 1

    # Masking Region of interest
    region_of_interest = np.array([[[150,0],[1200,0],[1200,720],[150,720]]], np.int32)
    
    mask = np.zeros_like(combined)
    if len(img.shape) > 2:
        channel_count = img.shape[2]  # i.e. 3 or 4 depending on your image
        ignore_mask_color = (255,) * channel_count
    else:
        ignore_mask_color = 255

    cv2.fillPoly(mask, region_of_interest, ignore_mask_color)
    
    masked_image = cv2.bitwise_and(combined, mask)
```

![Image - sobel thresholding and masking][image5]


#### Lane pixel identifation.

Once above preprocessing is complete, output image will be used for marking lanes. Approach was to use histogram to identify starting point in bottom half of of the mage. frequency buckets holding up most intensive pixels will be
identified and will be marked as starting point of lane. Then, rectangular boxes will be built vertically up from starting point. Lanes will be split into 9 rectangular boxes. Each box will re-estimate the center point and calculate 
rectangular box for marking.

Now pixels within the rectangular boxes that were NONZERO will processed through polyfit to identify quadratic equation parameters to draw a line. After identifying quadratic parameters, X,Y location for all Y values will be calculated. This was done
for both left and right lane.

Below functions are used for lane pixel identification.

**find_histogram_lane** : function used to find histogram that will capture high frequency location based on lane markings. These will be used as base point to start building
rectangular boxes that will be stacked up from bottom to top of the image in y-axis direction. Later, based on pixels within the rectangular boxes will be used to draw lanes using polyfit

![Lines drawn based on histogram and final image with lane marking][image6]

One X,Y value for both left and right lane were identified, below function is called that will Unwarp the image point and map it back to original images. This step will identify four lane points(2 left lane points and 2 right lane points) that can be
used to draw polynomial that will make lane marking. this step is achieved using below function.

**draw_org_img** : function used for drawing final markings on the lane and writing Radius of curvature and car position on the image.

![Final image with lane marking][image7]

#### Curvature calculation

Lane curvature calculated in meters based on below code. Below code need 
1. As lane curvature will be calculated in world space instead of pixels points, X,Y values identified in lane marking will be multiplied with UNIT conversion constants as shown below.
	- a. ym_per_pix = 30/720 # meters per pixel in y dimension
	- b. xm_per_pix = 3.7/650 # meters per pixel in x dimension -  **width between the lanes after prespetive transformation is around 650 pixels(average)
2. After unit conversion, points will be used to calculate quadratic points
3. Also, lane curvature is alway from point of reference. Therefore, max point in Y-axis(closest to camera) will be consider as point of reference.
4. Now, using point of reference and X,Y(point after unit conversion) will be used for curvature calculation based below formula displayed in python syntax.

```python
        # Define conversions in x and y from pixels space to meters
        ym_per_pix = 30/720 # meters per pixel in y dimension
        xm_per_pix = 3.7/650 # meters per pixel in x dimension
        y_eval = np.max(fity) #Point of reference
        
        # Fit new polynomials to x,y in world space
        fit_cr = np.polyfit(fity*ym_per_pix, fitx*xm_per_pix, 2)
        curved = ((1 + (2*fit_cr[0]*y_eval + fit_cr[1])**2)**1.5) / np.absolute(2*fit_cr[0])
```

#### Putting all together

**img_process** : Function that stritches all above functions together.

Above function "image_process" stritches all together.  Before calling this function camera calibration parameters stored in pickle file will be retreived and then following steps are run to complete pipeline

1. One time function(**getCalibrationParam**) call that will fetch camera calibration parameters
2. image read and undisorted using function(**undistort**)
2. undistorted image's corners unwarped using perspective transformation using function(**corner_unwarp**)
3. sobel thresholding applied on unwarped image using different functions mentioned in **color thresholding** section.
4. Image sent to histogram based lane identification process using function(**find_histogram_lane**)
5. Finally image process and lane markings layed over actual image using function(**draw_org_img**)

As per the requirement for lane smoothing, it was mentoined to capture the lane specific metrics that can be averaged accross the image frames in video. It was also suggested to
capture these metrics in Class Line(). All pre-defined metrics in Line are captured and can be displayed as shown below.


**Line Class** : This class was built to keep track certain data parameters as required by project. Below is Line() information captured

```python

print(left_lane)
self.lane_side:            left
self.detected:             True
self.recent_xfitted.shape: (1, 720)
self.allx.shape:           (720,)
self.ally.shape:           (720,)
self.bestx.shape:          (720,)
self.best_fit:             [  2.32524504e-02  -1.01567598e+01   1.16026153e+03]
self.current_fit:          [  2.32524504e-02  -1.01567598e+01   1.16026153e+03]
self.prev_fit:             [0.0, 0.0, 0.0]
self.radius_of_curvature:  3318.47187678 
self.line_base_pos:        3.29310820949 m
self.diffs:                [ 0.  0.  0.] m

print(right_lane)
self.lane_side:            right
self.detected:             True
self.recent_xfitted.shape: (1, 720)
self.allx.shape:           (720,)
self.ally.shape:           (720,)
self.bestx.shape:          (720,)
self.best_fit:             [  9.78403128e-03  -1.57865909e+01   6.38731380e+03]
self.current_fit:          [  9.78403128e-03  -1.57865909e+01   6.38731380e+03]
self.prev_fit:             [0.0, 0.0, 0.0]
self.radius_of_curvature:  2651.42896245
self.line_base_pos:        6.96245667006 m
self.diffs:                [ 0.  0.  0.] m
```

If in current frame lines are almost near to parallel, then, adding the current identified lane details to Line() class for averaging it. otherwise, picking up previous
average value like Line.bestx for drawing lane on the frame.


#### Applying pipeline on Video

[Apply Pipeline on Video](./white.mp4)

In final automation run, lane finding pipeline applied on video - project_video.mp4.

---

NOTE: Each of test images display to present each step taken in pipe are from first test image. There are 7 more images that were processed with pipeline and checked into /output_image folder.

###Discussion

Following are challenges that faced:
1. I had challenges in implementing the Line() class. Requirements were not very clear and it took a while before get the full understanding.
2. Averaging Lane.bestx. because, bestx was the value passed onto final lane marking to get smoothened lane fit values. Each test image was from
different lane altogether, it was not averaging well. 
3. Also had some challenges in sobel threshold value adjustment. Need to study more on Computer vision to provide better threshold values.

Where code might fail:
1. Even though, i was averaging out lane fit value, i think, it still need to be tested with-in city limit(ie away from highway). Within
city limits, there were sudden curves and complete 90 degree left/right turns. Now sure current technique will fully work. 
2. As mentioned in above point, i tried pipeline on very hard challenge video and pipeline did fail. Mainly, it seems like road are curvy, there,
Line() object capturing left_fit and right_fit will be significantly different between frames. therefore, averaging might not lead to good lane marking.





