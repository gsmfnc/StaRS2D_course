---
title: 'Control engineering basics using an educational 2D Starship Re-entry Simulator'
tags:
  - Processing
  - Space
  - Control engineering
authors:
  - name: Francesco Gismondi
    orcid: 0000-0002-3506-4077
date: 15 October 2025
bibliography: paper.bib
---

---

# Summary

This paper presents an open-source course to introduce the reader to control
engineering.
Through the use of a 2D Starship simulator on Processing `[reas2006processing]`,
the reader will be guided to design
controllers that complete the fascinating re-entry task.

# Statement of Need

Control engineering is becoming more and more important nowadays: just to
give an idea, the design of
modern aircraft, spaceships, industrial automation and autonomous cars heavily
relies on it.
However, even though control is everywhere, it is unknown to most.

Young students interested in technology typically think of control concepts as
abstract and have difficulties in finding how they relate to real-world
applications.
This course represents a first step to fill the gap by introducing control
engineering concepts without complicated mathematics and relying on a simulator
of a real-world-inspired problem.

The course presented in this paper aims at giving insights on what control
engineering is through a simple 2D Starship simulator
(StaRS 2D `[@gismondi2025stars2d]`) implemented in Processing
`[@reas2006processing]`.
This simulator allows to interact with Starship during its re-entry task.
Control engineering is crucial for the outcome of this mission and Starship has
gained lots of attention, thus making this course a possible motivating and
fascinating entry point to control theory.

The target audience are young students and early learners.
Little experience with coding and a basic understanding of physics, mathematics
and trigonometry are required.
The course is structured in three lessons and it is mainly intended to be
adopted by high-school students or early undergraduates.

# Course structure

This section describes the course structure by giving the links to each lesson
in the following table and then briefly explaining their contents in the
subsequent subsections.

| Lesson | Link |
|------|------|
| Lesson 1: A Basic Flight Controller | [Link](https://github.com/gsmfnc/StaRS2D_course/blob/main/lesson_01_basic_control/lesson_01_basic_control.md) |
| Lesson 2: Proportional-Integral (PI) Controllers | [Link](https://github.com/gsmfnc/StaRS2D_course/blob/main/lesson_02_proportional_control/lesson_2_proportional_control.md) |
| Lesson 3: A More Realistic Starship Model | [Link](https://github.com/gsmfnc/StaRS2D_course/blob/main/lesson_03_accelerations/lesson_3_accelerations.md) |

## Lesson 1: A Basic Flight Controller

The first lesson introduces the Starship re-entry problem as a motivating
scenario and describes the StaRS 2D simulator.
This simulator is crucial to visualize trajectories and link control inputs to
the vehicle motion.
Then, the reader is guided to design an intuitive controller, based
on computer programming logics that should be familiar.

## Lesson 2: Proportional-Integral (PI) Controllers

The second lesson points out the flaws in using those logics and introduces the
Proportional-Integral (PI) controllers.
These are explained by observing through the simulator possible problems that
may be introduced by the controller of the first lesson when something does not
go as planned.
Then, the parameters of the controller are tuned experimentally, showing the
idea of iterative design and testing through a simulator.

## Lesson 3: A More Realistic Starship Model

The third lesson considers a more realistic Starship model (adding simple
dynamics equations) and introduces the concept of multi-loop control.
The reader understands how to separate control objectives and design a
hierarchical PI controller.

# Conclusion

Control engineering plays a crucial role in the development of modern
technology.
This paper proposes a course designed to introduce young learners to this field
by
allowing them to guide Starship safely to the launch tower through a 2D
simulator.
The mathematical complexity behind control engineering is deliberately set aside
and the reader
is guided directly into the design of a flight controller, providing a practical
demonstration of its operating principles.
The simulator is lightweight, implemented in the open-source software
Processing and requires minimal setup, making it easily accessible for
educational use.

Future work will deal with the development of simulators and courses centered on
other exciting applications of control engineering.
Moreover, the course material will gradually include the mathematical
foundations of
control engineering, giving an immediate illustration of their practical
interpretation.

# References
