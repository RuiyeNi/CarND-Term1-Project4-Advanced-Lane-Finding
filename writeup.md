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

[image1]: ./output_images/UndistortedChess.png "Undistorted Chess"
[image2]: ./output_images/UndistortedRoad.png "Undistored Road"
[image3]: ./output_images/BinaryImage.png "Binary Example"
[image4]: ./output_images/PersepctiveTransform.png "Transform Example"
[image5]: ./output_images/LanePixel.png "Lane Detection"
[image6]: ./output_images/LanePlot.png "Lane Plot"
[video1]: ./project_video_1.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

This step was completed by code cell 2 and cell 3 in the jupyter notebook [T1P4.ipynb](https://github.com/RuiyeNi/CarND-Term1-Project4-Advanced-Lane-Finding/blob/master/T1P4.ipynb). Generally speaking, object points  `objpoints` and image points `imgpoints` are required to calculate the camera matrix and distortion coefficients. Image points are all the inner corners of a chess board in a 2D world, and objects points are the targeted locations of these inner corners in a undistored 3D world with z=0 in this application case. Twenty calibration images were used to collect `imgpoints` list using `cv2.FindChessboardCorners` . Object points `imgpoints` were replications of the same targeted points for these images. There were 3 images whose inner corners were not sucessfully detected, and were excluded from further calcualtion. Camera matrix and distorion coefficients were obtained by applying `cv2.calibrateCamera` to `objpoints` and `imgpoints`,  and the results were saved as pickle file for later use. 

With camera matrix and distortion coefficients, an example of undistored image could be obtained with `cv2.undistort`:
![undistored chess][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

Similarly as how I undistorted the chess board image, I corrected one distored road image as follows:
![undistorted raod][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

Initially, I experimented with a combination of color transforms and gradients for thresholding binary image. I ended up with just using color transforms since it yielded more stable results. For the single image demo here, l channel and s channel from hls color space were used to obtain a binarized image. The code is in cell 6 and cell 7 in the jupyter notebook [T1P4.ipynb](https://github.com/RuiyeNi/CarND-Term1-Project4-Advanced-Lane-Finding/blob/master/T1P4.ipynb) 
![binary example][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code performing perspective transform is in cell 8 of [T1P4.ipynb](https://github.com/RuiyeNi/CarND-Term1-Project4-Advanced-Lane-Finding/blob/master/T1P4.ipynb). The goal is to transform the source points in a pre-transformed image to the destination points of a transfomred image. Source and destination points were hardcoded in the following manner:

```python
src = np.float32(
    [[(img_size[0] / 2) - 62, img_size[1] / 2 + 100],
    [((img_size[0] / 6) - 20), img_size[1]],
    [(img_size[0] * 4 / 6) + 272, img_size[1]],
    [(img_size[0] / 2 + 65), img_size[1] / 2 + 100]])
dst = np.float32(
    [[(img_size[0] / 4), 0],
    [(img_size[0] / 4), img_size[1]-5],
    [(img_size[0] * 3 / 4), img_size[1]-5],
    [(img_size[0] * 3 / 4), 0]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 578, 460      | 320, 0        | 
| 193, 720      | 320, 715      |
| 1125, 720     | 960, 715      |
| 705, 460      | 960, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![trasform example][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Lane-line pixels were founded by detecting the locations of pixel histogram peaks of binary image using code in cell 11 of [T1P4.ipynb](https://github.com/RuiyeNi/CarND-Term1-Project4-Advanced-Lane-Finding/blob/master/T1P4.ipynb). A polynomial was fitted for left and right lane respectively using `np.polyfit` based upon detected lane pixels. Identifed lane-line pixels are in red for left lane and blue for right lane. Fitted polynomial  is in yellow:
![lane detection][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

Radius of curvature of the lane was calculated by using fitted x and y pixels of lane lines with curvature equation. Left/right lane curvature with a smaller value was displayed as the lane curvature. The position of the vehicle with respet to the center was obtained by subtracting the middlepoint x pixel of the image from that of road space shaped by the fitted lanes. The code is in cell 10 of [T1P4.ipynb](https://github.com/RuiyeNi/CarND-Term1-Project4-Advanced-Lane-Finding/blob/master/T1P4.ipynb)

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in cell 13 of [T1P4.ipynb](https://github.com/RuiyeNi/CarND-Term1-Project4-Advanced-Lane-Finding/blob/master/T1P4.ipynb).  Here is an example of my result on a test image:

![lane plot][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

The code for producing the video is in [T1P4-Pipeline.ipynb](https://github.com/RuiyeNi/CarND-Term1-Project4-Advanced-Lane-Finding/blob/master/T1P4-Pipeline.ipynb). 

Here's a link to my video result:
[![video image](https://img.youtube.com/vi/etul2D4iIOU/0.jpg)](https://youtu.be/etul2D4iIOU)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

To obtain a reasonble stable lane detection in video, I slightly modified the binary image threhsolding process by using l channel from LUV color space and s channel from hsv color space. In addition, I applied different thresholds to these two channels depending on the road condition such as whether there are shadows or the general color of the road is darker or lighter. Fitted polynomials were averaged by using one previous correctly detected frame and one current frame. Frame with detected lanes beyond a certain range was discarded. The resulted video has smooth lanes for most frames, but there are still several frames with lanes plot extending a little beyond the authentic lane lines, especiallying in the senario where there was sharp change of the road lightness and color. To make it more robust, I could try to combine multiple other thresholding methods such as gradients and use more previous frames.
