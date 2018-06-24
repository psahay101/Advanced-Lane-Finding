
[//]: # (Image References)

[image1]: ./carUndistort.png "Undistorted"
[image2]: ./chessboard.png "Undistorted"
[image3]: ./laneBinary.png "Binary"
[image4]: ./result.png "Result"
[image5]: ./birdsEye.png "Top Down View"

# Advanced Lane Finding
The challenge in this project is to detect a lane on a driving track whose video is given. This is the 4th project in Term 1 of Udacity Self Driving Car nanodegree.
## Installation & Resources
1. Python from Ananconda
2. Data provided by udacity
3. Udacity Carnd-term1 starter kit with miniconda installation

## Files
find_lanes.ipynb : a jupyter notebook containing all the codes and the examples.

## Goals
1. Calculate camera calibration matrix and provide example of the process through a corrected image image. Use the technique on the test images. Demonstrate through examples.
2. Transform the image by finding and then applying the gradients. Demonstrate through examples.
3. Transform the image by finding and then applying the perspective transform. Demonstrate through examples.
4. Show how to obtain lane-line pixels and then fit their  respective coordinates in an polynomial. Demonstrate through examples.
5. Show the method to calculate the radius of curvature of the lane and the position of the vehicle with respect to center of the lane.
6. Apply all of the above techniques on an image and show it as an example.
7. Apply all of the above techniques on a video as the result.
8. End discussion

## 1. Calibration
Photo taken by camera tends to have some type of distortion in it. There are two type of distortions: radial distortion (straight line appears curve) and tangential distortion (some area of an image looks nearer than expect). (Source: OpenCV). Distortion produces incorrect presentation of real scene for implementation such as Lane Dectection. To fix this, undistortion procedure can be applied by calibration

The image taken from a camera is usually distorted with either radial or tangential distortions. To correct this the first step is to calibrate the image. For this we would be needing some parameters which are known as the distortion coefficients and camera matrix.

The process to obtain them are as follows:
1. Create a list of object points with shape (nx*ny,3) which in turn represent (x,y,z) coordinates. Assume z=0 0 as the chessboard is stationary.
2. Apply cv2.findChessboardCorners(gray,(nx,ny),None) function on a number of chessboard images. This will help in obtaining a set of defined corner coordinates.
3. Then these coorner coordinates will be termed as image points and the list of coordinates in the step one will be called as the object points.
4. Apply the following fuction to find the camera matrix mtx and distortion coeffcients dist from objpoints and imgpoints information:

  cv2.calibrateCamera(objpoints,imgpoints,grayimage.shape[::-1],None,None)
5. Apply cv2.undistort(img,mtx,dist,None,mtx) as the final step to undistort the given image.



Below is an example of the undistorted image after a distorted image was being corrected through the above steps.
The left image is the original image(distorted one).
![][image2]


One more example is shown using the same technique on the image of a road rather than the chessboard. Here the differences are a bit more subtle to notice but with some attention minute differences can be noticed.
![][image1]

## 2. Binary Threshold
The next step is to convert the color image into a binary image which is more useful for lane finding. This is done as follows:
1. Combine gradient threshold on X direction OR S channel threshold (from HLS) AND V channel threshold (from HSV).
2. Apply a mask to the binary image to only keep the area required for us thus removing the noise.


Below is an example of all the three images in the following order: Original Image, Binary Image and Masked Image.
![][image3]

## 3. Transformation
After the previous step the image is transformed into a top down view or the bird's eye view as this view is more suitable for our task of finding the lane lines and their further parameters like curvature etc. The steps to do that are as follows:
 Apply cv2.getPerspectiveTransform(src,dst) to find a perspective transform matrix from source coordinate src and transform to destination dst.

 Below is an image showing the top down view transformation of an original image.
![][image5]

The Udacity's help codes are being referred for the src and dst.

## 4. Lane Lines Polynomial
The method to find lane lines polynomial is:

1. Find the highest peak locations of white pixels in binary image using the histogram plot.
2. Find all the pixels representing the left and the right lane lines and store them in a matrix.
3. Fit a curve into both the left and the right lane line coordinates using numpy.polyfit() and using the polynomial degree as 2.

## 5. Radius of Curvature and Off-center distance
### Radius of curvature (ROC)
Formula of radius of curvature is : ROC= [(1+(2*A*y)^2)^(3/2)]/|2*A| where f(y)=A*y^2 + B*y + C.

Here np.polyfit() is used to obtain the velues of A, B and C.
Also y = value of bottom most coordinate in the image.

Finally the radius of curvature will be the average of ROC of both the left and the right lanes.

### Off- Center distance (OCD)

To find the OCD we have to find the distance between the image center and the lane center as the car is located at the image center (as the camera is mounted at the center of the car).

But this distance will be in the form of the pixel distance which needs to be scaled into real dimensions which can be done by using the scaling factor given Below.
y_direction = 30/720, x_direction = 3.7/700.

These  scaling factors are inferred from the fact that the width of the lane is 3.7 meter and the length of the lane in the image is 30 meters, specified by Udacity.

## Results (Image)
![][image4]

## Results (Video)

Found on github and youtube : https://youtu.be/WsaQwr4TTDk

## Future Works
1. Code can be made faster by skipping the finding of all the left and right lane pixels for all the images by just doing it for the first time and then checking whether the polynomial cover some x percentage of white pixels on it or not and if this case isn't met then a function can be called to find the new lane line pixels as an update is needed.
2. The first lane line can be found from the center of the image by going left and right certain pixels as to avoid multiple lanes and only finding the lanes in which our car is currently positioned.
