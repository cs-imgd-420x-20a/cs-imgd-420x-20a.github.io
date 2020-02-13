# CS/IMGD 420x: Graphical Simulation of Physical Systems

This course focuses on the digital simulation of physical systems and strategies for interactive representation. Topics include:

- Parallel computing on graphics processing units (GPUs)  
- Realtime graphics techniques  
- The history of analog interactive visuals  
- Interaction techniques for controlling digital simulations  

Students will explore analog physical systems (such as fluids and video feedback) and use these explorations to guide the development of digital simulations. The class is explicitly designed to encourage both technical and aesthetic exploration.  

All students will be expected to maintain a course website that contains links to all assignments. Videos should be posted to "permanent" locations (Vimeo, YouTube etc.) and embedded / linked in your course website. Details on this website can be found in the [onboarding assignment](./onboarding.md). 

Assignments and examples for this class will primarily use GLSL / JavaScript, however, other environments and systems for GPU programming will be discussed.

## Course Outline

This is an experimental course being taught for the first time; as such this outline is subject to change. I am happy to make minor additions if there are related topics of interest that are not menttioned here.  

### Week 1: Getting Started / Review
1/15 - *Basic Intro to WebGL / Shader programmming*. [Notes](./notes.day1.md)Assignment:  Complete the [onboarding assignment](./onboarding.md). Due 1/16.  
1/16 - *GLSL Live Coding*.  Assignment:  I. Read / experiment with [The Book of Shaders](http://thebookofshaders.com) up through the lesson on [Noise](https://thebookofshaders.com/11/). There may be a quiz on this on 1/20. II. Complete [Shader Live Coding Assignment](./shader_live_coding.md) also due 1/20.

### Week 2: Analog / Video Synthesis
1/23 - *History and Techniques of Analog Video Synthesis*. [Notes](./notes.day3.md).   
Assignment: [Analog Video Synthesis -> Digital Video Synthesis](./analog_to_digital.md). Due 1/30 by start of class.  
Resources:  
- [Vidiot Tutorial](./vidiot_tutorial.md)
- [WebGL Fragment Shader Template](./webgl_template.md)
  
### Week 3: Textures, Video, and Video Feedback
1/27 - Using textures, live video input. [Notes](./notes.day4.md)  
1/30 - Video feedback, motion blurs etc.  

### Week 4: Automata, Reaction Diffusion, and Rendering to Texture
2/3 - 1D / 2D Automata, Rendering to Texture, Reaction Diffusion. [Mini-Assignment](./automata_assignment.md) due 2/10.  
2/7 - NO CLASS... complete the Reaction Diffusion tutorial. I'll be at [the International Conference on Live Coding](http://iclc.livecodenetwork.org/2020/schedule.html).  

### Week 5: Perlin Flows, Navier-Stokes, and other Fluid Simulations
2/10 - Guest lecture with [Alexander Dupuis](http://alexanderdupuis.com/) on performing with video feedback systems... MOVED. To be rescheduled. Assignment: Read the chapters on [Cellular Noise](https://thebookofshaders.com/12/) and [Fractal Brownian Motion](https://thebookofshaders.com/13/) in the Book of Shaders.  
2/13 - Perlin Flows / fluid simulations.  

### Week 6: Interaction in Digital Arts
2/17 - Guest lecture with [(Angus Forbes](https://creativecoding.soe.ucsc.edu/angus/) on interacting with fluid simulations  
2/20 - OSC / MIDI / Mapping strategies for digital arts practice  

### Week 7: OpenCL + WebGPU
2/24 - Libraries for computing on the GPU  
2/27 - NO CLASS: Academic Advising Day  
2/28 - READING / MAKEUP DAY  

### Week 8: Final Project Presentations &amp; Wrapup  
3/2 - Wrap-up, preliminary final project critiques  
3/5 - Final Project Presentations  

# Grades
Your course grade comes from three parts:

Assignments (50%)  
Final Project (40%)  
Quizzes, in-class assignments, attendance (10%)  

There will most likely be 4–5 assignments in the course in addition to the final project. I reserve the right to adjust the above if needed. 

I don’t accept late homework, doing so is unfair to your fellow students and to course staff. It is much better to submit partially complete work than nothing at all. If you don’t have the homework done on time you will receive a zero for that assignment. You are expected to complete the assignment and have it deployed by the due date.

# Attendance
Attendance is required. Please notify the instructor in advance if you must miss class. Missing more than two sessions will result in an automatic grade of NR.

# Academic Integrity
The goal of this class is to both create aesthetically interesting content and understand the code used to create it. In order to understand the code, you need to author it yourself. Copying and pasting code is not allowed in this class, unless explicitly stated by the instructor. If you have a question about this, ask the instructor!!

Collaboration is encouraged in this class. There are many ways in which you can assist others without giving them code and answers. Providing low-level implemetation details (small code fragments that contribute to, but don't complete on their own, major portions of assignments) is great. For example, telling a classmate that they need to use the `smoothstep()` function in GLSL and showing them how to call it.
