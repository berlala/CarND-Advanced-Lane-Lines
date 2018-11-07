
# Advanced Lane Finding Project
Bolin ZHAO (bolinzhao@yahoo.com)

<script type="text/javascript"
   src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML">
</script>


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

[image1]: ./output_images/undist_1.png "Undistorted"
[image11]: ./camera_cal/calibration1.jpg "Undistorted"
[image2]: ./output_images/undist_2.png "Undistorted road"
[image21]: ./test_images/test5.jpg "Undistorted road"
[image3]: ./output_images/binary.png "Binary Example"
[image4]: ./output_images/persp.png
[image5]: ./output_images/test_boundline.png 
[image6]: ./output_images/example_2.png "Output"
[video1]: ./output_images/project_video_output.mp4 "Video"

---

### Camera Calibration and Distortion

#### 1.  Identify Object 
First look into the chessboard pics, identify how much cross points it shows. In this case, the total objects are 9*6.
#### 2. Find Corners
After convert the pics into gray, log all the 2D points in the plant and 3D points on the original pictures by  ```findChessboardCorners``` function as follow.   

```
 ret, corners = cv2.findChessboardCorners(gray, (nx,ny),None)
```   

Right after all the chessboard pics are processed, any picture from the camera can be distorted by the following code,   
```
test_undist = cal_undistort(test_img, objpoints, imgpoints)
```
The output is the picture with distortion. An example can be found below,

![original][image11]
![After remove distortion][image1]    

### Pipeline (single images)

#### Step 1. Remove the distortion  by defined points from camera calibration session.

![original][image21]    
![After remove distortion][image2]    

#### Step 2. Convert the processed picture into binary picture.
In this step, three criteria are applied. They are saturation and light from HSL chance, and the gradation from X direction.  Saturation is mainly working for the left yellow  lane. Light can be used to weak the influence during the  shadow on the route. Gradation in X direction can pick up both lanes.


![Binary][image3]

#### Step 3. Perspective Transform 
The important step is to transform the picture into perspective view. Firstly, the interested area is defined as ```[250,705],[1155,705],[785,505],[530,505]```. This is help to remove the noise from the rest of route. The target area is defined as  ```[250,705],[1155,705],[1155,505],[250,505]```. It is a rectangle area.
The function is called ``` perspective_tf(img)```.   

![ Perspective Transform ][image4]

#### Step 4. Identify the pixels
The iteration windows method is used here to identify all the pixels. The pixels which are binary are picked out by the defined rectangle window in each iteration from bottom (big Y value) to the top.   

The x position of the window is re-center during every iteration by the average x value  of found pixels.   

![Pixels of lanes][image5]

#### 5.  Radius of curvature of the lane and the Position of the vehicle 
The lanes are given as the splashes as ```[leftx, ploty] ```and ```[rightx,ploty] ```from the function ```fit_polynomial(test_persp)```.     
Firstly, the left and right lanes which represent by pixels as '''[leftx, lefty]''' and '''[rightx, righty]''' can be processed by ```polyfit``` to get a second order equations.    
Secondly, based on the coefficients of the second-order equation, all the y values can be calculated by the defined```ploty```  which is a  uniform distribution on Y axle.    
At last, the curvature value can also calculated by the definition of the curvature radius, since the derivation of the equation can easily got as the coefficients of fitted second-order equation. The detail is in function ``` measure_curvature_real() ``` .    
    
The vehicle center $X_{car}$ can be assumed as the same as the picture center line(which means the camera is installed at the middle behind the windshield). The route center can be calculate as follow,   

$$
X_{center} = X_{left} + \frac{X_{right}- X_{left}}{2}
$$

The different between $X_{center}$ and  $X_{car}$ is the error.


#### 6. Final plot

![Final output][image6]

---

### Pipeline (video)


Here's the processed video [video1]

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The following are  the main problems and thinkings I have faced during this project,
1) sometime the ```polyfit``` can not find lane due to there is no pixels in the interested area. In the real implementation, a keep or delay function should be applied to hold the past lane for a while.    
2) The shadows on the route show a strong affect on the pixel identification. More ciritera may be introduced to help remove the effect.    
3) The curvature value should be calculated in the front of vehicle but not the current position if it is used to control the steering wheel due to the time delay. 
