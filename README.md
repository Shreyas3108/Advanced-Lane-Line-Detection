# Advanced-Lane-Line-Detection
In this project , We try to create a improved lane finding algorithm using Computer vision. 

**Goals / Step **

The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.


Here I will consider the rubric points individually and describe how I addressed each point in my implementation. 

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it! 


### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The first step we follow is to calibrate our camera , Why ? Since the lenses on the camera distort the image. The real analogy to it would be how in a photo the object which are farther away appear smaller than it's actual size. In terms of road , The two lanes appear to converge. To remove this we must calibrate our camera and undistort the image. 

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![Before and After , Chessboard](https://github.com/Shreyas3108/Advanced-Lane-Line-Detection/blob/master/Screenshot%20(33).png?raw=true)

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![Before and After , Test image](https://github.com/Shreyas3108/Advanced-Lane-Line-Detection/blob/master/Screenshot%20(34).png?raw=true)

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps at cell 19 ).  Here's an example of my output for this step. 

The thresholding involves white and yellow masking and then we combine it to get out binary output 
![Input](https://github.com/Shreyas3108/Advanced-Lane-Line-Detection/blob/master/test_images/test3.jpg?raw=true)
![Output Sobel](https://github.com/Shreyas3108/Advanced-Lane-Line-Detection/blob/master/Screenshot%20(36).png?raw=true)

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warp()`, which appears in cell 15 .  The `warp()` function takes as inputs an image (`img`),source (`src`) and destination (`dst`) points are inside the warp function .  I chose the hardcode the source and destination points in the following manner:

```python
src = np.float32(
    [[(img_size[0] / 2) - 55, img_size[1] / 2 + 100],
    [((img_size[0] / 6) - 10), img_size[1]],
    [(img_size[0] * 5 / 6) + 60, img_size[1]],
    [(img_size[0] / 2 + 55), img_size[1] / 2 + 100]])
dst = np.float32(
    [[(img_size[0] / 4), 0],
    [(img_size[0] / 4), img_size[1]],
    [(img_size[0] * 3 / 4), img_size[1]],
    [(img_size[0] * 3 / 4), 0]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 580, 460      | 260, 0        | 
| 700, 460      | 1040,0      |
| 1040, 680     | 1040, 720      |
| 260, 680      | 260, 720        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image. Here are the warped as well as unwarped image returned from the function. 

![Warped](https://github.com/Shreyas3108/Advanced-Lane-Line-Detection/blob/master/Screenshot%20(37).png?raw=true)
![Unwarped](https://github.com/Shreyas3108/Advanced-Lane-Line-Detection/blob/master/Screenshot%20(38).png?raw=true)

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I used 2nd order polynomial to fit my lane lines like this :

![Fitting the Lines](https://github.com/Shreyas3108/Advanced-Lane-Line-Detection/blob/master/Screenshot%20(39).png?raw=true)

The idea was to use the histogram to find the pixels on the image and use the values generated through the histogram to compute the lane detection using the sliding window on the function polyfit_sliding , The sliding function takes the mid point of the histogram and computes the max value through the left and right side to find the pixels. 

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in cell #22 through function measure_curvature. Whose left and right fit values are calculated by the function polyfit_sliding in line number 21. 

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I created a function called pipeline which calls all the function and implements the steps prescribed. The function returns the image output as this 

![Result](https://github.com/Shreyas3108/Advanced-Lane-Line-Detection/blob/master/Screenshot%20(35).png?raw=true)

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](https://github.com/Shreyas3108/Advanced-Lane-Line-Detection/blob/master/project_video_done_2.mp4)
--- 

### Discussions 

The lane identification performs well in case of the first video but fails in case of the challenge video which could be due to the functions of fitting the line. The lane also predominantly works / made to work with yellow and white lanes but if there are any other colour lane (Not that i know of) It would fail. It would fail in india for sure , because there is no lane so as well that's a seperate discussion. 
