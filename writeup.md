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
s    # NOTE: The output you return should be a color image (3 channel) for processing video below
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

Within the draw_lines() function, it was useful to keep the original plot of all the Hough lines visible until the end, as it was easier to see what was going on. Next I calculated the slope of each line within the double loop. I then applied a conditional statement that ignored small slopes, and sorted the slopes on the left and right (along with their x and y positions) into seperate arrays. As long as these arrays were not empty, I could then determine the average slope and the average position for each lane. Then it was "just geometry" to draw the lines. I actually drew 4 lines, 2 for each lane. I used the average position that I calculated as the start point for my lines, which I then drew to the upper y limit of my trapezoid and then again to the bottom of the screen.

``` python
def draw_lines(img, lines, y_horizon, y_floor, color=[255, 0, 0], thickness=5):
    """
    NOTE: this is the function you might want to use as a starting point once you want to 
    average/extrapolate the line segments you detect to map out the full
    extent of the lane (going from the result shown in raw-lines-example.mp4
    to that shown in P1_example.mp4).  
    
    Think about things like separating line segments by their 
    slope ((y2-y1)/(x2-x1)) to decide which segments are part of the left
    line vs. the right line.  Then, you can average the position of each of 
    the lines and extrapolate to the top and bottom of the lane.
    
    This function draws `lines` with `color` and `thickness`.    
    Lines are drawn on the image inplace (mutates the image).
    If you want to make the lines semi-transparent, think about combining
    this function with the weighted_img() function below
    """
    # initialize arrays
    leftslope = np.empty([0,0])
    rightslope = np.empty([0,0])
    leftx = np.empty([0,0])
    rightx = np.empty([0,0])
    lefty = np.empty([0,0])
    righty = np.empty([0,0])
    
    for line in lines:
        for x1,y1,x2,y2 in line:
            # plot all of the lines just to see what's going on
            #cv2.line(img,(x1,y1),(x2,y2),(0,255,0),10)
            
            # plot one line on each side
            slope = ((y2-y1)/(x2-x1))
            if np.absolute(slope) > 0.3:  # filter out any small slopes
                if slope < 0:
                    leftslope = np.append(leftslope,slope)
                    leftx = np.append(leftx,(x1+x2)/2)
                    lefty = np.append(lefty,(y1+y2)/2)
                else:
                    rightslope = np.append(rightslope,slope)
                    rightx = np.append(rightx,(x1+x2)/2)
                    righty = np.append(righty,(y1+y2)/2)
    
    if (leftslope.size != 0) and (rightslope.size != 0):
        leftslopeavg = np.average(leftslope)
        leftxposavg = np.average(leftx)
        leftyposavg = np.average(lefty)
        rightslopeavg = np.average(rightslope)
        rightxposavg = np.average(rightx)
        rightyposavg = np.average(righty)
        
        ## These are the lines (left and right) that go from the average point to the top of the picture
        # ((y2-y1)/(x2-x1)) = m , y2 = 290
        #  (290 - yavg)/(x2 - xavg) = m
        # x2 = (290 - yavg)/m + xavg
        y2 = y_horizon
        x2 =((y2 - leftyposavg)/leftslopeavg + leftxposavg)
        x2 = x2.astype(int)
        cv2.line(img, (leftxposavg.astype(int), leftyposavg.astype(int)), (x2, y2), color, thickness)
        
        x2 =((y2 - rightyposavg)/rightslopeavg + rightxposavg)
        x2 = x2.astype(int)
        cv2.line(img, (rightxposavg.astype(int), rightyposavg.astype(int)), (x2, y2), color, thickness)
        
        ## These are the lines (left and right) that go from the bottom of the picture to the average point
        # ((y2-y1)/(x2-x1)) = m ,  y2 = yavg
        #  (yavg - 0)/(xavg - x1) = m
        # x1 = -(yavg - 0)/m + xavg
        y1 = y_floor
        x1 =-(leftyposavg - y1)/leftslopeavg + leftxposavg
        x1 = x1.astype(int)
        cv2.line(img, (x1, y1),(leftxposavg.astype(int), leftyposavg.astype(int)), color, thickness)
        
        x1 =-(rightyposavg - y1)/rightslopeavg + rightxposavg
        x1 = x1.astype(int)
        cv2.line(img, (x1, y1),(rightxposavg.astype(int), rightyposavg.astype(int)), color, thickness)
```
Here are the images of my test output:

![alt text][image1]
![alt text][image2]
![alt text][image3]
![alt text][image4]
![alt text][image5]
![alt text][image6]


### 2. Identify potential shortcomings with your current pipeline


In the optional challenge, the only modification I made was to resize the video so that my mask from the previous portion didn't require any modifications. However, I suppose there is a smarter way to apply the mask based on other features of the image. 

Additionally, my lines went off course when the pavement changed from blacktop to concrete. Yellow on concrete was especially rough. I suppose this leads into the next lessons on how to fix this...


### 3. Suggest possible improvements to your pipeline

The constants for Canny and Hough will need to be applied more reliably, instead of manually "guessing and checking."

Also, I'm not sure if my "draw_lines" function is the most efficient. It would be nice to get rid of the 'for' loops and replace them with matrix operations. Maybe there is a better way to manipulate arrays than numpy. 
