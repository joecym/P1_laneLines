## P1. Finding Lane Lines on the Road
## Joe Cymerman
## 9 June 2018

---

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report

[//]: # (Image References)

[image1]: ./test_images_output/solidWhiteCurve.jpg "Solid White Curve"
[image2]: ./test_images_output/solidWhiteRight.jpg "Solid White Right"
[image3]: ./test_images_output/solidYellowCurve.jpg "Solid Yellow Curve"
[image4]: ./test_images_output/solidYellowCurve2.jpg "Solid Yellow Curve 2"
[image5]: ./test_images_output/solidYellowLeft.jpg "Solid Yellow Left"
[image6]: ./test_images_output/whiteCarLaneSwitch.jpg "White Car Lane Switch"

---

### Reflection

### 1. Description of my pipeline. 

My pipeline consisted of six steps: apply grayscale, apply smooting, apply canny, apply mask, apply Hough Transform, and finally adding the original image to my lane lanes. The pipeline is contained in the function "process_image," below:

``` python
def process_image(image):
    # NOTE: The output you return should be a color image (3 channel) for processing video below
    # TODO: put your pipeline here,
    # you should return the final output (image where lines are drawn on lanes)
    
    # Apply Grayscale
    gray_img = grayscale(image)

    # Apply Smoothing
    blur_img = gaussian_blur(gray_img, kernel_size)
    
    # Apply Canny
    canny_img = canny(blur_img, low_threshold, high_threshold)

    # Apply the Mask
    masked_img = region_of_interest(canny_img, vertices)

    # Apply Hough Transform
    lines_img = hough_lines(masked_img, rho, theta, threshold, min_line_length, max_line_gap, y_horizon)

    # AddWeighted and Show
    result = weighted_img(lines_img, image, α=0.8, β=1., γ=0.)
    return result
```

Within the draw_lines() function, it was useful to keep the original plot of all the Hough lines visible until the end, as it was easier to see what was going on. Next I calculated the slope of each line within the double loop. I then applied a conditional statement that ignored small slope, and sorted the slopes on the left and right (along with their x and y positions) into seperate arrays. As long as these arrays were not empty, I could then determine the average slope and the average position for each lane. Then it was "just geometry" to draw the lines. I actually drew 4 lines, 2 for each lane. I used the average position that I calculated as the start point for my lines, which I then drew to the upper y limit of my trapezoid and then again to the bottom of the screen.

My pipeline consisted of 5 steps. First, I converted the images to grayscale, then I .... 

In order to draw a single line on the left and right lanes, I modified the draw_lines() function by ...

If you'd like to include images to show how the pipeline works, here is how to include an image: 

![alt text][image1]


### 2. Identify potential shortcomings with your current pipeline


One potential shortcoming would be what would happen when ... 

Another shortcoming could be ...


### 3. Suggest possible improvements to your pipeline

A possible improvement would be to ...

Another potential improvement could be to ...
