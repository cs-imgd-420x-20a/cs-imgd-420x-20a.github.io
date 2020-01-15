# CS/IMGD 420x: Graphical Simulation of Physical Systems

This course focuses on the digital simulation of physical systems and strategies for interactive representation. Topics include:

- Parallel computing on graphics processing units (GPUs)  
- Realtime graphics techniques  
- The history of analog interactive visuals  
- Interaction techniques for controlling digital simulations  

Students will explore analog physical systems (such as fluids and video feedback) and use these explorations to guide the development of digital simulations. The class is explicitly designed to encourage both technical and aesthetic exploration.  

All students will be expected to maintain a course website that contains links to all assignments. Videos should be posted to "permanent" locations (Vimeo, YouTube etc.) and embedded / linked in your course website. Details on this website can be found in the [onboarding assignment](./onboarding.md). 

## Course Outline

This is an experimental course being taught for the first time; as such this outline is subject to change. I am happy to make minor additions if there are related topics of interest that are not menttioned here.  

### Week 1: Getting Started / Review
1/15 - *Basic Intro to WebGL / Shader programmming*. Assignment:  Complete the [onboarding assignment](./onboarding.md). Due 1/16. 
1/16 - *GLSL waveforms and GLSL Live Coding*.   
  

### Week 2: Analog / Digital Video Synthesis, Rendering to Textures
1/20 - *History and Techniques of Analog Video Synthesis*. Assignment: Complete the analog synthesis (Vidiot) tutorial. Create a one minute video showing an interesting parameter space you found to explore using the Vidiot. Work towards recreating this sketch in GLSL, by implementing a minimum of eight Vidiot features. Some of these will be simple (noise, modulation) some will be trickier (video input, waveforms, audio input).  
1/23 - *Video input / processing, Analog video feedback, no-input performances*.  

### Week 3: Automata, Reaction Diffusion
1/27 - 1D / 2D Automata, Rendering to Texture
1/30 - Reaction Diffusion, Convolution

### Week 4: Video feedback + Rendering to Texture
2/3 - 
2/7 - TBA (I'll be at [http://iclc.livecodenetwork.org/2020/schedule.html](ICLC)).

### Week 5: Perlin Flows, Navier-Stokes, and other Fluid Simulations
2/10 - Guest lecture with [Alexander Dupuis](http://alexanderdupuis.com/) on performing with video feedback systems. Perlin Flows  
2/13 - Navier-Stokes and other fluid simulations  

### Week 6: Interaction in Digital Arts
2/17 - Guest lecture with [https://creativecoding.soe.ucsc.edu/angus/](Angus Forbes) on interacting with fluid simulations  
2/20 - OSC / MIDI / Mapping strategies for digital arts practice  

### Week 7: CUDA & OpenCL 
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
Attendance is required. Please notified the instructor in advance if you must miss class. Missing more than two sessions will result in an automatic grade of NR.

# Academic Integrity
The goal of this class is to both create aesthetically interesting content and understand the code used to create it. In order to understand the code, you need to author it yourself. Copying and pasting code is not allowed in this class, unless explicitly stated by the instructor. If you have a question about this, ask the instructor!!

Collaboration is encouraged in this class. There are many ways in which you can assist others without giving them code and answers. Providing low-level implemetation details (small code fragments that contribute to, but don't complete on their own, major portions of assignments) is acceptable. For example, showing how to call the `smoothsttep()` function in GLSL.
