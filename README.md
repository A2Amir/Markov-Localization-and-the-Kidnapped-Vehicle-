# Markov Localization and the Kidnapped Vehicle

The localization module culminates in the Kidnapped Vehicle situation. In this situation, our vehicle has been kidnapped and placed in an unknown location. We must leverage our knowledge of localization to determine where our vehicle is.

Markov Localization or Bayes Filter for Localization is a generalized filter for localization and all other localization approaches are realizations of this approach. By learning how to derive and implement this filter, we develop intuition and methods that will help us solve any vehicle localization task, including implementation of a particle filter.

We generally think of our vehicle location as a probability distribution, each time we move, our distribution becomes more diffuse (wider). We pass our variables (map data, observation data, and control data) into the filter to concentrate (narrow) this distribution, at each time step.

## Localization Posterior: Introduction

What we want to estimate is the transformation between the local coordinate system of the car and the global coordinate system of the map. If we know this transformation, then we also know poses of the car in the global map.

**We assume these variables are known:**

* A map with all landmarks in a global coordinate system, which could be a grid map of the global environment, or a database, which includes global feature points and the lane geometry. Here, we do not add the time index t to the map because we assume the map does not change over time (**m** represents the map, which could be grid maps, feature maps, landmarks).



* Observations from the on-board censor, which are defined as a vector **z**, which includes all observations from timestep one to t. The observations could be range measurements, bearing angles, or images (**z<sub>1:t</sub>** represents the observation vector from time 0 to t (range measurements, bearing, images, etc)). 


* The local coordinates system and the information how the car moves between two time steps. We also have the controls of the car as a vector **u** which includes all control elements from time step one to t. Typically, you have low pitch or roll rates and velocity information (**u<sub>1:t</sub>** represents the control vector from time 0 to t (yaw/pitch/roll rates and velocities)).



**These values are unknown:**

The position of the car at time t is defined with **x**, If we assume we have a 2D map for example, x includes a position with x and y coordinates and the orientation phi.


* x and y coordinates and also the orientation phi (**x<sub>t</sub>** represents the pose (position (x,y) + orientation θ)).

vehicle localization task

