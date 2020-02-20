# The Dark Side of the Shader Program (aka vertex shaders)

So far we've relentlessly focused on fragment shaders in the course, basically just using vertex shaders to put a full-screen quad up on the screen. But vertex shaders can be convenient for executing certain types of simulations, and can use many of the same feedback techniques we've exploited in fragment shaders. A combination of vertex and fragment-based simulations can also be useful for certain models, like flocking. So today we'll dive into vertex shaders, more info about array buffers, and learn about transform feedbacks.

## Displaying a single point using a vertex shader

Let's start from [our basic WebGL template](./webgl_template.htm) and alter it to display a simple point. We'll do this using `gl.POINTS` as our drawing mechanism. The basic procedure is:

1. Define an Array Buffer to hold the position of our point. We'll use this instead of our previous array buffer that we used to draw two triangles.
2. Change our call to `gl.drawArrays()` to use `gl.POINTS` instead of `gl.TRIANGLES`
3. We'll also update our code to use WebGL 2 (we had been using WebGL 1).
4. Update our vertex and fragment shaders.
5. Set our OpenGL background color to black, since we won't be filling the entire canvas.

### Change our array buffer to hold a single point

In the course webgl template, here is the current vertex array buffer code we're using:
```js
// create a buffer object to store vertices
const buffer = gl.createBuffer()

// point buffer at graphic context's ARRAY_BUFFER
gl.bindBuffer( gl.ARRAY_BUFFER, buffer )

const triangles = new Float32Array([
  -1, -1,
  1,  -1,
  -1, 1,
  -1, 1,
  1, -1,
  1, 1
])

// initialize memory for buffer and populate it. Give
// open gl hint contents will not change dynamically.
gl.bufferData( gl.ARRAY_BUFFER, triangles, gl.STATIC_DRAW )
```

We'll change this array to hold a single point in the center of our screen. Remember that OpenGL coordinates use {-1,-1} as the bottom-left corner of the screen, rising to the upper-right corner.

```
const buffer = gl.createBuffer()
gl.bindBuffer( gl.ARRAY_BUFFER, buffer )
gl.bufferData( gl.ARRAY_BUFFER, new Float32Array([ 0,0 ]), gl.STATIC_DRAW )
```

We'll see as we build up our particle effects that we'll primarily do this by creating more buffer objects to use. Buffer objects can store data that will be associated with each *vertex* in our WebGL scene. In this case, we're going to create one vertex and are providing it with XY coordinates (0,0) to place it in the center of the screen. Later on in this tutorial we'll want to change our `gl.STATIC_DRAW` to something that can be dynamically updated, but for now we only want to read the data in once.

### Change our call to `gl.drawArrays()`
In the last line of our `render` function, we'll change to draw with point sprites instead of triangles. Point sprites are 
"billboarded" to always face the camera.

`gl.drawArrays( gl.POINTS, 0, 1 )`

### Use WebGL 2
To use WebGL 2 we just need to change the context we request from our `<canvas>` element; we'll make some needed shader changes in the next step. Using WebGL 2 will enable us to use feedback techniques in the vertex shader.
  
Change the second line of our `window.onload` function to be: `gl = canvas.getContext( 'webgl2' )`

### Change our vertex and fragment shaders
Let's update our shaders. There's a couple of steps we need to take. First, we'll add `#version 300 es` to the top of every shader, to tell the GPU that we're using OpenGL ES 3.0. We'll change our point sprite size to make it bigger in our vertex shader. Last but not least note the `in` and `out` keywords that are used in the shader. As you might haved guessed, this specifies whether data is an input or an output to a shader. In this case our `a_position` attribute is an `in` while our `frag` variable is an `out`. There is no longer a `gl_FragColor` variable in fragment shaders; you simply replace this with an `out` variable of your choosing in your `main()` function.

```html
  <script id='vertex' type='x-shader/x-vertex'>#version 300 es
    precision mediump float;
    in vec2 a_position;

    void main() {
      gl_PointSize = 50.;
      gl_Position = vec4( a_pos, 0., 1. );
    }
  </script>

  <script id='fragment' type='x-shader/x-fragment'>#version 300 es
    precision mediump float;
    
    uniform float time;
    uniform vec2 resolution;
    
    out vec4 frag;
    
    void main() {
      frag = vec4(1.);
    }
  </script>
```

### Change the background color
Add these two lines to our `render()` function, before calling `gl.drawArrays`:

```js
  gl.clearColor( 0,0,0,1 )
  gl.clear( gl.COLOR_BUFFER_BIT )
```

The first line defines the background color, while the second line clears the buffer of color information, leaving the "clear" color behind.

With all that done, you should have a single point up. Great! Make a copy of this file and move on to the next section, where we'll use transform feedback to move our point across the screen over time.


## Moving our point sprite with transform feedback
Transform feedback lets us transform our vertices in the vertex shader and read these transformations back in on the next frame. Similar to how we setup our frame buffer objects with our fragment shader feedback systems, we'll need to make an extra buffer that we can pingpong reading fromm / writing to. We then add a few lines of code to our setup to enable the feedback, and then a few more to our render function.

### Creating our buffers

Change the buffer creation part of our `window.onload` to the following:

