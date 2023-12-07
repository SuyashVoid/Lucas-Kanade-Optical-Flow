# Lucas Kanade Tracker

## Introduction

Optical flow is the pattern of apparent motion of image objects between two consecutive frames caused by the movemement of object or camera. It is 2D vector field where each vector is a displacement vector showing the movement of points from first frame to second. Consider the image below (Image Courtesy: OpenCV documentation):
![Optical Flow](https://docs.opencv.org/3.4.0/optical_flow_basic1.jpg)

## Part 1: Two-frame Optical Flow

![Two Frame Optical Flow](/Outputs/Part1All.png)

- This is the output of the program for all the images in the dataset. To get to this point, we first calculate the x, and y partial spatial derivatives of the first frame(using five-tap filter).
- Then we calculate the partial temporal derivative of the second frame (blurred).
- Then we precalculate $Ix^2, Iy^2, IxIy, IxIt, IyIt,$ before convolving it with a simple WxW filter made of all ones (to get the sum of the values in the WxW window). Kinda like box filter but without the division by $W^2$.
- Then we loop through each pixel and find out the $A^TA$ matrix and $A^Tb$ vector for each pixel that looks like: 
$A^TA = \begin{bmatrix} \sum Ix^2 & \sum IxIy \\ \sum IxIy & \sum Iy^2 \end{bmatrix}$
$A^Tb = \begin{bmatrix} -\sum IxIt \\ -\sum IyIt \end{bmatrix}$
- Then we solve the equation $A^TAx = A^Tb$ to get the x and y components of the optical flow vector for each pixel.

## Part 2: Warp and Subtract

<!-- Make table for 3 images -->

| Difference                                         | Expected                                         | Warped                                         |
| -------------------------------------------------- | ------------------------------------------------ | ---------------------------------------------- |
| ![Warp and Subtract](/Outputs/Part2Difference.png) | ![Warp and Subtract](/Outputs/Part2Expected.png) | ![Warp and Subtract](/Outputs/Part2Warped.png) |

- Here, we just warp the first image using the optical flow vectors calculated in part 1 and subtract it from the second image to get the difference image.
- Warping is done using scipy.interpolate.rectbivariatespline function.

## Part 3: Lucas-Kanade Tracker

![Lucas Kanade Tracker](/Outputs/Part3.gif)

- Here, we start by finding Harris corners in the first frame using the response function: $$R = det(M) - k(trace(M))^2$$ where $M$ is the structure tensor: $$M = \begin{bmatrix} \sum Ix^2 & \sum IxIy \\ \sum IxIy & \sum Iy^2 \end{bmatrix}$$
- Then, we Non-Maximally Supress the corners to eliminate the corners that are too close to each other.
- Then, we threshold it and pick random 20 corners from the remaining corners.
- Then, we calculate the optical flow vectors for each of the 20 corners using the method described in part 1 but this time, we only calculate the flow for a small window around the chosen keypoints instead of the whole image.
- Then, we draw the optical flow vectors on each frame.
