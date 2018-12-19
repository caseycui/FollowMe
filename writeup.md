## Project: Search and Sample Return
### Writeup Template: You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

---


**The goals / steps of this project are the following:**  

**Training / Calibration**  

* Download the simulator and take data in "Training Mode"
* Test out the functions in the Jupyter Notebook provided
* Add functions to detect obstacles and samples of interest (golden rocks)
* Fill in the `process_image()` function with the appropriate image processing steps (perspective transform, color threshold etc.) to get from raw images to a map.  The `output_image` you create in this step should demonstrate that your mapping pipeline works.
* Use `moviepy` to process the images in your saved dataset with the `process_image()` function.  Include the video you produce as part of your submission.

**Autonomous Navigation / Mapping**

* Fill in the `perception_step()` function within the `perception.py` script with the appropriate image processing functions to create a map and update `Rover()` data (similar to what you did with `process_image()` in the notebook). 
* Fill in the `decision_step()` function within the `decision.py` script with conditional statements that take into consideration the outputs of the `perception_step()` in deciding how to issue throttle, brake and steering commands. 
* Iterate on your perception and decision function until your rover does a reasonable (need to define metric) job of navigating and mapping.  

[//]: # (Image References)

[image1]: ./imgs/auto_mode_run.PNG
[image2]: ./calibration_images/example_grid1.jpg
[image3]: ./calibration_images/example_rock1.jpg 
[image4]: ./calibration_images/example_edge.jpg 
[image5]: ./calibration_images/example_obs1.jpg 


## [Rubric](https://review.udacity.com/#!/rubrics/916/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  

You're reading it!

### Notebook Analysis
#### 1. Run the functions provided in the notebook on test images (first with the test data provided, next on data you have recorded). Add/modify functions to allow for color selection of obstacles and rock samples.
For rock sample detection, I have utilitzed the fact that rocks are yellow in color. Therefore, I created a filter which set the R and G channels to be above 100, and B channel to be below 50.

![alt text][image3]

The obstacles are relatively dark compared to the sand, therefore, I created a color threshold for obstacles so that if all channels are below 100, it is identified as obstacles.

![alt text][image5]

#### 1. Populate the `process_image()` function with the appropriate analysis steps to map pixels identifying navigable terrain, obstacles and rock samples into a worldmap.  Run `process_image()` on your test data using the `moviepy` functions provided to create video output of your result. 
In the process_image() step, the navigable terrain, obstacles and rock samples are mapped into a worldmap.

The video directory is:  ./output/test_mapping.mp4 

### Autonomous Navigation and Mapping

#### 1. Fill in the `perception_step()` (at the bottom of the `perception.py` script) and `decision_step()` (in `decision.py`) functions in the autonomous mapping scripts and an explanation is provided in the writeup of how and why these functions were modified as they were.
In the perception_step(), the following filters and transforms are performed on the image: 1) color thresholding to identify rocks, navigable terrain, and obstacles 2) perspective transform to map image from front view to birds-eye view 3) convert image data to rover-centric coordinates 4) covert rover-centric coordinates to world coordinates 5) update world map 6) covert to polar coordinates for rover control purposes
In the decision_step(), the main update is to set the rover to the mean of navigable angles calculated from above step 

#### 2. Launching in autonomous mode your rover can navigate and map autonomously.  Explain your results and how you might improve them in your writeup.  

**Note: running the simulator with different choices of resolution and graphics quality may produce different results, particularly on different machines!  Make a note of your simulator settings (resolution and graphics quality set on launch) and frames per second (FPS output to terminal by `drive_rover.py`) in your writeup when you submit the project so your reviewer can reproduce your results.**

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  

##### Result
The rover is able to run in autonomous mode and start mapping and finding rock samples.

![alt text][image1]

I have tried to identify the edges of the path such as the following to set the rover steer to go along edges. But didn't find a good way to detect it in time.

![alt text][image4]

I have also tried to set the mean angles to be biased towards the edge, but it's hard to set one rule for all cases.

##### Improvements & Future work
This is a fun project, it's relatively easy to reach a passing point, so I was hoping to get back and work on it later but never got the time. 

For improvement, there're several things that can be done. First, implement the pickup rock path so that when a rover finds a rock, it will navigate towards the rock and pick it up. Second, obstacle avoidance needs to be improved, sometimes the rover gets stuck behind an obstacle. Third, find ways to navigate the rover to explore new terrains.

Inside these tasks, there're subtasks like, how to identify the edges (or something else) in order to allow the rover to pick up rocks more effectively; implement way-outs from obstacles, etc.
