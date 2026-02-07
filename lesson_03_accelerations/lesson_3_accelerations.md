# Learn the basics of Control Engineering with StaRS 2D
## Lesson 3: A More Realistic Starship Model

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
we introduced PI controllers and discussed their use to successfully
complete the mission.
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

Letting <em>L</em> be the length of Starship (the "longer" side),
the rotational dynamics are described by

```math
\tau=T\frac{L}{2}\sin(\theta)
```

The linear accelerations can be computed by multiplying the forces
by the inverse of Starship's mass, while the angular acceleration is
obtained by multiplying the torque by the inverse of the moment of inertia.
Then, simple integration of these acceleration equations using a sampling time
of $T_s=0.1$ seconds produces the motion dynamics.

You do not really need to understand the meaning of this sentence: it is
sufficient to understand that every execution of the Processing's draw()
function computes the future position of Starship $0.1$ seconds from the current
time, assuming that the commands you give are applied for $0.1$ seconds.

## Multi-loop control
### Vertical control

We begin with vertical control.
A multi-loop control architecture means that we define our controller in
multiple stages.
Each loop regulates a different variable.
Typically, the outer and middle loops generate reference values (such as desired
positions or velocities), while the inner loop is responsible for tracking those
references by directly commanding the actuators.

The outer loop determines the desired vertical speed ```vyDes```.
This is done by first definining the desired vertical position, which is set to
$y=0$, and then computing the position error with respect to the current height.
The value of ```vyDes``` will be proportional to this error, with gain
```vyGain```.
To avoid excessively large commands, we limit its value to ```vyMax```.
Therefore,

```
vyDes = max(-vyMax, min(vyMax, vyGain * (0 - env.getStarshipYPosition())));
```

The inner loop determines the thrust command to track the desired
vertical speed.
We do so by implementing a simple proportional controller with gain
```thrustPGain```.
The code will look like:

```
cmd.setThrustCommand(0.5 - thrustPGain * (env.getStarshipVy() - vyDes));
```

Note that we also provide a constant command of 0.5 that we know it is
sufficient to counteract gravity when Starship is upright.
Therefore, the proportional controller only adds or subtracts from
this constant value.
These two nested loops together compose the vertical controller used in this
lesson.

### Horizontal control

Horizontal control is divided into three phases: approach, slow down and
landing.

#### Approach

This phase starts at the beginning of the simulation and ends when
Starship's horizontal position is within 200 pixels of the destination.
During this phase, we keep a constant horizontal speed to move toward the
landing tower.

During this phase, the horizontal control is a multi-loop controller composed of
three nested loops.

The outer loop computes the desired Starship angle using a PI controller acting
on the horizontal speed error.
We require a desired horizontal speed of $3$ pix/sec, thus the horizontal
speed error ```vxError``` and its integral ```vxIError``` can be computed as

```
vxError = 3.0 - env.getStarshipVx();
vxIError = vxIError + vxError * env.getSamplingTime();
```

Note that differently from the previous lesson, the integral error is now
computed as the sum of ```vxError``` multiplied by the sampling time.
This is due to the fact that we are now using a dynamic model of Starship and
every execution of Processing's draw() computes the future position of Starship
$0.1$ seconds from the current time.

To obtain the desired Starship angle, we multiply these errors
by the two PI gains, ```anglePGain``` and ```angleIGain```, representing the
proportional and the integral gains, respectively:

```
angleDes = anglePGain * vxError + angleIGain * vxIError;
```

The middle loop computes the desired Starship angular rate using a proportional
controller acting on the Starship angle error, thus

```
omegaDes = omegaPGain * (angleDes - env.getStarshipAngleInDegrees());
```

where ```omegaPGain``` is the proportional gain.

Finally, the inner loop computes the thrust angle command using a proportional
controller acting on the angular rate error.
We will define its proportional gain as ```thrustAnglePGain```.
Hence,

```
cmd.setThrustAngleCommand(thrustAnglePGain * (env.getStarshipOmegaInDegrees() - omegaDes));
```

#### Slow down

Once Starship's horizontal position is within 200 pixels of the destination,
we implement a PI controller to move Starship closer to the target horizontal
position, specifically toward $x=20$ pixels.

During this phase, the horizontal controller is implemented as a multi-loop
architecture with four nested loops.

The outer loop computes the desired horizontal velocity ```vxDes``` with a
proportional
controller that considers the error in the distance from the target horizontal
position $x=20$ pixels.

