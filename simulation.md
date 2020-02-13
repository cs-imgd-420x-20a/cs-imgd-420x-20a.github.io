# Assignment 4 - Simulation programming

At this point in the course, we have looked at a couple of different simulations running on the GPU, and also explored using FBOs for video feedback. Now it's time for you to implement your own simulation using FBOs to read/write values,. Some suggested simulations include:

1. [SmoothLife](https://arxiv.org/pdf/1111.1567.pdf): Conway's Game of Life in a continuous domain. [Video Example](https://www.youtube.com/watch?v=KJe9H6qS82I)
2. [Langton's Ants](https://en.wikipedia.org/wiki/Langton%27s_ant) or perhaps some other form of [Turmite](https://en.wikipedia.org/wiki/Turmite). [Video explanation/examples](https://www.youtube.com/watch?v=w6XQQhCgq5c) with sound by the Vasulkas. Note this is typically simple enough to run on the CPU unless you are running hundreds/thousands of ants at once.
3. [Flocking (aka Boids)](http://www.red3d.com/cwr/boids/). A great tutorial on this can be found in Chapter 6 of [The Nature of Code](https://natureofcode.com/book/chapter-6-autonomous-agents/). This can also be run on the CPU, but it will be interesting to see how many agents you can get with a GPU accelerated version.
4. [Slime Mold](https://softologyblog.wordpress.com/2019/04/11/physarum-simulations/). [Video demo](https://www.youtube.com/watch?v=xFimP0gFyIc)

In addition to choosing the simulation you'd like to research (be careful! some are much more difficult than others), you can also choose whether or not you'd like to make a composition with the simulation, or add interactivity to it. The grading rubric will be slightly different according to which option you choose.

Grading Rubric
---
- 10% Posted to your personal course website correctly, with video in permanent hosting location. No spelling or grammatical errors.
- 10% Summary paragraph concisely explaining technical and aesthetic goals.
- 10% Paragraph describing feedback received and how it met / didn't meet your expectations.
- 10% "Modular" js / glsl design. Separate files for your JavaScript file and each of your shaders, combined via browserify / glslify. Place all files in a GitHub repo and include a link to this repo on your project page.
- 35% Simluation implementation. Please include a paragraph describing the general process of implementing your simulation and any challenges you faced.
- 15% Development of piece OR interaction design.
  - For compositions, how does your piece change over time? Is there a peak? How did you parameterize the simulation and use these parameters?
  - For interaction, is interacting with the simulation fun? Is it intuitive? How did you parameterize the simulation and what did you expose to interactivity?
- 10% Visual Aesthetics. Is there a discernible theme in the work? What constraints are in place (limited color-palette, limited number of shapes etc.)? 

This assignment will count for 10–12.5% of your final grade, depending on whether we do four or five assignments in the course. Please take time to ensure that spelling / grammar are correct and that the assignment is linked / posted / hosted correctly so that you don't lose any easy points.
