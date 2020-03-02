# Shaders + Simulations in Other Environs

We've mostly look at running our simulations in the context of WebGL in this course. Today we'll get a brief overview of how GPU-based shaders and simulations run in a variety of other contexts.

1. OpenCL
2. Metal
3. WebGPU
4. Unity

That's leaving out two big players in the space, CUDA and Vulkan, just because I don't have a NVIDIA graphics card. But many of the concepts are roughly the same to what we'll be discussing in other systems.

## OpenCL
OpenCL is a C or C++ wrapper around operations on the GPU, called *kernels*. A kernel is typically designed to operate on buffers of data. For example, here's a kernel that squares a buffer of numbers:

```c
__kernel void square( __global float* input, __global float* output ) {
  size_t i = get_global_id(0);
  output[i] = input[i] * input[i];
}
```
(Adapted from the [macOS OpenCL documentation](https://developer.apple.com/library/archive/samplecode/OpenCL_Hello_World_Example/Listings/hello_c.html)).

... we can see that, minus a few extra keywords (`global`, `kernel`), we're basically looking at C code. You can look at the fully commented source code above to get the details on how to setup the operation, but basically:

1. Get a reference to our GPU
2. Create a program using our kernel
3. Create our two buffers (input and output)
4. "Enqueue" our operation and pass our two buffers as arguments to the operation.
5. Specify the range of data in our buffers that the operation will run on, and how many threads will be used.
6. Read the output of the operation from the buffer into an array of type `float` for further processing on the CPU.
7. Release any memory used.

## Metal
Metal is the solution in macOS / iOS for low-level graphics and GPGPU programming. For GPGPU programming, the basic ideas is the similar to OpenCL... we provide a kernel that is compiled to run on the GPU. Below is a kernel for adding two numbers:

```c
#include <metal_stdlib>
using namespace metal;
/// This is a Metal Shading Language (MSL) function equivalent to the add_arrays() C function, used to perform the calculation on a GPU.
kernel void add_arrays(device const float* inA,
                       device const float* inB,
                       device float* result,
                       uint index [[thread_position_in_grid]])
{
    result[index] = inA[index] + inB[index];
}
```

The big difference from OpenCL is the magickal `[[thread_position_in_grid]]` which is an instruction to Metal to define the position in the *threadgroup*, which helps us divide the task among different threads. The boilerplate to run this kernel is slightly streamlined from the C API used by OpenCL, you can see this [here](https://developer.apple.com/documentation/metal/basic_tasks_and_concepts/performing_calculations_on_a_gpu?preferredLanguage=occ).

## WebGPU

To use WebGPU in Chrome type `chrome://flags` in the address bar, and then do a search for "GPU". Turn the "Unsafe WebGPU" option on. You may need to restart your browser after doing this.

Once this is done, [there's a nice website](https://austineng.github.io/webgpu-samples/#) you can checkout for different examples of WebGPU in action, including a buffer-swapping flocking simulation. 

Note that WebGPU is a framework for GPGPU programing *as well as* a low-level graphics rendering API. Many of the examples are designed to explore rendering techniques rather than GPGPU techniques. It is intended that, with time, WebGPU will eventually be the defacto 3D-rendering engine for the browser... although it's low-level enough that there will probably be a large ecosystem of higher-level abstractions that grow around it.

Google has a [nice walkthrough of GPGPU programmming using WebGPU](https://developers.google.com/web/updates/2019/08/get-started-with-gpu-compute-on-the-web) that doesn't involve any rendering. Note that the "kernel" code is just regular GLSL, as opposed to a dedicated kernel language or C...

## Unity

We're going to play around a bit with an [implementation of the game of life running on the GPU](https://github.com/sevelee/2d-game-of-life-by-frag-shader).

When you open this project up, you'll see there's a simple plane displayed with a some materials and scripts assigned to it. The most important of the scripts is the "WorldContorl" (their spelling mistake, not mine) script. WorldControl (I'll just name it correctly from here on out) has the relevant setup code for getting our textures loaded, passing them to our shader, rendering to a texture, and then swapping the textures out. The basic pipeline goes like this:

1. We have a texture that determines the starting condition for our simulation. We specify this texture in the "Original Texture" field of the GUI controlling our WorldControl script.
2. We also pass in a blank texture for the "Render Texture" field of the WorldControl script GUI. This will be one of our two swap textures; we'll create the other programmatically.
3. In our WorldControl script, we copy the "Original Texture" into our "Render Texture" using `Graphics.Blit()`. We then create our swap texture with the same dimensions as our "Render Texture"
4. On each frame update, we call `Graphics.Blit()` and pass in our two textures and our `Material` object, which has a reference to our shader. We then call `Graphics.Blit()` again but swap the order of our textures. 

If you look at the main shader file that's applied to the Material used by our plane, you'll see it's fairly readable, despite being written in Cg instead of GLSL.

Now we can start having a bit of fun with this. We can substitute our Plane mesh for a 3D geometry to have the game of life run on that. Just select "World" in the scene hierarchy, and then select the Plane (Mesh Filter) tab in the Inspector sidebar on the right side of your screen. Click on the small target icon to choose a new mesh.

We can then add a small script to rotate the mesh. Go to the Scripts folder in your Assets Manager, and then create a new C# script and paste in the following:

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Rotate : MonoBehaviour {
    void Start(){}
    void Update(){
      transform.Rotate(0, 0, .1f, Space.Self);
    }
}
```

Select the "World" in your scene hierarchy, and then drag the new script file into the Inspector, *above the WorldControl script*. If you hit play there should now be some simple rotation.

Last but not least we should probably update our texture size to get better resolution. Whenever we do this, we need to update it in two places: first, on the render target texture we pass to our material, and second, on the Cell Material that uses the Cell Shader. Try setting the texture values to 512x512 to see the difference.

Unity also has [GPGPU (compute) shaders](https://docs.unity3d.com/Manual/class-ComputeShader.html)