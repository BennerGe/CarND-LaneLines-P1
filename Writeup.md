# **Finding Lane Lines on the Road** 


**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./examples/grayscale.jpg "Grayscale"
[input_image]: ./test_images_steps/image_step0.jpg  "Input Image"
[image_step1]: ./test_images_steps/image_step1.jpg  "Pipeline step 1: Grayscale"
[image_step2]: ./test_images_steps/image_step2.jpg  "Pipeline step 2: Gaussian smoothing"
[image_step3]: ./test_images_steps/image_step3.jpg  "Pipeline step 3: Canny edge detection"
[image_step4]: ./test_images_steps/image_step4.jpg  "Pipeline step 4: ROI"
[image_step5]: ./test_images_steps/image_step5.jpg  "Pipeline step 5: Hough transform"
[output_image]: ./test_images_steps/image_step6.jpg  "Output Image"

[challenge_image]: ./test_video_frames/3.8.jpeg  "Challenge image [3.8s]"
---

### Reflection

### Find line pipeline

My pipeline to detect lines on the road consisted of the following 5 steps: 

1. Convert image to grayscale 
2. Blurring the image with Gaussian blur
3. Find edges with Canny edge detection
4. Define the region of interest (ROI)
5. Hough transform to find lines using the detected edges

To show how the pipelines is working, I have included an image for every step. 

![Input Image][input_image]

The first two steps are necessary to prepare the image for the Canny edge detection. First the image is converted to grayscale and second possible noise and some incorrect gradients is removed by blurring the image with a Gaussian filter. 

After these two steps the image looks like this:

![Pipeline step 2][image_step2]

In order to find edges in the image, I have used the Canny implementation of OpenCV, with `low_threshold=50` and `high_threshold=150`. 

The result is a black and white image, which only shows edges. Any other informations of the image, like brightness, are removed. But at this point, the detected edges are only white, single points in the image. To find the lines there is another step necessary, namely the hough transform. But before this step, I restrict the region of the image, where the lines are supposed to be.  

![Pipeline step 3][image_step3]

To define the region of interest, where the lines should be, I have defined a polygon with values relative to the image size. 
Every pixel outside of this mask gets filled black. 

```python  
roi = np.array([[(im_shape[1]/10,im_shape[0]),
                 (im_shape[1]/2.2, im_shape[0]/1.65), 
                 (im_shape[1]-im_shape[1]/2.2, im_shape[0]/1.65), 
                 (im_shape[1],im_shape[0])]], dtype=np.int32)
             
```
![Pipeline step 4][image_step4]

Last but not least, the hough transform is applied to the image. 

Parameters: 

```python
    rho = 2 # distance resolution in pixels of the Hough grid
    theta = 1*np.pi/180 # angular resolution in radians of the Hough grid
    threshold = 35     # minimum number of votes (intersections in Hough grid cell)
    min_line_length = 5 #minimum number of pixels making up a line
    max_line_gap = 2    # maximum gap in pixels between connectable line segments
```

As a result we receive many line segments in the image. The goal is to summarize these into two lines: the left and the right lane marking. I have added the boolean parameter `extrapolate` to the `draw_lines()`function. If `extrapolate` is `false`, all line segments are drawn into the image seperately. But if `extrapolate` is actived, the function calculates the slope and intercept of every line. Lines with a slope `> 0` are assigned to the left lane marking, all others are assigned  to the right lane marking. 

```python
slope = ((y2-y1)/(x2-x1))
intercept = y1 - slope * x1
```

Last I take all line segments of each lane marking and calculate an average slope and intercept. This two lines are then drawn into the initially input image.  

![Output image][output_image]

To understand what's happening in the pipeline, I have introduced the parameters `show_images` and `substep`. 
If `show_images` is `true`, the image of every step is printed. The parameter `substep` affects which image is returned by the `find_line_pipeline` function. 

```python 
    substep = -1 # Return images of every step (Array)
    substep =  0 # Return last image of the pipeline
    substep =  1 # Return second last image of the pipline
    ...
```

### Challenge video 

With the pipeline from above the output of the two provided videos `solidWhiteRight.mp4` and `solidYellowLeft.mp4` is satisfying, but the result of the challenge video isn't good at all. To optimize the pipeline for the challenge video I have first determined the possible sources of error: 

1. The hood of the car is visible in this video 
2. The color of the street isn't consistent
3. There are shadows and stains on the street 

![Challenge image][challenge_image]

The hood of the car leads to a strong edge. In order to ignore this undesired edge, the ROI could be adjusted, so that the hood would be masked out. This is one possibility, but I choose another option, because adjusting the ROI for every video isn't practicable. 

My prefered way is to remove all lines which are more horizontally than vertically as lane markings should be. To achieve this, I improved the `draw_lines` function. The absolute value of the slope of every line segment has to exceed a defined threshold to be considered as part of a lane marking. With `min_slope = 0.4` the result of the challenge video is much more satisfying. 


### Potential shortcomings of the pipeline and possible improvements

There are different shortcomings with the current pipeline. First the lane markings cannot always be determined, sometimes there are no lines found, which can be assigned to the left or right lane marking. This could be improved by averaging the found lines over multiple image frames, instead of estimating the line position for every frame individually. 

Another shortcoming is the detection of edges which aren't part of the lane marking, for example different colors of the road surface, which can be seen in the above image of the challenge video. An improvement for this could be, to consider the color of the edges, since lane markings are always white or yellow. 