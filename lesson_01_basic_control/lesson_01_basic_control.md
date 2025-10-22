# Learn the basics of Control Engineering with StaRS 2D
## Lesson 1: A basic flight controller

<br>
<p align="center">
Welcome to the first lesson of "Learn the basics of Control
Engineering with StaRS 2D"!</p>
<br>

<p align="center">
  <img src="https://github.com/gsmfnc/StaRS2D_course/blob/main/lesson_01_basic_control/imgs/animation.gif" />
</p>

<br>

Control engineering is becoming more and more important nowadays: just to
give you an idea, the design of
modern aircrafts, spaceships and autonomous cars widely relies on it.

But... what is control engineering? Well, this course aims at giving you
insights to answer this question by considering a current fascinating control
engineering application: the <em>Starship re-entry task</em>.

In this course, you will:
-   Learn how to design simple controllers using standard control
engineering techniques;
-   Use an educational 2D Starship Re-entry Simulator
([StaRS 2D](https://github.com/gsmfnc/StaRS2D)) in
[Processing](https://processing.org/) to see your controller in action!

---

Pre-requisites:
1.  Some experience with coding
2.  Understanding of basic physics, mathematics and trigonometry

---

Outline of the lesson:
1.  [Installation](#installation)
2.  [Main script](#main-script)
3.  [Thrust vectoring](#thrust-vectoring)
4.  [Instrumentation](#instrumentation)
5.  [Forces and torques simplification](#forces-and-torques-simplification)
6.  [Rules](#rules)
7.  [Your first flight controller design](#your-first-flight-controller-design)
8.  [Exercises](#exercise)

## Installation

To begin with, we need to install Processing.
You can follow the instructions of this [link](https://processing.org/download)
to download the latest Processing software version.
However, the code was tested with older versions (4.3 and 4.4) that can be
downloaded
[here](https://github.com/processing/processing4/releases/tag/processing-1293-4.3)
and
[here](https://github.com/processing/processing4/releases/tag/processing-1304-4.4.4).

To install StaRS 2D, just clone this
[github repository](https://github.com/gsmfnc/StaRS2D) and open
<em>stars2d.pde</em> with Processing (File -> Open... and select the
file wherever you cloned the repository).
Once you open <em>stars2d.pde</em>, the Processing window will look like this:

<p align="center">
  <img src="https://github.com/gsmfnc/StaRS2D_course/blob/main/lesson_01_basic_control/imgs/processing_screen.png" />
</p>

This code already includes an implementation of a working flight controller, so
if you press run on the top left, you should see the very first animation above!

## Main script

<em>stars2d</em> is the main script.
In order to not
compromise the functionality of the simulator, do not modify the other files
you see (i.e., <em>environment</em>, <em>starship</em> and <em>utils</em>).

The very first lines of code declare two important variables: 'env' and 'cmd'.
These must always be declared.
Moreover, three other variables are defined: 'phase', 'velDes' and
'thrustAngleDes'.
These are only auxiliary variables to make the pre-designed flight controller
work.

<p align="center">
  <img src="https://github.com/gsmfnc/StaRS2D_course/blob/main/lesson_01_basic_control/imgs/processing_screen_highlight_1.png" />
</p>

In Processing, the setup() function is executed only once right after the "Run"
button is pressed.
In this case, it creates a window of size 1200x600 pixels and then it
initializes 'env' and 'cmd'.

<p align="center">
  <img src="https://github.com/gsmfnc/StaRS2D_course/blob/main/lesson_01_basic_control/imgs/processing_screen_highlight_2.png" />
</p>

The draw() function is executed in loop right as soon as setup() terminates.
For every loop, it calls
env.initialize() that creates the graphics shown below.

<p align="center">
  <img src="https://github.com/gsmfnc/StaRS2D_course/blob/main/lesson_01_basic_control/imgs/processing_screen_highlight_3.png" />
</p>
<p align="center">
  <img src="https://github.com/gsmfnc/StaRS2D_course/blob/main/lesson_01_basic_control/imgs/graphics_highlight.png" />
</p>

Here, we can see:
1.  The initial position of Starship at the top right;
2.  The instrumentation showing several parameters of Starship at the bottom
right;
3.  The "re-entry" tower at the bottom left;
4.  The red circle beside the reentry tower representing the "zero" coordinate
and the destination point.

First thing we will do is to delete the current flight controller.
To do that, we have to remove the initialization of the auxiliary variables
('phase', 'velDes' and 'thrustAngleDes') and only keep env.initialize() in
draw().
The following picture shows how the script should now look like.

<p align="center">
  <img src="https://github.com/gsmfnc/StaRS2D_course/blob/main/lesson_01_basic_control/imgs/blank_processing_script.png" />
</p>

## Thrust vectoring

To steer Starship towards the re-entry tower, we must provide thrust
vectoring inputs.
To do so, StaRS 2D provides the following functions:

| Function | Description |
| :---------------- | :-------------------- |
| cmd.setThrustCommand(val) | Determines the thrust. 'val' must be between 0 and 1: negative values will be forced to 0, whereas values greater than 1 will be forced to 1. When the angle $\theta$ of Starship equals 0, a thrust command of 0.5 perfectly compensates gravity.|
| cmd.setThrustAngleCommand(val) | Determines the angle of thrust. It is limited to [-30,30] degrees, thus values outside this interval will be forced to either -30 (if val<-30) or 30 degrees (if val>30).|

Varying 'val' in cmd.setThrustCommand(val) gradually from 0 to 1, you will see
the animation of Starship changing like this:

<p align="center">
  <img src="https://github.com/gsmfnc/StaRS2D_course/blob/main/lesson_01_basic_control/imgs/thrust.gif" />
</p>

Similarly, varying 'val' in cmd.setThrustAngleCommand(val) gradually from -30 to
30, you will see:

<p align="center">
  <img src="https://github.com/gsmfnc/StaRS2D_course/blob/main/lesson_01_basic_control/imgs/thrust_angle.gif" />
</p>

## Instrumentation

The position and the attitude of Starship is expressed with respect to a
coordinate system that has its origin in the "landing point" of the re-entry 
tower (see figure below).
Starship's position is indicated with a pair $(x,y)$ of decimal values
representing the pixel [pix] that the center of Starship occupies.
The velocity along the axes $x$ and $y$ is denoted with $Vx$ and $Vy$.
It is expressed in pixel per second [pix/sec].
The attitude is represented by the angle $\theta$ and its associated angular
velocity is denoted with $\omega$.
These are expressed in degrees [deg] and degree per second [deg/sec].
Finally, the instrumentation shows the percentage of thrust, the thrust angle
and the elapsed time.

<p align="center">
  <img src="https://github.com/gsmfnc/StaRS2D_course/blob/main/lesson_01_basic_control/imgs/position_attitude.png" />
</p>

StaRS 2D provides the following functions to programmatically access the
information of the instrumentation:

| Function | Description |
| :---------------- | :-------------------- |
| env.getStarshipXPosition() | Returns x-coordinate of Starship's position|
| env.getStarshipYPosition() | Returns y-coordinate of Starship's position |
| env.getStarshipVx() | Returns x-coordinate of Starship's velocity |
| env.getStarshipVy() | Returns y-coordinate of Starship's velocity |
| env.getStarshipAngle() | Returns $\theta$ angle value (in radians) |
| env.getStarshipOmega() | Returns angular velocity $\omega$ value (in radians) |
| env.getStarshipAngleInDegrees() | Returns $\theta$ angle value (in degrees) |
| env.getStarshipOmegaInDegrees() | Returns angular velocity $\omega$ value (in degrees) |
| env.getElapsedTime() | Returns the elapsed time |

## Forces and torques simplification

In this lesson, applied forces and torques are simplified to
applied velocities and angular
velocities to make the design of controllers simpler.
So, gravity is a constant velocity pushing downwards.
When Starship is "upright", a thrust command of $0.5$ with
a zero thrust angle perfectly "defeats" gravity.
To simplify the simulation model, you must provide '1' as an argument in the
initialization of env, as shown below.

<p align="center">
  <img src="https://github.com/gsmfnc/StaRS2D_course/blob/main/lesson_01_basic_control/imgs/processing_screen_simplified_on_highlighted.png" />
</p>

## Rules

You fail the re-entry mission if:
1.  You crash to the ground;
2.  You hit the launch tower;
3.  You land too quickly (both velocity and angular velocity must be between
-0.1 and 0.1 at the moment you reach the landing point).

You succeed the descent and the landing if:
1.  You reach any point in the square defined by $x$ and $y$ both in $[-1,1]$
pixels while having $\theta$ in $[-0.5,0.5]$ degrees, $\omega$ in
$[-0.1,0.1]$ degree/second and both $Vx$ and
$Vy$ in $[-0.1,0.1]$ pixel/second.

# Your first flight controller design

Since we have a simulator at out disposal, we will design our first flight
controller by slowly building up pieces and seeing how the simulation goes.

To begin with, we will set a thrust command of $0.5$ (i.e.
we add `cmd.setThrustCommand(0.5)` in the draw() function), that we know being
sufficient to "defeat"
gravity when Starship is perfectly perpendicular to the ground.
Then, since we need to move towards the re-entry tower, we need to push
Starship to its left.
To do so, we can give a thrust angle command of, e.g., $-2$ degrees, making
Starship rotate counter clockwise.
The overall script will look like as in the following image.
Note that in order to actually give the thrust commands, we need to call
`env.updateStarship(cmd)` at the end of the draw() function.

<p align="center">
  <img src="https://github.com/gsmfnc/StaRS2D_course/blob/main/lesson_01_basic_control/imgs/coding_01.png" />
</p>

If we hit the Run button now, we will see Starship starting to rotate, slowly
losing altitude and hit the ground.
Obviously, we need to make Starship stop rotating at some point.
For example, we can make it stop around $30$ degrees.
One possible way to proceed is to use an if statement: if Starship's angle is
smaller than $30$ degrees, then give a thrust angle command of $-2$ degrees;
otherwise, set thrust angle command to zero.
Note that we can measure the attitude of Starship using some built-in functions
of the simulator.
In this case, we need `env.getStarshipAngleInDegrees()`.
The next image shows how the code turns out.

<p align="center">
  <img src="https://github.com/gsmfnc/StaRS2D_course/blob/main/lesson_01_basic_control/imgs/coding_02.png" />
</p>

By pressing the Run button, Starship rotates until it gets to $30$ degrees and
then it starts translating to its left.
Note that since the thrust command is still at $0.5$ but Starship is not
perpendicular with the ground, we will also lose a little bit of altitude.
However, we are ok with that as we are higher than the re-entry tower.

If we keep giving these commands, Starship will not reach the destination point.
As a matter of fact, it will just fly past it.
First, we notice that when we are close to the tower, Starship is too high in
the sky.
So, what we could do is reducing the thrust command when we are close to the
re-entry tower.
This will make gravity "win" and get Starship to lose altitude.
Let's do so with another if statement: if Starship's x position is sufficiently
close to zero (i.e. to the destination point), set the thrust command to $0.25$.
We can measure the x position of Starship with `env.getStarshipXPosition()` and
we will decrease the thrust command when we are below $100$ pixels to the
destination point.
Also, we want to slow down a little as we are closer to landing and we require
carefulness.
So, we will do the following: if the x position of Starship is below $100$
pixels,
then if Starship's angle is above 10 degrees, set thrust angle command to $2$
degrees; otherwise, set it to zero.
This way, Starship's x velocity $Vx$ will be smaller as $\theta$ will be
smaller.
The overall logic has been implemented with two nested if statements, as shown
below.

<p align="center">
  <img src="https://github.com/gsmfnc/StaRS2D_course/blob/main/lesson_01_basic_control/imgs/coding_03.png" />
</p>

Hitting the Run button, we will see Starship approaching the re-entry tower,
slowing down and losing altitude until it hits the ground... whoops!
We need to bring back the thrust command to a value that keeps the altitude
constant when we reach our desired value.
The y-coordinate of the destination point is zero so we will add an if statement
to give a thrust command of $0.5$ when the y position of Starship is around
zero.
We will use `env.getStarshipYPosition()` as depicted in the next image.

<p align="center">
  <img src="https://github.com/gsmfnc/StaRS2D_course/blob/main/lesson_01_basic_control/imgs/coding_04.png" />
</p>

This still does not succeed the landing!
We need to further slow down and keep Starship more straight when we are close
to the destination point.
To succeed, we need $\theta$ to be smaller than $0.5$ degrees and all the
velocities below $0.1$ pixel/second.
So, as Starship's x position is below 10 pixels, we will set a thrust angle
command of $0.2$ degrees until $\theta$ becomes smaller than $0.5$ degrees.
We will notice that this brings all the velocities inside the limits to succeed
the landing.

<p align="center">
  <img src="https://github.com/gsmfnc/StaRS2D_course/blob/main/lesson_01_basic_control/imgs/coding_05.png" />
</p>

Running the script we find out that this is still not sufficient!
We hit the tower because Starship does not manage to keep the altitude to $0$.
The thrust command of $0.5$ that we gave to stop losing altitude is not
sufficient as Starship is not completely upright.
Let's substitute it with $0.51$ to have a slight push up.

<p align="center">
  <img src="https://github.com/gsmfnc/StaRS2D_course/blob/main/lesson_01_basic_control/imgs/coding_06.png" />
</p>

We did it!
Congratulations on your first flight controller design for the re-entry task!

# Exercise

1.  Can you make Starship land in less time? Building on the controller of the
previous section, change some of the parameters to reduce the landing time.
