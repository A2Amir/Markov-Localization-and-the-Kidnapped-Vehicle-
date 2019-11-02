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


<p align="right"> <img src="./img/1.jpg" style="right;" alt="vehicle localization task" width="600" height="400"> </p> 

We will never know the state **x<sub>t</sub>** with perfect accuracy. What we want is to form a sufficiently accurate belief of the state  **x<sub>t</sub>** and we want to formulate this belief in a probabilistic way. The definition of the posterior distribution for the state x at time t can be explained like below:

<p align="right"> <img src="./img/2.jpg" style="right;" alt="The definition of the posterior distribution for the state x at time t" width="500" height="200"> </p> 

## Localization Posterior: Explanation and Implementation


Localization is all about estimating the probability distribution of the state xt, which is the pose of the car, another condition that all previous observations that from time 1 to t (see above figure). Before we go deeper into math, I want to show you how we define the different input data for a specific 1D localization scenario. This means I will explain and show you how the car is sensing and moving and how the map looks.

1. Map

* The map includes the position of street lamps and trees in 1D, this means we are working with landmark-based maps, which are, in general, sparser than grid-based maps. In the 1D case, the map is a vector of the position where these objects are. Here, the map includes six landmarks with the values 9, 15, 25, 31, 59, and 77. 

<p align="right"> <img src="./img/3.jpg" style="right;" alt="the map includes six landmarks with the values 9, 15, 25, 31, 59, and 77" width="600" height="400"> </p> 


2. Observation

* For the observation, we state that the car measures the nearest k seen static objects, in driving direction. 
* We assume that the car can detect the distances to street lamps and trees, this results in an observation list which includes, for each time stamp t, a vector of distances z<sub>t</sub>, from 1 to z<sub>t</sub><sup>k</sup>.

<p align="right"> <img src="./img/4.jpg" style="right;" alt="For the observation" width="600" height="400"> </p> 

3.  Control vector

* The control vector includes a direct move of the car between consecutive time stamps. This means the control is defined by the distance the car traveled between t and t minus 1. In this case, the car moves 2 meters to the right. 

<p align="right"> <img src="./img/4.jpg" style="right;" alt="Control vector" width="600" height="400"> </p> 


**The true pose of the car is somewhere on the mapped area. Since the map is discrete, the pose of the car could be any integer between 0 and 99 meters. This means the belief of x<sub>t</sub> is defined as a vector of hundred elements, and each element represents a probability that the car is located at the corresponding position. The goal is now to estimate these values.**

<p align="right"> <img src="./img/5.jpg" style="right;" alt="The true pose of the car" width="600" height="400"> </p> 



## Bayes' Filter For Localization

We can apply Bayes' Rule to vehicle localization by passing variables through Bayes' Rule for each time step, as our vehicle moves. This is known as a Bayes' Filter for Localization. The generalized form Bayes' Filter for Localization is shown below.

<p align="right"> <img src="./img/6.jpg" style="right;" alt="Bayes' Filter For Localization" width="300" height="200"> </p> 

With respect to localization, these terms are:


1.	P(location∣observation): This is P(a|b), the normalized probability of a position given an observation (posterior). 
2.	P(observation∣location): This is P(b|a), the probability of an observation given a position (likelihood)
3.	P(location): This is P(a), the prior probability of a position 
4.	P(observation): This is P(b), the total probability of an observation

