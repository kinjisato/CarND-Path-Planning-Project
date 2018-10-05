## CarND-Path-Planning-Project
### Kinji Sato / 5th Octorber 2018

---

[//]: # (Image References)
[image1]: ./results/Rubric01.png
[image2]: ./results/Rubric02.png
[image3]: ./results/results01.png

[video1]: ./project_video.mp4

## [Rubric](https://review.udacity.com/#!/rubrics/1020/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup

### Compilation

![alt text][image1]

### Valid Trajectories

![alt text][image2]
![alt text][image3]


### Reflection

As fisrt, the lane is found where the car (not ego) is running.

```C++
                float d = sensor_fusion[i][6];
                int car_lane = -1;
                // check which lane car is running
                if (d > 0 && d < 4){
                    car_lane = 0;
                } else if (d > 4 && d < 8){
                    car_lane = 1;
                } else if (d > 8 && d < 12){
                    car_lane = 2;
                }
                if (car_lane == -1){
                    // out of the lanes
                    continue;
                }
```

After that, the car speed and its position at XY cordinate are checked. (same as lecture code).
When the car is running on my lane, the closest car speed is stored.
When the car is running on other lane, and the potision is between -15 behind and 30 ahead, this is stored as the car is on the lane.

```C++
                double vx = sensor_fusion[i][3];
                double vy = sensor_fusion[i][4];
                double check_speed = sqrt(vx*vx+vy*vy);
                double check_car_s = sensor_fusion[i][5];
                check_car_s += ((double)prev_size* .02*check_speed);
                
                if (car_lane == lane){ // car is in my lane
                    if((check_car_s > car_s) && ((check_car_s - car_s) < 30)){
                        car_ahead = true;
                        if((check_car_s - car_s) < s_closest){
                            s_closest = check_car_s;
                            car_vel = check_speed*2.24;
                        }
                    }
                } else if(car_lane == (lane -1) ){ // car is in left of my lane
                    if(((check_car_s - car_s) < 30) &&( (check_car_s - car_s) > -15)){
                        car_left = true;
                    }
                } else if(car_lane  == (lane +1)){ // car is in right of my lane
                    if(((check_car_s - car_s) < 30) && ((check_car_s - car_s) > -15)) {
                        car_right = true;
                    }
                }

```

If there is other car in front of my car, my car is going to change to the lane which no other car on the lane. If all lane is not free, my car keeps current lane. 
And when my car is not running center lane, and the center lane is free, my car is going to back to center.


```C++
            if(car_ahead == true){
                if((car_left == false)&&(lane > 0)){
                    lane -= 1;
                } else if((car_right == false)&&(lane < 2)){
                    lane += 1;
                }
            } else {
                if(lane != 1){
                    if(((lane == 0)&&(car_right == false)) || ((lane == 2)&&(car_left == false))){
                        lane = 1;
                    }
                }
            }
```





The code for this step is contained in the first code cell of the IPython notebook (or in lines # through # of the file called `some_file.py`).  

I started by reading in all the `vehicle` and `non-vehicle` images.  Here is an example of one of each of the `vehicle` and `non-vehicle` classes:

![alt text][image1]

I then explored different color spaces and different `skimage.hog()` parameters (`orientations`, `pixels_per_cell`, and `cells_per_block`).  I grabbed random images from each of the two classes and displayed them to get a feel for what the `skimage.hog()` output looks like.

Here is an example using the `YCrCb` color space and HOG parameters of `orientations=8`, `pixels_per_cell=(8, 8)` and `cells_per_block=(2, 2)`:


![alt text][image2]

#### 2. Explain how you settled on your final choice of HOG parameters.

I tried various combinations of parameters and...

#### 3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).

I trained a linear SVM using...

### Sliding Window Search

#### 1. Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?

I decided to search random window positions at random scales all over the image and came up with this (ok just kidding I didn't actually ;):

![alt text][image3]

#### 2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?

Ultimately I searched on two scales using YCrCb 3-channel HOG features plus spatially binned color and histograms of color in the feature vector, which provided a nice result.  Here are some example images:

![alt text][image4]
---

### Video Implementation

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)
Here's a [link to my video result](./project_video.mp4)


#### 2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

I recorded the positions of positive detections in each frame of the video.  From the positive detections I created a heatmap and then thresholded that map to identify vehicle positions.  I then used `scipy.ndimage.measurements.label()` to identify individual blobs in the heatmap.  I then assumed each blob corresponded to a vehicle.  I constructed bounding boxes to cover the area of each blob detected.  

Here's an example result showing the heatmap from a series of frames of video, the result of `scipy.ndimage.measurements.label()` and the bounding boxes then overlaid on the last frame of video:

### Here are six frames and their corresponding heatmaps:

![alt text][image5]

### Here is the output of `scipy.ndimage.measurements.label()` on the integrated heatmap from all six frames:
![alt text][image6]

### Here the resulting bounding boxes are drawn onto the last frame in the series:
![alt text][image7]



---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  

