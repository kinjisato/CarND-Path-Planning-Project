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

If there is no car ahead, target maximum speed would be 49.5mph. When there is other car ahead, my car is going to adjust the speed to trace its car by adjusting acceleration.


```C++
            double dvel = 0.0;
            const double MAX_SPEED = 49.5;
            const double MAX_ACC = .224;
            double target_speed = MAX_SPEED;
            
            
            if(target_speed > car_vel){
                target_speed = car_vel;
            }
            
            double vel_diff = target_speed - car_speed;
            dvel = vel_diff * MAX_ACC / 10;
            if(dvel > MAX_ACC){
                dvel = MAX_ACC;
            } else if(dvel < -MAX_ACC){
                dvel = -MAX_ACC;
            }
```
---
After that, 
1. create a list of widely spaced(x,y) waypoints, evenly spaced at 30m
2. reference x,y yaw states, 
   Use two points that make the path tangent to the previous path's end point
3. In Frenet add evenly 30m spaced points ahead of the starting reference
4. create a spline
5. Calculate how to break up spline points so that we travel at our desired refefence velocity
6. Fill up to the rest of our path planner after filling it with previous poits, here we will always output 50 pints.
7. rotate back to normal after rotating it earlier

These are the same and copied from lecure video.
And then, finally I got 
```C++
                next_x_vals.push_back(x_point);
                next_y_vals.push_back(y_point);
```
