## Overview
This package aims to be a one-stop solution for reporting all **```events```** occurring in the current CRAM framework.

* **Event**: A **Sum-of-Products** or **Product-of-Sums** of **```constraints```**, identified by its *name*. On escalation of a violation of a constraint, the SoP/PoS expression is evaluated and checked if the event is to be reported or not.
* **Constraint**: A group of rule(s) which determine the normal functioning of the system. In case of any abnormal functioning, the constraint is violated and the relevant event is checked. 

The user registers an event and then just checks the output on the desired feedback channel (ROS message, ROS service or ROS parameter).

To register an event, a message is sent with details such as its name, the constraints, the Boolean Pos/SoP expression, the event reporting mechanism, etc.
An event can be of many types. As of now, following types of events are supported:
* **Object event**: Event generated by the objects in the **Bullet World** without any information of how it is generated. It is just monitored and the relevant information is forwarded to the required ROS feedback mechanism.
* **Physics event**: In this the focus is on information generated/captured by the physics engine (such as position, velocity and acceleration, both linear and angular) to determine when the event has occurred. It consists of list of Physics Constraint.

A Physics Constraint can be of two types:
* Normal: In this a target-object and a reference is used. The data of target-object is taken wrt the reference while calculating the constraints. The data can be either 
    * Position
    * Velocity
    * Acceleration

  of the object. Also, it can be either linear or angular (in the latter case, all data is in Fixed Axis coordinate frame)

  Also, a  list of 3D points is provided along with 2 optional flags
    * is_scalar
    * is_interior
* Custom: In this mode, the only difference is that you can provide your own function to do the checking for constraint violations. This is helpful especially when you are providing constraints like (No. of people in a room), etc. The string to be provided can use the event in which it is being specified, but it has to be only the function body, nothing else.


## Reasoning for choosing **Sun-of-Products/Product-of-Sums**

The SoP/PoS expressions are heavily used in Boolean logic, especially in Hardware Description Languages to cover a wide array of logical situations encountered while implementing checks for buffer overflow, memory allocation, segmentation,  etc.

This allows complex situations like handling noise of non-uniform behavior very easy.

## Detailed analysis of is_scalar and  is_interior options
* **is_scalar** essentially allows simpler algorithms to be used instead of vector/higher dimension algorithms. Eg: If 2 points are provided with is_interior as TRUE, then if is_scalar is TRUE, then only the magnitudes of the current-value and the provided fields are checked, otherwise a number of checks are done to ensure that the current-value is on the line segment joining the provided vectors.
* **is_interior** essentially refers to not raising a concern if the current-value either falls on the same half as the origin (in case of is_scalar being FALSE for higher number of input points), or if the current-value is inside the polygon/polyhedron formed by the provided points (in case of is_scalar being TRUE)

## Effect of number of points provided on the algorithm
1. One point: is_scalar ensures that the check is for either magnitude or the vector provided. is_interior checks if the point is same as the provided. If not, and is_interior is TRUE, then a concern is raised
2. Two points: First point is assumed to be smaller than the second point. Then based on is_scalar and is_interior, checks are done to determine if an event has to be raised or nor. (See Detailed analysis above for more information)
3. 3 or more points:
    * is_scalar is TRUE: a planar case is assumed and in-polyhedron results are passed on for further processing. In this case, is_interior make full sense.
    * is_scalar is FALSE: a 3D model is considered as the basis for all further calculations and is_interior == TRUE is thought of as the half divided by the plane containing the origin

  The order of the points provided is very vital.

This architecture allows us to 
* specify the normal operating parameters of a randomly complicated system 
* check for the events without worrying about how they are handled by the upper layers of software, as well as what special condition they represent, since everything can be registered as an event uniformly.
