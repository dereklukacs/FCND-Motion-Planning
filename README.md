# Udacity Flying Car Nano Degree Quadrotor City Path Planning #
#### Derek Lukacs ####
This repository is a branch and add on to a project from the Udacity Flying Car Nano Degree (FCND) program.

## Overview of Code ##
The starter code is broken into two main pieces:

- `planning_utils.py` with helpful functions
- `motion_planning.py` that inherits from Drone

`planning_utils` defines several helpful functions including:
* `create_grid`: returns a grid representation of the 2D configuration space
* `Action`: an enum which defines actions and there associated cost
* `Valid_actions`: which defines which actions can take place given the current node and grid
* `a_star`: an implementation of the a-star search algorithm for a 2 dimensional problem.
* `heuristic`: this defines the a star algorithm search heuristic, in this case we are using the euclidean distance to the goal.

`motion planning.py` is a class to work with the FCND simulator for this project. It is broken down into the following key sections

* Callback functions: These handle what should be done when state variables are changed. Most actions are based on the current state.
* Transition functions: These functions handle the handoff from one state to the next within the finite state machine.
* Helper functions: these functions serve to facilitate the other activities of the callback, transition, and start functions.
* Path Planning: The planpath() function is given with several TODO’s which are the main goal of this project.

## Background ##
Path planning for this project is made up of three distinct steps. 
1. Build a map
2. Plan a path
3. Simplify path (prune waypoints)

The map for this project is given in `colliders.csv` which contains representations of the buildings inside of the "arena" that the quad will fly in.

The path is planned using an A* algorithm in the generated map given a endpoint and a start point. 

The path that the A* algorithm generates is pruned using a combination of collinearity and ray tracing algorithms. 

This path is then sent to a simulator guides the quadrotor along the path.

The simulator for this project can be downloaded from 

## Building the Map ##
The map is read in from `colliders.csv` which has a listing of all of the collidable objects and their dimensions and locations. This is imported and converted to a 2D grid based on the desired flight altitude. 

## Planning an Optimal Path ##
For this project a simple 2D A* search was used after constructing the map. The general algorithm goes as follows (for 2D).

1. Define a heuristic for each point based on its location relative to the GOAL	
	* Manhattan distance. |x1 − x2| + |y1 − y2|, or
	* Euclidean distance. 􏰂(x1 − x2)<sup>2</sup> + (y1 − y2)<sup>2</sup>
2. Define costs of moving between each possible waypoint (simple for a grid)
3. Calculate cost of moving to all possible adjacent locations from current location
4. Pick best choice based on the cost of getting to that location PLUS the heuristic cost to the finish 
5. Repeat until reached goal

For the first step, the euclidean distance heuristic was used. 

Valid actions and their costs are defined in `valid_actions()`

The rest of the steps were already implemented in the standard implementation of A* .

## Pruning Excess Waypoints ##
Having planned a path with A* in a grid based map there will be many redundant points. Unless a feedforward velocity is used it will take a lot longer to follow these points. The less points needed to describe the path, the better. In order to reduce the number of points, pruning methods were used. Typically paths are pruned using either collineraity or ray tracing algorithms. 

Due to an insufficiency found in the collinearity algorithm, I implemented both Ray Tracing and Collinearity.
The algorithm can be stated qualitatively as follows, if for 3 points:
1. All are collinear down to a ε value of error then remove the center point
2. The second is in one of the cells laid out by the Bresenham algorithm between points 1 and 3, remove the center point
3. No cells in the Bresenham ray tracing algorithm from 1 to 3 have an obstacle in them, remove center point.

To demonstrate the importance of path pruning, the following simulation does not use any path pruning. The quad actually crashes because it is not stable enough. With all of those points.
<img src="https://github.com/dereklukacs/FCND-Motion-Planning/blob/master/images/no_pruning.gif?raw=true"
     alt="Simulation with no path pruning being used, quad crashes"/>

Now, using path pruning the path is simple for the quadrotor to follow and it successfully makes it to the finish. 
<img src="https://github.com/dereklukacs/FCND-Motion-Planning/blob/master/images/with_pruning.gif?raw=true"
     alt="Simulation with path pruning, quad completes goal" />

## Conclusion and Future Expansion ##
A quadrotor is able to be simulated and find its way to any valid location on the map using an A* search algorithm. The path is then succesfully simplified with path pruning methods to reduce excess waypoints. 


#### Graph Based Search ####
An expansion to this program could be to use graph based search which can be performed quicker once a graph has been constructed.

In order to construct a graph I would start with my 2D grid generated from the 2.5D map given in colliders.csv.
Using medial axis and voronoi algorithms this could be implemented.

In order to implement this a new definition for costs will need to be made. This can be done by defining a general euclidean cost function based on the total distance traveled.