Without going into detail yet, be aware that P(location) is determined by the motion model. The probability returned by the motion model is the product of the transition model probability (the probability of moving from x<sub>t−1</sub> --> x<sub>t</sub> and the probability of the state x<sub>t−1</sub>.


In the next sections, our focus will be on:
1.	Compute Bayes’ rule
2.	Calculate Bayes' posterior for localization
3.	Initialize a prior belief state
4.	Create a function to initialize a prior belief state given landmarks and assumptions


## Calculate Localization Posterior

To continue developing our intuition for this filter and prepare for later coding exercises, some examples for determining posterior probabilities at several pseudo positions x, for a single time step is [Here](https://github.com/A2Amir/Markov-Localization-and-the-Kidnapped-Vehicle-/blob/master/Markov%20Localization%20.ipynb) prepared. You can go through to get better Intuition about Calculation of Localization

<p align="right"> <img src="./img/7.jpg" style="right;" alt="alculate Localization Posterior" width="600" height="400"> </p> 


## Initialize Belief State

To help develop an intuition for this filter and prepare for later coding exercises, let's walk through the process of initializing our prior belief state. That is, what values should our initial belief state take for each possible position? Let's say we have a 1D map extending from 0 to 25 meters. We have landmarks at x = 5.0, 10.0, and 20.0 meters, with position standard deviation of 1.0 meter. If we know that our car's initial position is at one of these three landmarks, how should we define our initial belief state?

Since we know that we are parked next to a landmark, we can set our probability of being next to a landmark as 1.0. Accounting for a position precision of +/- 1.0 meters, this places our car at an initial position in the range [4, 6] (5 +/- 1), [9, 11] (10 +/- 1), or [19, 21] (20 +/- 1). All other positions, not within 1.0 meter of a landmark, are initialized to 0.
We normalize these values to a total probability of 1.0 by dividing by the total number of positions that are potentially occupied. In this case, that is 9 positions, 3 for each landmark (the landmark position and one position on either side). This gives us a value of 1.11E-01 for positions +/- 1 from our landmarks (1.0/9). So, our initial belief state is:

```python
from decimal import Decimal
print('%.2E' %Decimal((1.0)/9 ))
```
'1.11E-01'

{0, 0, 0, 1.11E-01, 1.11E-01, 1.11E-01, 0, 0, 1.11E-01, 1.11E-01, 1.11E-01, 0, 0, 0, 0, 0, 0, 0, 1.11E-01, 1.11E-01, 1.11E-01, 0, 0, 0, 0}

You can find [Here](https://github.com/A2Amir/Markov-Localization-and-the-Kidnapped-Vehicle-/blob/master/Markov%20Localization%20.ipynb) other example to get better Intuition.In the next concept, we will implement belief state initialization in C++.


## Initialize Priors Function:

[Here](https://github.com/A2Amir/Markov-Localization-and-the-Kidnapped-Vehicle-/blob/master/InitializePriorsFunction.cpp) is created a function in C++ that initializes priors based on the above explained agreement (initial belief state for each position on the map) given landmark positions, a position standard deviation (+/- 1.0), and the assumption that our car is parked next to a landmark.Note that we input a control of moving 1 step but our actual movement could be in the range of 1 +/- control standard deviation. The position standard deviation is the spread in our actual position.

The result of the code:
[0, 0, 0 ,0, 0.111111, 0.111111, 0.111111 ,0, 0, 0.111111, 0.111111, 0.111111 ,0 ,0 ,0 ,0 ,0 ,0, 0, 0.111111, 0.111111, 0.111111, 0 ,0, 0]


## How Much Data: Explanation

Before we go back to math, I want to make sure you understand how much data Z<sub>1:t</sub> represents, so you can consider what the performance consequences would be. So let's pretend that

* The car has driven for 6 hours (6*60(min)*60(second))
* LIDAR refreshes 10 times per seconds (10 Hertz)
* LIDAR sends 100,000 data points per observation
* Each of the 100,000 observations contains 5 pieces of data
* Each piece of data requires 4 bytes


How much data is contained in this Z<sub>1:t</sub>?  Multiplying out gives **432 GB** of data!

you have a sense of the quantity of the data that a real car would use if it's updates step took into account all historical observations. Later in next section, I will show you how we can get around this limitation.

## Derivation schema

Up to here, there are two problems if we want to estimate the posterior directly. 

* The first one is the localizer must process on each cycle a lot of data. 
* The second is the amount of data increases over time. 
This won't work for real time localizer, In the following, I will present a mathematical proof showing that we can change this so our localizer, 
* only needs to handle a few bytes on each update (A little data bytes per update) 
* handles the same amount of data per update regardless of drive time(amount of data remains constant). 
let's start with an overview of what we want to achieve. 

##  Apply Bayes Rule with Additional Conditions

You already learned the observation vector could be a lot of data, and we do not want to carry the whole observation history to estimate the state beliefs. We aim to estimate state beliefs **bel(x<sub>t</sub>)** without the need to carry our entire observation history. We will accomplish this by manipulating **our posterior (x<sub>t</sub>∣z<sub>1:t−1</sub>,μ<sub>1:t</sub>,m)** obtaining **a recursive state estimator**. For this to work, we must demonstrate that **our current belief bel(x<sub>t</sub>)** can be expressed by the belief **one step earlier bel(x<sub>t−1</sub>)** then use **new data** to update only **the current belief**. This recursive filter is known as the Bayes Localization filter or Markov Localization and enables us to avoid carrying historical observation and motion data. 


