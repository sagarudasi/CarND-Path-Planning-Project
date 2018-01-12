### Project Writeup

---

**CarND Path Planning**
Goals of this project are the following:
* The car is able to drive at least 4.32 miles without incident
* The car drives according to the speed limit.
* Max Acceleration and Jerk are not Exceeded.
* Car does not have collisions
* The car stays in its lane, except for the time between changing lanes.
* The car is able to change lanes
* Implementation writeup


[//]: # (Image References)
[image1]: ./images/1.png
[image2]: ./images/2.png
[image3]: ./images/3.png
[image4]: ./images/4.png


### [Rubric](https://review.udacity.com/#!/rubrics/1020/view) Points
#### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

Please check [main.cpp](https://github.com/sagarudasi/CarND-Path-Planning-Project) for the implemenation.

There are two parts of the solution

1. Trajectory Generation
-  'Project Walkthrough and Q&A' by Aaron & David explains this very well. I have used it as reference for this implementation.
    Also, I understood the basic concept behind it.


2. Behavior Planning
- Starting of the car.
     Car must gradually start to avoid the longitudinal jerk due to acceleration. This part is already taken care in the trajectory generation and explained in the 'Project Walkthrough and Q&A'

  The following image shows how the car gradually starts in the center lane.

  ![alt text][image1]

  Notice the speedometer on the right.

- This is achived by the following code

```
line 234:	
    // Starting Lane - 1
    int lane = 1;

    // Have reference velocity to target
    double ref_vel = 1; //mph
```

- I then increase the car speed gradually only if there is no vehicle in the same lane which car is in.
```
line 353:	
    else if (ref_vel < 49.5)
    {
        ref_vel += 0.224;
    }
``` 

- Car will be the same lane if there is no vehicle till 30m from our car. To detect the vehicle, we compare S-distance of each car to our car in the same lane. 
```
line 303:
    if ((check_car_s > car_s) && ((check_car_s - car_s) < 30))
    {

        too_close = true;

    }
```

- If the vehicle is too close, we set a boolean value too_close to true which is considered further in the code.
- We then have to switch lane in order to keep moving at desired velocity. 
- For this purpose I implemented a funtion that checks the desired lane if its safe to move there. If its safe, it returns true otherwise false.

```
line 169:
    bool is_lane_safe( int clane, auto &sensor_fusion, int prev_size, double car_s )
```

- We then consider different scenarios one by one.
    * To switch lanes, two conditions must be true. 
        
        1. too_close must be true
        2. Speed of our car must be less than 40

```
line no: 312

    if (too_close)
    {

        // Object is too close, start slowing down to safe lane change velocity
        ref_vel -= 0.224;

        // Attempt lane change only if velocity is less than desired
        if( ref_vel < 40 ){

```
- 
    * Now once we achive desired velocity for safe lane change, we check the below conditions

        1. If we are in lane 0 or 2, we check and move to lane 1
        2. If we are in lane 1 then we check lane 0 first and then lane 2

```
line: 322
    if (lane == 2 || lane == 0)
    {
        if(is_lane_safe(1, sensor_fusion, prev_size, car_s)){
            // Lane 1 is safe to switch to
            std::cout << "Switched to lane 1" << std::endl;
            lane = 1;
        }
    }
    else{
        // We are in center lane
        // Attemp lane 0 first and then lane 2
        if(is_lane_safe(0, sensor_fusion, prev_size, car_s)){
            // Lane 0 is safe to switch to
            std::cout << "Switched to lane 0" << std::endl;
            lane = 0;
        }
        else if(is_lane_safe(2, sensor_fusion, prev_size, car_s)){
            // Lane 2 is safe to switch to
            std::cout << "Switched to lane 2" << std::endl;
            lane = 2;
        }
    }

```

### Result
I was successfully able to drive the car for 4.62 miles without any collison.

  ![alt text][image2]
  ![alt text][image3]
  ![alt text][image4]


### Challenges and Issues
1. I am not considering the vehicles which are behind us for lane changing. This let to collision at around 4.62 miles. 
2. If the front vehicle applies brake too strong, then also there is possiblity of collision. WAR can be to go a bit slow or to decrease our speed rapidly as per the front vehicle.

