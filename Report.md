## Project: Search and Sample Return
### Project report and execution
---

## Project report
The goal of this project is to help navigate the canyon landscape and in that process detect the rocks tha are interest to us.


This project provided a good introduction to the three principles of a robotic system.

* Perception
* Decision
* Action

Let's look in detail these priciples, and how they apply in this project.

### Perception
In order to successfully navigate a landscape the rover needs to sense it's navigable path, obstacles, and targets it's looking for. 

The primary sensor used in this porject is the rover's camera. It feeds in images from the front of the rover. 

Since the lanndscape is limited to navigable terrain, sky, rock, and canyon walls and they have a range of RGB values, we can use thme to detect one feature at a time.

By trial and error it was observed that the min and max threshold for RGB colors for 

* navigable landscape are `((180, 255),(180, 255),(160, 255))`
* obstacle are `((0, 70),(0, 70),(0, 50))`
* rocks are `((140, 255),(110, 255),(0, 50))`

Once these indivdual feature have isolated (i.e. each image detects one thing at a time) we change the image from camera to satellite view by doing perpective transform.

However this is not enough as we are still looking it from a fixed frame of reference of the world, and we change it to rover's frame of reference.

This is achieved by rotating the frame of reference to fit the rover's orientation and translating to rover's position in the frame of reference of the world.

Each of the pixel for navigable landscape, obstacles, & rocks is turned into a polar coordinate, which gives the distance and angle from rover. The average of the angles and the distances can be used decision step for navigation control.

In the code additional member variables were added to the Rover class which would collect rock's and obstacle's distances and angles.

If you were to draw an analogy with machine learning this step is similar to feature engineering, where you would identify relevant inputs, extract necessary features and pass it on prediction algorithm for training / predicting.

### Decision
The original code scaffolding came with basic decsion for navigation, which is summaried as follows

* Go forward by controlling the throttle.
* Steer towards navigable landscape by taking an navigable angle.
* If you stop because you encountered and obstacle change the orientation and mode forward.

In the second part of the project we extended ths function additionally to pick up the rock samples when detected. The steps for this are summaried as below.

* Continue to go forward.
* When you detect the rock sample, stop, orient towards rock sample, and continue to go forward.
* If the rover is near the target, stop pick up the target and reset into stop mode.
* We also records the direction in which the rover oriented itself, so that we can re-orient the rover to maintain the oriinal heading.
* In an even where the rover loses the sight of the target, the rover will orient itself towards the navigable landscape.

#### Drawbacks
There are various drawback in this `if-elif-else` approach.
* Difficult to scale
* Difficult to forsee all the edge case or predict situation.
* Further improvements can be made by implementing either imitation learning or reinforcemnt learning to smoothly control the rover.

### Action
The decision and action principles are intertwined in this project. 

The actions can as simple as accelerating, decelarating (however it doesn't happen smoothly) / braking, orienting / steering or as complex as picking up a rock.

All the actions are preceded by the perception, where is the navigable landscape and where is rock sample etc, which direction to go, when to slow down / stop.

### Metrics
The metrics used to evaluate the project are
* Percentage of mapped terrain
* Fidelity: % of navigable pixel detected by perception vs the actual navigable pixels from reference map
* Number of rocks detected (Looks like simulator has error here, and it detects less than it should)
* We also added additional metric which is number of rocks collected.

### Notebook Analysis
The notebook (`Rover_Project_Test_Notebook`) was primarily used to put together components of perception step together and simulate the perrception from previously recorded run.

Additiional notebooks from the class were also used in order to find a suitable range of RGB thresold for navigable landscape, rocks, & obstacles. 

The threshold datastructure is a tuple `((R_min, R_max), (G_min, G_max), (B_min, B_max))`

The implementaion to the notebook were
* Applyting perspective transform
* Applying color threshold for navigable landscape, rocks, & obstables
* Changing the frame of reference from world to the rover by rotation and translation
* Update the world map


### Autonomous Navigation and Mapping

The perception step in notebook and autonomous largely remains the same with one exception that we have made changes to member variables of `Rover` class in `drive_rover.py` to collect navigable, obstacle, & rocks angles and distances.

The decision step has two implementations one provided by Udacity as is and other is experimental. How to invoke each is described in the execution section.

The experimental decision step extends the existing implementation to detect and navigate towards rock and if it's near it then pickin git up as described earlier.

One way to look at Perception + Udacity decision step is imagine Udacity model as a black box model and inner working of it are unknown, then to improve the metrics the best we can do is to tune the features in percetion step to maximize them.

In the Udacity implementation the baseline metrics of `mapped area: 40%` and `Fidelity: 60%` were satisfied, however in the experimental implementation it's a hit or miss, more skewed towards hit. 

Another problem encountered in both the implementations was that the rover gets stuck in the loop and stay there unless there is manual intervention. This can be taken care by remembering the history of actions taken by the rover and breaking this pattern by a function.

Interestingly one factor that affected metrics was is it's initial orientation which influence the starting conditions and the perception, decision making subsequently.

Also, another flaw in the implementation that was noticed is that sometimes (I haven't figured out the cause yet) while picking up the rocks the count of rocks collected increases enormously and the rover goes rogue afterwards, I'm yet to figure out the cause for this.

### Further improvements
Some of the improvemts that can be made are.
* Test cases
* Making decision step scalable.
* Remembering history to implement RL model
* Code quality improvements

### Execution

To invoke default Udacity decision step here's the command, any other use of type shall give `NotImplementedError`

```sh
python drive_rover.py --image_folder /path/to/output --type udacity
```

To invoke the experimental decision step here's the command

```sh
python drive_rover.py --image_folder /path/to/output --type exp
```




