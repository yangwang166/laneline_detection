# **Finding Lane Lines on the Road**


**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on my work in a written report


[//]: # (Image References)

[origin]: http://chuantu.biz/t6/348/1532625928x-1404829133.png "Origin Image"
[gray_scale]: http://chuantu.biz/t6/348/1532625963x-1404829133.png "Grayscale"
[blur]: http://chuantu.biz/t6/348/1532625981x-1404829133.png "Blur"
[canny]: http://chuantu.biz/t6/348/1532626000x-1404829133.png "Canny Transformation"
[roi]: http://chuantu.biz/t6/348/1532626018x-1404829133.png "Region of Interest"
[after_roi]: http://chuantu.biz/t6/348/1532626034x-1404829133.png "Apply Region of Interest"
[hough]: http://chuantu.biz/t6/348/1532626053x-1404829133.png "Hough Transformation"

---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.


Let's have a look at the origin image:

![origin][origin]

First of all, I converted the images to grayscale:

![gray_scale][gray_scale]

Then, I used Gaussian filter to the grayscale image into blur image, which can suppress noise and can improve the effect when we do edges detection:

![blur][blur]

Then I used Canny Edge Detection to get the raw line:

![canny][canny]

Since I only interested in a portion of image, so I created a region of interest as follow:

![roi][roi]

Then I apply the region of interest mask to the edge image, I got:

![after_roi][after_roi]

Finally I using Hough Transformation to convert the pixels into lines segments:

![hough][hough]


In order to draw a single line on the left and right lanes, I modified the draw_lines() function by grouping the lines into left group and right group, then using `np.ployfit` to average the left line and the right line. `np.ployfit` gives me the slope and index of the two lines, then I choose a starting point, like `y_top=320`, to calculate the start point and end point of the left line and right line. Here is the actual code piece:

```python
def draw_lines2(img, lines, color=[255, 0, 0], thickness=4):
    left_x_points = np.array([])
    left_y_points = np.array([])
    right_x_points = np.array([])
    right_y_points = np.array([])

    y_max = img.shape[0]
    x_max = img.shape[1]
    y_top = 320

    if lines is Null: return
    for line in lines:
        for x1, y1, x2, y2 in line:
            if (y2 - y1) * (x2 - x1) == 0:
                continue
            slope = (y2 - y1) / (x2 - x1)

            if slope < 0: # left line
                if (x2 < x_max / 2) and (x1 < x_max / 2): # to make sure the line seg is in the left zone
                    left_x_points = np.append(left_x_points, x1)
                    left_x_points = np.append(left_x_points, x2)
                    left_y_points = np.append(left_y_points, y1)
                    left_y_points = np.append(left_y_points, y2)
            else: # right line
                if (x2 > x_max / 2) and (x1 > x_max / 2): # to make sure the line seg is in the right zone
                    right_x_points = np.append(right_x_points, x1)
                    right_x_points = np.append(right_x_points, x2)
                    right_y_points = np.append(right_y_points, y1)
                    right_y_points = np.append(right_y_points, y2)

    if left_x_points.size < 1:
        left_x_points = np.append(left_x_points, 1)
    if left_y_points.size < 1:
        left_y_points = np.append(left_y_points, 1)
    if right_x_points.size < 1:
        right_x_points = np.append(right_x_points, 1)
    if right_y_points.size < 1:
        right_y_points = np.append(right_y_points, 1)  

    left_line_deg = np.polyfit(left_x_points, left_y_points, 1) # degree = 1
    right_line_deg = np.polyfit(right_x_points, right_y_points, 1)

    x1_left = int((y_top - left_line_deg[1]) / left_line_deg[0])
    x2_left = int((y_max - left_line_deg[1]) / left_line_deg[0])
    x1_right = int((y_top - right_line_deg[1]) / right_line_deg[0])
    x2_right = int((y_max - right_line_deg[1]) / right_line_deg[0])        

    left_lane = np.array([[[x2_left, y_max, x1_left, y_top]]])
    right_lane = np.array([[[x2_right, y_max, x1_right, y_top]]])

    lines = np.concatenate((left_lane, right_lane))
    for line in lines:
        for x1,y1,x2,y2 in line:
            cv2.line(img, (x1, y1), (x2, y2), color, thickness)
```


Here is the video:
* Before Hough Transformation: https://youtu.be/rwV12XTlC2M
* After Hough Transformation1: https://youtu.be/lKioHvaOwVw
* After Hough Transformation2: https://youtu.be/nIZe8DGnJq0
* Final Challenge: https://youtu.be/C36vlHwKwa4


### 2. Identify potential shortcomings with your current pipeline


One potential shortcoming would be what would happen when the camera is placed at a different position; and one or more lane lines are missing; different light condition.


### 3. Suggest possible improvements to your pipeline

A possible improvement would be to dynamically update the region of interest. And if the lane line is missing or not detected, we can guess the line based on previous observation. And also choose another colour space instead of RGB maybe useful to ease the bad light condition.

Yang