```
vxDes = vxPGain * (env.getStarshipXPosition() - 20.0);
```

The first middle loop computes the desired Starship angle using a PI controller
acting on the horizontal velocity error.
Therefore, just like the horizontal controller of the approach phase, we will
need to compute the error ```vxError``` and its integral ```vxIError``` and then
define ```angleDes```:

```
vxError  = vxDes - env.getStarshipVx();
vxIError = vxIError + vxError * env.getSamplingTime();

angleDes = anglePGain * vxError + angleIGain * vxIError;
```

The second middle loop computes the desired angular rate ```omegaDes``` using a
proportional controller acting on the Starship angle error:

```
omegaDes = omegaPGain * (angleDes - env.getStarshipAngleInDegrees());
```

Finally, the inner loop computes the thrust angle command using a proportional
controller acting on the angular rate error:

```
cmd.setThrustAngleCommand(thrustAnglePGain * (env.getStarshipOmegaInDegrees() - omegaDes));
```

Notice that this horizontal controller is very similar to the one of the
approach phase.
The only difference relies on the definition of the desired horizontal speed
```vxDes```.
Here, ```vxDes``` is defined through an outer loop whereas it was fixed to
$3$ pix/sec during the approach phase.

#### Landing

We keep slowing down until the vertical position is sufficiently close to the
target vertical position, namely when $|y|<1$ pix.
At this point, the landing phase begins.

During landing, the vertical position is maintained within the interval
$[-1,1]$, while the commanded horizontal speed is reduced proportionally to the
distance from the landing tower to a small residual value of $0.05$ pix/sec.

During this phase, the controller is again composed of four nested loops.
The structure of the controller is identical to that of the slow down phase,
with the only difference being the computation of the desired horizontal speed.

During landing, the desired horizonal velocity ```vxDes``` is defined so that it
is:
-   always positive;
-   bounded by a maximum value, chosen as $1$;
-   proportional to the distance from the point $x=10$ pixels (with a
proportional gain of $0.1$).
When Starship reaches $x<10$, the desired horizontal speed is no longer reduced
proportionally but fixed to a constant value of $0.05$ pix/sec, ensuring a
successful landing.
All these requirements can be achieved by defining ```vxDes``` as:

```
vxDes = max(0.0, min(1.0, vxPGain * (env.getStarshipXPosition() - 10.0))) + 0.05;
```

#### Handling phases switches and tuning

We now need to define the logic to switch between the mission phases.

The controller starts in the approach phase, which is represented by setting
the variable `phase=1`.

The transition from phase 1 (approach) to phase 2 (slow down) occurs when
Starship is $200$ pixels away from the landing point along the horizontal
direction.
Note that this transition needs to be performed only if the current phase is
equal to $1$ to avoid switching back to the slow down phase from the final
phase (landing).

```
if (abs(env.getStarshipXPosition() - env.getDestinationX()) < 200 &&
        phase == 1) {
    phase = 2;
}
```

Finally, the controller switches to phase $3$ (landing) when Starship's vertical
position is within $1$ pixel of the landing point's vertical coordinate.
Again, to avoid a direct switch to phase $3$ from phase $1$, the transition must
occur only if Starship is currently in the slow down phase (phase $2$).

```
if (abs(env.getStarshipYPosition() - env.getDestinationY()) < 1 && phase == 2) {
    phase = 3;
}
```

Throughout the lesson, we have introduced several gains for our P and PI
controllers.
Now, we have to tune them in order to obtain satisfactory performance.

For the vertical controller, we need to tune three parameters.
We start with ```vyMax```, i.e. the maximum value for the vertical speed, that
is set to $1$ pix/sec.
Then, we tune the proportional gain for the outer loop ```vyGain```, which is
chosen to be equal to
$0.1$, and the proportional gain of the inner loop ```thrustPGain```,
which is set to $1$.

```
float vyGain = 0.1;
float vyMax = 1.0;
float thrustPGain = 1.0;
```

Regarding the horizontal controller, all phases require tuning the
gains for:
1.  A PI controller for the desired Starship angle, with proportional gain
`anglePGain = 1` and integral gain `angleIGain = 0.5`;
2.  A P controller for the desired angular rate with gain
`omegaPGain = 1`;
3.  A P controller for the thrust angle command with gain
`thrustAnglePGain = 0.25`.

The slow down and landing phases also require an additional gain ```vxPGain```
for the computation of the desired horizontal velocity.
This gain will be set to:
1.  $0.016$ during the slow down phase;
2.  $0.1$ during the landing phase.

