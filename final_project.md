# Final Project

For the final project, the goal is to explore visual processing and interaction with your simulation. Your project should incorporate *two* of the following three explorations:

1. Given a simulation, how can we interact with it in interesting ways? We will discuss some of these options in class on Monday, but for now consider:

-  Interaction using mobile devices. How can we "touch" the simulation, and what new opportunities would multitouch offer versus a simple mouse? What about using the accelerometer / gyro / compass?
-  Interaction via MIDI devices or music programming environments.
-  Interaction via other types of gestural control (Kinect, Leap Motion, RGB-based computer vision etc.)

2. Consider the representation of your simulation, and explore rendering techniques to improve it aesthetically.

- What types of post-processing effects could be used to add visual interest to your work? Think motion blurs, depth extrusions, lighting.
- Could the simulation instead be used to manipulate another visual process? What happens when reaction-diffusion is used to spatially control the amount of feedback in a live video signal?
- How can your simulation be applied to 3D? Simply texturing a 3D object is insufficient here... what about bump / displacement mapping? Terrain / maze generation?

3. Implement another simulation of your choosing.

If you don't choose #3 as one of your two options, you are welcome to use the simulation you completed for the last assignment as your starting point, or perhaps begin with reaction diffusion.

Grading Rubric
---
- 10% Posted to your personal course website correctly, with video documentation in a permanent hosting location. No spelling or grammatical errors. Video documentation should include a brief demonstration of interacting with your simulation; if representation was your focus you could instead simply allow it to evolve over time. Please also include a link to a running version of your simulation.
- 10% 300–400 words concisely explaining technical and aesthetic goals.
- 10% A paragraph describing feedback received and how it met / didn't meet your expectations.
- 10% "Modular" js / glsl design. Separate files for your JavaScript file and each of your shaders, combined via browserify / glslify. Place all files in a GitHub repo and include a link to this repo on your project page. *Note:* as discussed in class, you don't have to use abstractions like `gl-toy` if you'd prefer to continue working with raw WebGL code. You can use the [game of life example using gl-toy](https://github.com/cs-imgd-420x-20a/glslify_gol) as a starting point. 
- 50% Implementation. Please include 300-400 words describing the general process of implementing your user experience and any challenges you faced. Use this text justify the work you put into the project, which should be about ~30 hours over the next two weeks.
- 10% Visual Aesthetics. Is there a discernible theme in the work? What constraints are in place? Briefly describe your efforts in this area.

Please take time to ensure that spelling / grammar are correct and that the project is linked / posted / hosted correctly so that you don't lose any easy points. You will be expected to demo the project on the final day of class (3/5). All final project documentation must be available on your course website by 11:59 PM on Saturday, March 7th.