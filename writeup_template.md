# **Finding Lane Lines on the Road** 


---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[grayscale_image]: ./test_images_output/solidWhiteCurve.jpg_grayscale.jpg "Grayscale"
[blurred_image]: ./test_images_output/solidWhiteCurve.jpg_blurred.jpg "Blurred"
[canny_image]: ./test_images_output/solidWhiteCurve.jpg_cannyEdges.jpg "Canny"
[masked_image]: ./test_images_output/solidWhiteCurve.jpg_grayscale.jpg "Masked"
[line_image]: ./test_images_output/solidWhiteCurve.jpg_final.jpg "Line"

---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

**My pipeline consisted of 5 steps:**
- Convert to grayscale: Converting to grayscale merges the color data to make subsequent processing steps easier
![grayscale_image]
- Gaussian blur: I applied a gaussian blur with a kernel size of 9. This removes noise that can be picked up as edges by subsequent steps. 
![blurred_image]
- Canny edge detection: Used Canny edge detection to find edges in the entire blured picture.
![canny_image] 
- Masked region: Mask the region of interest using a quadrilateral shape. Only keep the edges that are within the mask
![masked_image]
- Hough transform: Perform the hough transform on the masked edges. This results in a collection of lines representing edges in the mask
- Fliter: Out of all the lines, find the best left line and the best right line and draw them. This process is discussed in more detail in the next section
![line_image]

The pipeline is implemented in the pipeline() and process_image() functions. Parameters for the pipeline can be found in the pipelineParameters class. 

**In order to draw a single line on the left and right lanes, I modified the draw_lines() function by ...**
I created a LaneLines class to hold a left lane line (leftLine) and a right lane line (rightLine). Each lane line consists of an internal state variable representing, slope and intercept of the lane line. When the draw_lines() function is called, instead of drawing lines (so it's a bit of a misnomer now) it pushes the data onto a list of lines. The LaneLines class does the following with this list:
- calculate the slope and intercept of each line (I think I'd actually prefer to do this in polar cooridnates to avoid getting infinite slope and intercept for vertical lines, but I didn't have time to do this)
- for every line in the list of found lines, compare it to the expected slope and intercept of leftLine. Combine the errors into a cost function
- choose the line that has the lowest cost function. This is the left lane line of this frame. 
- Average this newly found line's slope and intercept into leftLine's internal state variables. I've currently set the code to average for 20 frames. This helps to smooth out the response when the car bounces, while being able to react somewhat rapidly to changin conditions. 
- plot leftLine
- repeat for rightLine





### 2. Identify potential shortcomings with your current pipeline


1. As evidenced by my results from the challenge video, the algorithm has problems with low contrast lines. Such as faded lines on a white concrete road
2. The parameters have to be tuned. I've tuned them to work well with the example images and videos, but there are many more situations that may not work well with these parameters.
3. It needs lane lines. This will not work in a parking lot or a back country road where there may not be lane lines. 
4. It takes a while to generate the video. For this to work on self-driving cars, the response needs to be real time. This algorithm is too slow. 
5. The bounding box and looking for lines based on the last set of lines, means it might not be able to react to sharp turns where the slope of the lines might change drastically and lie outside the bounding box. 



### 3. Suggest possible improvements to your pipeline

1. Make use of color data, instead of compressing the color into a grayscale image, I could have done the analysis on each color, and make a decision based on the combination results from processing all three images. 
2. Dynamically adjust the bounding box. If the lanes narrow or widen, the lines may fall outside of the bounding box, or we might have too many extraneous lines. I could adjust the sides of the bounding box to be a bit larger than my calculated left and right lines. 
3. Process images with different settings and choose the best results. For example, I could up the Canny thresholds. If I can't find anything, drop the thresholds by 10% and search for thresholds again. This way I have the least number of extraneous lines that can mess with the pipeline. 