Finally, during the landing phase, the integral gain ```angleIGain``` will be
chosen as $0.1$, while the proportional gain ```omegaPGain``` will be $2$.
Therefore, we will define two new variable names, ```angleIGainLanding``` and
```omegaPGainLanding```, to reflect their phase-specific use.

```
float vxPGain = 0.016;
float anglePGain = 1.0;
float angleIGain = 0.5;
float omegaPGain = 1.0;
float thrustAnglePGain = 0.25;

float vxPGainLanding = 0.1;
float angleIGainLanding = 0.1;
float omegaPGainLanding = 2.0;
```

The overall Processing code should look like this:

```
Environment env;
Command cmd;

int phase = 1;

// vertical controller variables
float vyDes;
float vyGain = 0.1;
float vyMax = 1.0;

float thrustPGain = 1.0;

// horizontal controller variables
float vxError;
float vxIError = 0.0;
float vxPGain = 0.016;

float angleDes;
float anglePGain = 1.0;
float angleIGain = 0.5;

float omegaDes;
float omegaPGain = 1.0;

float thrustAnglePGain = 0.25;

float vxDes;

float angleIGainLanding = 0.1;
float omegaPGainLanding = 2.0;
float vxPGainLanding = 0.1;

void setup() {
    size(1200, 600);
    
    env = new Environment();
    cmd = new Command();
}

void draw() {
    env.initialize();
    
    // -------------------------------------------------------------------------
    // ------------------------- Controller design -----------------------------
    // -------------------------------------------------------------------------
    // Vertical control
    vyDes = max(-vyMax, min(vyMax, - vyGain * env.getStarshipYPosition()));
    cmd.setThrustCommand(0.5 - thrustPGain * (env.getStarshipVy() - vyDes));
        
    // Horizontal control
    // Phase 1: approach
    if (phase == 1) {
        vxError = 3.0 - env.getStarshipVx();
        vxIError = vxIError + vxError * env.getSamplingTime();

        angleDes = anglePGain * vxError + angleIGain * vxIError;
        omegaDes = omegaPGain * (angleDes - env.getStarshipAngleInDegrees());

        cmd.setThrustAngleCommand(thrustAnglePGain *
            (env.getStarshipOmegaInDegrees() - omegaDes));
    }

    // Phase 2: Slow down
    if (phase == 2) {    
        // Horizontal control
        vxDes = vxPGain * (env.getStarshipXPosition() - 20.0);

        vxError = vxDes - env.getStarshipVx();
        vxIError = vxIError + vxError * env.getSamplingTime();

        angleDes = anglePGain * vxError + angleIGain * vxIError;
        omegaDes = omegaPGain * (angleDes - env.getStarshipAngleInDegrees());

        cmd.setThrustAngleCommand(thrustAnglePGain *
            (env.getStarshipOmegaInDegrees() - omegaDes));
    }

    // Phase 3: Landing
    if (phase == 3) {
        // Horizontal control
        vxDes = max(0.0, min(1.0,
            vxPGainLanding * (env.getStarshipXPosition() - 10.0))) + 0.05;

        vxError = vxDes - env.getStarshipVx();
        vxIError = vxIError + vxError * env.getSamplingTime();

        angleDes = anglePGain * vxError + angleIGain * vxIError;
        omegaDes = omegaPGainLanding * (angleDes -
            env.getStarshipAngleInDegrees());

        cmd.setThrustAngleCommand(thrustAnglePGain *
            (env.getStarshipOmegaInDegrees() - omegaDes));
    }

    // Phases switching
    if (abs(env.getStarshipXPosition() - env.getDestinationX()) < 200 &&
            phase == 1) {
        phase = 2;
    }
    if (abs(env.getStarshipYPosition()) < 1 && phase == 2) {
        phase = 3;
    }
    // -------------------------------------------------------------------------
    // -------------------------------------------------------------------------
    // -------------------------------------------------------------------------

    // Update
    env.updateStarship(cmd);
}
```

## Exercises

**Exercise 1.**
What would you do to make Starship reach $y=0$ more quickly?
Try to identify and tune the appropriate parameter.

**Exercise 2.**
Now, try to speed up the approach phase by enforcing a larger desired horizontal
speed.

**Exercise 3.**
Modify the controller so that Starship reaches $x=0$ before reaching $y=0$.
At this point, you will need to drive the horizontal speed to zero and slowly
descend to reach the landing point.
