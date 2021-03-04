# **Finding Lane Lines on the Road** 

The goals / steps of this project are the following:

	* Make a pipeline that finds lane lines on the road
	* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./writeup/roi.png "Roi visualisation"
[image2]: ./writeup/ht.png "Hue transformation and line markings"
[image3]: ./writeup/final.png "Final output"
---

### Reflection

### 1. Pipeline description

The image processing pipeline consists of 7 steps:

1. Image resizing - this is implemented to improve the robustness in the *challenge* video
2. Grayscale conversion
3. Edge detection - using Canny algorithm, with the value of **30** for low threshold and **100** for high threshold
4. Selecting region of interest - I used quadrilateral to select this region. To improve the robustness in the *challenge* video, the quadriliteral is not at the bottom of the screen, but rather it's moved upwards a little. This doesn't affect the performance on the standard videos much, but it has a big impact in the challenge video. 

![Roi visualisation][image1]

5. Blurring the image - I used the kernel of size **3**
6. Hough transformation and marking the lanes - Rho value **2**, theta **pi/110**, threshold **110**, minimum line length **10** and maximum line gap **3**

![Hue transformation and line markings][image2]

7. Combining the original image and lane markings

![Final output][image3]

Most of these steps are using the predefined functions, which were part of the project framework. 

As a starting point with some sensible values, I used the values I identified as sensible in the curriculum quizzes. Once I got the pipeline running, I started with parameter fine-tuning, making small changes in the parameter values and checking how the output is affected. It was rather easy to find the lines in the static images, however, it required many more experiments to get good results in the video files. Running enough experiments with different parameters essentially gave me the intuition how changes in different parameters are going to affect the output.  

Once the pipeline was able to identify the lanes quite well, I implemented changes in *draw_lines()* function. As suggested in the function comment, I split the lines in two categories based on their slope (positive/negative). Each category was then separately used to draw a single line - either on the left or on the right. I tested multiple approaches to drawing the lines:

- drawing the line based on average line coordinates and extrapolation - first I calculated and average of the coordinates (the result is 4 coordinates - x1, y1 and x2, y2) and the then I calculated the slope and intercept values for this line. The result was a linear equation, which I used to calculate the coordinates of extreme values for the closest and farthest point of the lane line. Although this solution generally works well, I noticed the lane markings were very jiggly, based on how well did I detect the lines in hough transformation.
- drawing the line based on median coordinate values and extrapolation - the difference in this case is that I used median value to find the corrdinates used for calulating slope and intercept. However, this solution wasn't noticeably better than the previous one and the line markgins were still jiggly.
- combination of previous two approaches - this is the final solution. First I calculated median values for the coordinates and then I filtered out all outliers in line data (separately for both positive and negative slope). Using trial and error method, I figured out I got good results by accepting lines with coordinates in interval with the span of 35% of the median value. I also filtered out all lines with slope smaller than 0.4 (positive or negative). Once the outliers were filtered out, I used the coordinate averaging and extrapolation method to draw the line.  


### 2. Potential pipeline shortcomings

The pipeline, as designed, is not extremly robust and could fail to detect the lanes on many occasions, e.g.:

- different lane colors - even though the range of shades od gray used for the line detection is quite wide, we could still fail to detect them in specific cases
- low contrast between the lanes and the road or a lot of shadows on the road
- very short lines - if the lane lines are very short, the algorithm would simply ignore them
- big gaps between the lines - if the gaps are too big, the algorithm may fail to identify the correct slope of the lane line
- bad road quality - this is especially visible in the challenge video, where we get a lot of false positivies 
- missing lane lines - if the lines are missing due to road conditions, the car will lose the track of it's position


### 3. Possible pipeline improvements

The issues identified in chapter 2 could be mitigated in multiple ways. It seems like there is still enough space for an improvement in parameter tuning, as these have the largest impact on the reliability of the algorithm. The algorithm seems to be especially struggling with bad road quality and if there is low contrast between the road and the lanes, so picking better parameters could help there a lot. This should be also accompanied with better outlier removal, as these tend to skew line lanes a lot. Another improvement could be sharing the information about the lane position between the video frames, as we can expected them to be more or less in the same place as in previous frame. This would also help if the lane lines will go missing, since the car can assume where they should be. 

