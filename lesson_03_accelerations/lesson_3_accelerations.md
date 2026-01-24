# Learn the basics of Control Engineering with StaRS 2D
## Lesson 3: A more realistic Starship model

<br>
<p align="center">
Welcome to the third lesson of "Learn the basics of Control
Engineering with StaRS 2D"!</p>
<br>

<p align="center">
  <img src="https://github.com/gsmfnc/StaRS2D_course/blob/main/lesson_02_proportional_control/imgs/animation.gif" />
</p>

<br>

---

In the
[previous lesson](https://github.com/gsmfnc/StaRS2D_course/blob/main/lesson_02_proportional_control/lesson_2_proportional_control.md),
we introduced PI controllers and discussed their use to succeed the mission.
In this lesson, we continue to rely on PI controllers but we consider a more
realistic dynamic model of Starship.
Moreover, we will introduce a multi-loop control strategy to ensure a safe
landing.

---

Outline of the lesson:
1.  [A more realistic Starship model](#a-more-realistic-starship-model)
2.  [Multi-loop control](#multi-loop-control)
3.  [Exercises](#exercises)

## A more realistic Starship model

Unlike the first two lessons, Starship’s dynamics are modeled here in a more
realistic manner.
It is assumed that the center of mass coincides with the center of gravity.
Under this assumption, letting $T$ denote the thrust command, the translational
force equations are given by

```math
F_x=T\sin(\theta)-0.1v_x^2\cos(\theta),\ F_y=T\cos(\theta)-g
```

The term proportional to the square of the horizontal velocity $v_x^2$ in $F_x$
represents the aerodynamic drag force due to wind, which opposes the motion
along the horizontal direction.

Letting <em>L</em> be the length of Starship ("longer" side),
the rotational dynamics are described by:

```math
\tau=T\frac{L}{2}\sin(\theta)
```

The linear accelerations can be computed by multiplying the forces
by the inverse of the Starship's mass, while the angular acceleration is
obtained by multiplying the torque by the inverse of the moment of inertia.
Then, simple integration of these acceleration equations using a sampling time
of $T_s=0.1$ seconds produces the motion dynamics.

## Multi-loop control
### Vertical control

We begin with vertical control.
A multi-loop control architecture means that we define our controller in
multiple stages.

The inner (or first) loop determines the thrust command to track a desired
vertical speed $V_{y,des}$.
We will do so by implementing a simple proportional controller with gain
thrustPGain.
The code will look like:

```
cmd.setThrustCommand(0.5 - thrustPGain * (env.getStarshipVy() - VyDes));
```

The outer (or second) loop determines the desired vertical speed $V_{y,des}$.
This is done by first definining the desired vertical position, which is set to
$y=0$, and then computing the position error with respect to the current height,
i.e.

```
VyDes = max(-VyMax, min(VyMax, VyGain * (0 - env.getStarshipYPosition())));
```

These two nested loops compose the vertical controller of this lesson.
This controller is kept active during the whole simulation.

### Horizontal control

Horizontal control is divided into three phases: approach, slow down and
landing.

#### Approach

This phase starts at the beginning of the simulation and ends when
Starship's horizontal position is within 200 pixels of the destination.
During this phase, we keep a constant horizontal speed to move toward the
landing tower.

The horizontal control is a multi-loop controller composed of three nested
loops.
The inner loop is proportional angular-rate controller with gain
thrustAnglePGain and computes the thrust angle command to track a desired
angular speed $\omega_{des}$, i.e. considering the error between the current
angular speed of Starship and the desired one:

```
cmd.setThrustAngleCommand(thrustAnglePGain * 
    (env.getStarshipOmegaInDegrees() - omegaDes));
```

The desired angular speed is defined in the middle loop.
This loop is a PI controller that determines $\omega_{des}$ considering a
desired Starship angle $\theta_{des}$.
The error will be computed as the difference between the desired angle
and the current Starship's angle:

```
omegaDes = omegaPGain * (thrustAngleDes - env.getStarshipAngleInDegrees()) +
    omegaIGain * thrustAngleIError;
```

Here, omegaPGain and omegaIGain are the proportional and the integral gain,
respectively.
The variable thrustAngleIError represents the integral of the Starship's angle
error.

The outer loop computes the desired thrust angle based on the horizontal
velocity error with respect to a desired horizontal speed of $3pix/sec$:

```
thrustAngleDes = 3.0 - env.getStarshipVx();
thrustAngleIError = thrustAngleIError + thrustAngleDes;
```

#### Slow down

Once Starship's horizontal position is within 200 pixels of the destination,
we implement a PI controller to get Starship closer to the target horizontal
position, specifically toward $x=20$ pixels.

#### Landing

We keep slowing down until the vertical position is sufficiently close to the
target vertical position, namely when $|y|<1pix$.
At this point, the landing phase begins.
During landing, the vertical position is maintained within the interval
$[-1,1]$, while the commanded horizontal speed is reduced proportionally to the
distance from the landing tower to a small residual value of $0.05pix/sec$.

## Exercises
