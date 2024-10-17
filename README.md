# Robot Localization Project
### Kate McCurley and Charlie Mawn


## Project Goals
- Improve our skills using ROS2
- Learn about robot localization
- Understand the particle filtering algorithm
- Develop debugging strategies using visualizers
- Expand our knowledge in the trade offs between computational accuracy and algorithm efficiency

## Localization and Particle Filtering

In our design, we make one key assumption: the robot has a map of the region, but it does not know where in the map it is. On top of this, we also have a known starting pose for the robot. So, our primary objective is to use the LIDAR scanner to estimate where the robot is. In conjunction with this, we also estimate how far the robot has gone by keeping track of its wheel velocities and turns. 

### Steps to Accomplish This

- **Initialization**
    - We compute a set of particles that each have a pose (reprisented as x,y,θ)
    - To compute this initial set of particles, We used a normal distribution centered around the robot starting pose
    - We give all of these initial particles a weight given by 1 divided by the amount of particles created
- **Motion Update**
    - Given two of our subsequent odometry poses of the robot, we update the particles
    - To implement this we created a transformation matrix based on the previous and current x,y,θ values. We then multiply the pose of each particle by this matrix to get the new particle positions
- **Observation Update**
    - Using the LIDAR scanner, we determine a new weight for each particle based its compatibility
    - To determine this new weight, we compared the distance to the closest object according to the LIDAR scan to the distance to the closest object according to the occupancy field
    - The weight is then calculated by taking the inverse of this difference between distances and then normalizing all the weights together. 
- **Locate**
    - Given our new weighted set of particles, we determine the estimated robot pose
    - to determine the pose, we take a weighted average of all the particles, then convert that average from x,y,θ values to a robot pose
- **Iterate**
    - Given our weighted set of particles, resample our new set of particles for the next iteration
    - To do this, we take the highest 25% of weights, and keep them for the next round of iteration
    - We remove the other 75% of particles with lower weights and create new particles using the normal distribution we described above
    - This ensures that we keep the best particles, while still allowing for new, potentially more accurate particles, to be created

## Design Decisions

One of the first design decisions we encountered was how we wanted to initialize the particle cloud. Our initial idea was to create a set of particles located in a grid patern evenly spaced across the map. After some research and consideration, we decided to change this to a normal distribution. Our goal with this decision was to make the initial particle cloud less deterministic, and more of a random assortment based on the starting robot location. We decided that we needed to use two normal distributions: one for the location (x,y) and one for the heading (θ). We did have to make some (educated) arbitrary decisions with this. We decided that the standard deviation of the location was .25m, and  π/2 radians as the standard deviation for the angle heading. We believe that using a standard deviation rather than an evenly spaced grid gives us a more accurate starting point for the particle cloud without sacrificing any computational efficiency. 

## Challenges  

Our biggest issue is that in the vizualization, our particles do not move. We are currently judging the accuracy of our implementation based on the relative locations of the laser scan and the map, as well as the position of the robot in the rviz vizualization. Using print-based debugging we were able to confirm that the locations and weights of our particles are changing in the code, but not in a way that displays in Rviz - this is likely an issue of compatibility between the given template and our code, where the values we're changing are not the ones used to generate the particle visualization, but we have not yet been able to pinpoint / fix this error. 

## Expansion

Given more time, we would have liked to expand upon this project in a few ways. Many of these would involve more trial and error around the design decisions we made in our implementation to determine what choices would be best - for example, trying different ways of updating the particle weights from the laser (maybe considering angle and not just distance) or different ways of calculating the robot pose (maybe not considering every single point, or using a more complex formula than a weighted average). This trial and error could also be with the values we used, many of which were arbitrarily selected, including values such as standard deviation in the distribution of the particles or percentage of particles to keep. 

Another major way we would have liked to improve upon this project was by not giving the robot a known map or starting location. We believe that in a real world situation where a search and rescue robot is used, it is unlikely that the map will be given to the robot. For example, even if there is a building with a known layout, there is always a possibility of debris being in the way of the normal map. So, it is important that a robot can create a map of its environment as it traverses it. To accomplish this, we likely would have implemented a system of storing all the LIDAR scans of the robot. Using our estimated location of the robot, we would overlay the scans on top of each other to map out the environment as it moved.

## Lessons for Future Projects

One lesson we learned throughout this project is the importance of synchronous work. During Covid, a lot of work became very asynchronously-accessible, which can be good in a lot of ways. People don't have to schedule time to meet, and during Covid it was especially important for stopping the spread. However, in this asynchronous world we all now work in, the importance of synchronous work seemed to dwindle away. For the first portion of this project, we tried to do work seperately, but we always ended having a bunch of questions and issues with implementation. We found that even when working on seperate portions of the project, it is valuable to be working together so we always had another person to bounce ideas off of, and someone to help with misconceptions. We consistantly made it through mental road-blocks by simply explaining our issues aloud and talking through them together. So, if we have one piece of advice for future programmers working on projects, it is to make sure to do some synchronous work, as it can make the whole process run more smoothly. 