```js
// create a buffer object to store vertices
buffer1 = gl.createBuffer()
buffer2 = gl.createBuffer()

// point buffer at graphic context's ARRAY_BUFFER
gl.bindBuffer( gl.ARRAY_BUFFER, buffer1 )
// we will be constantly updating this buffer data
gl.bufferData( gl.ARRAY_BUFFER, new Float32Array([-1,0]), gl.DYNAMIC_COPY )

gl.bindBuffer( gl.ARRAY_BUFFER, buffer2 )
// four numbers, each with 4 bytes (32 bits)
gl.bufferData( gl.ARRAY_BUFFER, 8, gl.DYNAMIC_COPY )      
```

We're not doing much extra here, but we are changing the type of data from `gl.STATIC_DRAW` to `gl.DYNAMIC_COPY` to reflect that we'll be updating this data constantly. Also note that when we specify the data for our second buffer, we simply give its size in bytes, as opposed to creating a second underlying an array. 

### Setting up transform feedback

In our `window.onload` add the following code *after* creating our shader program, but *before* linking and using the program:

```js
transformFeedback = gl.createTransformFeedback()
gl.bindTransformFeedback(gl.TRANSFORM_FEEDBACK, transformFeedback)
gl.transformFeedbackVaryings( program, ['o_vpos'], gl.SEPARATE_ATTRIBS )
```

This tells our shader program that our `o_vpos` (short for output_verticalPosition) will be fed back into the shader.

### Update our render function
Change the render function to the following:

```js
function render() {
  // schedules render to be called the next time the video card requests 
  // a frame of video
  window.requestAnimationFrame( render )
  gl.clearColor(0,0,0,1)
  gl.clear(gl.COLOR_BUFFER_BIT)
  
  // update time on CPU and GPU
  time++
  gl.uniform1f( uTime, time )

  gl.bindBuffer( gl.ARRAY_BUFFER, buffer1 )
  gl.vertexAttribPointer( position, 2, gl.FLOAT, false, 0,0 )
  gl.bindBufferBase( gl.TRANSFORM_FEEDBACK_BUFFER, 0, buffer2 )
  
  gl.beginTransformFeedback( gl.POINTS )
  gl.drawArrays( gl.POINTS, 0, 1 )
  gl.endTransformFeedback()
  
  gl.bindBufferBase( gl.TRANSFORM_FEEDBACK_BUFFER, 0, null )
  
  let tmp = buffer1;  buffer1 = buffer2;  buffer2 = tmp
}
```

This addes in the necessary transform feedback calls, as well as swapping between our read/write buffers of data.

### Update our vertex shader

Last but not least:
```c
#version 300 es
precision mediump float;
in vec2 a_pos;

out vec2 o_vpos;

void main() {
  float x = a_pos.x + .01;
  if( x >= 1. ) x = -1.;
  
  gl_PointSize = 5.;
  o_vpos = vec2( x, a_pos.y );
  
  gl_Position = vec4( o_vpos, 0., 1. );
}
```

Since `o_vpos` is an `out`, this now gets exported from the vertex shader and stored in our buffer object. This only happens because of our previous call to: `gl.transformFeedbackVaryings( program, ['o_vpos'], gl.SEPARATE_ATTRIBS )`

Make a copy of your file and continue working from that.

## Lots of particles

First specify a global `particleCount` variable:
`particleCount = 1024`

OK, let's change our buffer data inside of `window.onload` to store many particles.

```js
const particleData = new Float32Array( particleCount * 4 )
for( let i = 0; i < particleCount * 4; i+= 4 ) {
  particleData[ i ] = -1
  particleData[ i + 1 ] = -1 + Math.random() * 2
  particleData[ i + 2] = Math.random() * .025
}

// create a buffer object to store vertices
buffer1 = gl.createBuffer()
buffer2 = gl.createBuffer()

// point buffer at graphic context's ARRAY_BUFFER
gl.bindBuffer( gl.ARRAY_BUFFER, buffer1 )
// we will be constantly updating this buffer data
gl.bufferData( gl.ARRAY_BUFFER, particleData, gl.DYNAMIC_COPY )

gl.bindBuffer( gl.ARRAY_BUFFER, buffer2 )
// four numbers, each with 4 bytes (32 bits)
gl.bufferData( gl.ARRAY_BUFFER, particleCount * 4 * 4, gl.DYNAMIC_COPY )
```

We'll use the `z` member to store a velocity for our particles.

### Update our shaders
This basically just adds in dynamic velocity, and turns down the brightness of our point sprites.

```html
  <script id='vertex' type='x-shader/x-vertex'>#version 300 es
    precision mediump float;
    in vec4 a_pos;
    out vec4 o_vpos;
    void main() {
      float x = a_pos.x + a_pos.z;
      if( x >= 1. ) x = -1.;
      
      gl_PointSize = 10.;
      o_vpos = vec4( x, a_pos.y, a_pos.z, 1.);
      gl_Position = o_vpos;
    }
  </script>

  <!-- fragment shader -->
  <script id='fragment' type='x-shader/x-fragment'>#version 300 es
    precision mediump float;
    
    uniform float time;
    uniform vec2 resolution;
    
    out vec4 o_frag;
    
    void main() {
      o_frag = vec4(.5,.1,.1,.1);
    }
  </script>
```  

### Update our render function
All we need to do here is change our call to `gl.drawArrays` to reflect the correct count, in this case `particleCount`:

`gl.drawArrays( gl.POINTS, 0, particleCount )`

### Enable blending
Add this to our window.onload function to get blending:

```js
gl.enable(gl.BLEND);
gl.blendFunc(gl.SRC_ALPHA,gl.ONE_MINUS_SRC_ALPHA);
```

