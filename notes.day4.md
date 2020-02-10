# Textures and GLSL

In this workshop we'll work towards understanding how textures work on the GPU. Textures are generic objects for transferring 2D and 3D data from the CPU to the GPU. We'll start with using an HTML <canvas> object as a texture, and work towards using live video feeds. In addition to being a useful starting point to exploring textures, the techniques we'll use in this exercise are also useful for image processing on the GPU.
  
We'll build off of the [WebGL Template](./webgl_template.htm) that we covered in the last class session.
  
## Getting a canvas element on the GPU

The <canvas> element provides raster-drawing to HTML pages (both 2D and 3D). It's a quick and easy way to make a 2D texture to load onto the GPU. Let's go ahead and make a canvas in our `window.onload` function, paint it entirely green, and display it next to the canvas in the template that runs the fragment shader. First, let's get rid of the lines that set the width and the height of our shader canvas, so that it no longer fills the whole window.

```js
/* COMMENT THESE TWO LINES OUT AT THE START OF WINDOW.ONLOAD */
//canvas.width = window.innerWidth
//canvas.height = window.innerHeight
```

Next, add `greencanvas` and `greenctx` to the variable list at the top of the JS `<script>` tag. Finally, inside `window.onload`, go ahead and make our canvas.
```js
// create our canvas
greencanvas = document.createElement( 'canvas' )
// get a drawing context for our canvas
greenctx = greencanvas.getContext( '2d' )
// set our painting color
greenctx.fillStyle = 'green'
// draw our rectangle... 350x150 are the default canvas dimensions
greenctx.fillRect( 0,0,350,150 )
// put the canvas on the page
document.body.appendChild( greencanvas )
```

If you load the modified template, you should see the shader running in our original canvas and the green canvas displayed next to it.

### Making a OpenGL texture

We'll create a function to instantiate our texture, and then call that function from within our `window.onload` method.

```js
function makeTexture() {
  // create an OpenGL texture object
  const texture = gl.createTexture()

  // since canvas draws from the top and shaders draw from the bottom, we
  // have to flip our canvas when using it as a shader.
  gl.pixelStorei(gl.UNPACK_FLIP_Y_WEBGL, true);

  // this tells OpenGL which texture object to use for subsequent operations
  gl.bindTexture( gl.TEXTURE_2D, texture )

  gl.texImage2D( 
    gl.TEXTURE_2D,    // target: you will always want gl.TEXTURE_2D
    0,                // level of detail: 0 is the base
    gl.RGBA, gl.RGBA, // color formats
    gl.UNSIGNED_BYTE, // type: the type of texture data; 0-255
    greencanvas       // pixel source: could also be video or image
  )
  // how to map when texture element is more than one pixel
  gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_MAG_FILTER, gl.LINEAR )
  // how to map when texture element is less than one pixel
  gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_MIN_FILTER, gl.LINEAR )
}
```

Place a call to our new `makeTexture` function in our `window.onload`, right before the call to `render()`.

### Update the fragment shader to show the 2D canvas in the WebGL canvas

Two things we need to do to finish this off. First, we'll need to add a uniform giving our fragment shader the resolution of our canvases. We've made it easy on ourselves; since both our canvases are the same resolution we only need one `vec2` uniform for this.

Place the following code in our `window.onload` script, right under where we initialize the `time` uniform. Note: this code is already in the updated webgl template.

```js
const uRes = gl.getUniformLocation( program, 'resolution' )
gl.uniform2f( uRes, gl.drawingBufferWidth, gl.drawingBufferHeight )
```

The last task is to (finally) update our fragment shader.

```c
#ifdef GL_ES
precision mediump float;
#endif

uniform float time;
uniform vec2 resolution;

// GLSL gives us this for free... our first sampler2D automatically
// points to our first bound texture.
uniform sampler2D uSampler;

void main() {
  // texture2D lets us lookup a pixel in a texture by passing xy values from 0–1
  // to get those normalized values we divide gl_FragCoord (measured in pixels) by our resolution
  vec4 textureColor = texture2D( uSampler, gl_FragCoord.xy / resolution );
  gl_FragColor = textureColor;
}
```

Victory! You should now have a canvas displayed as a texture on the GPU. We can use such textures to store all types of interesting information, as we'll see when we start doing simulations (soon!)

## Putting an image as a texture on the GPU

Most of the hard stuff is done now, so we can play around with changing the type of data we're sending to the GPU. If we remove all of the `greencanvas` related code, we can easily replace it with an image. We can use a web search to find an image that's also 256x256 pixels. It will need to come from a server with cross-origin resource sharing enabled (CORS), and we'll need to add anonymous crossOrigin credentials. The example immage below resides on a server (wikimedia) with CORS enabled.

We'll also tell our `makeTexture` function to be called once the image is finised downloading. Go ahead and delete the line that automatically calls it in `window.onload` above `render()`

Add in some image code to our `.onload` function:

```js
img = document.createElement( 'img' )
img.src = 'https://upload.wikimedia.org/wikipedia/commons/b/be/JPEG_example_image.jpg'
img.crossOrigin = 'Anonymous'
img.onload = makeTexture
document.body.appendChild( img )
```

### Image processing (quick blur)

We can do some simple image processing using a convolution kernel. Basically, for every pixel, we'll look at some of its surrounding neighbors, and then use the color of the neighbors and the color of the pixel to determine the pixels filtered value. We'll do this in a fairly naïve way first, and then borrow some code to do it more "professionally".

Naïve:
```c
#ifdef GL_ES
precision mediump float;
#endif

uniform float time;
uniform vec2 resolution;

uniform sampler2D uSampler;

void main() {
  // since texture2D accepts values between 0–1 for x and y coordinates, we need to
  // figure out how much moving one pixel in a given direction is in normalized coordinates.
  vec2 offset1 = 1. / resolution;
  
  // get the original color value of our pixel
  vec3 base = texture2D( uSampler, gl_FragCoord.xy / resolution ).rgb * .25;
  
  // get the color one pixel to the left and one pixel down
  // note that we're weighting this *more* than our original pixel (.375 vs. .25)
  vec3 leftdown = texture2D( uSampler, gl_FragCoord.xy / resolution - offset1 ).rgb * .375;
  
  // get the color one pixel to the right and one pixel up
  vec3 rightup  = texture2D( uSampler, gl_FragCoord.xy / resolution + offset1 ).rgb * .375;

  // add them all together and we have a really cheap blur
  gl_FragColor = vec4( base + leftdown + rightup, 1. );
}
```

*Using a better blur*  
Let's steal [a better blur function](https://github.com/Jam3/glsl-fast-gaussian-blur/blob/master/5.glsl). Note we don't need the final line of code in this example, as it's specific to a technology called glslify that we'll discuss later in the course. 

The function needs to be passed our sampler, the current pixel coordinates, the resolution of the shader, and a blur direction. It uses a gaussian distribution to calculate the blur in five passes.


```c
#ifdef GL_ES
precision mediump float;
#endif

uniform float time;
uniform vec2 resolution;

uniform sampler2D uSampler;

vec4 blur5(sampler2D image, vec2 uv, vec2 res, vec2 direction) {
  vec4 color = vec4(0.0);
  vec2 off1 = vec2(1.3333333333333333) * direction;
  color += texture2D(image, uv) * 0.29411764705882354;
  color += texture2D(image, uv + (off1 / res)) * 0.35294117647058826;
  color += texture2D(image, uv - (off1 / res)) * 0.35294117647058826;
  return color; 
}

void main() {
  vec2 pos = gl_FragCoord.xy / resolution;
  gl_FragColor = blur5( uSampler, pos, resolution, vec2(2.) )
}
```

The direction parameter gives us control over the amount of blur as well. Later in the course, we'll look at getting motion blurs write each frame of video to a texture that we can then access to use in blurring on the subsequent frame.

## Video input from webcams

We don't need to do a whole lot to do use video, just change from using our image/canvas to using a live video source instead. First, add three new variables to our declaration at the top of our JS element: `video`, `textureLoaded`, and `texture`. You can also safely remove `greencanvas, greenctx, img`. Then we'll add a function to start our video up and call it in our `.onload` call. Note that users will have to grant permission before the video begins.

```js
function getVideo() {
  video = document.createElement( 'video' )

  navigator.mediaDevices.getUserMedia({
    video:true
  }).then( stream => { 
    video.srcObject = stream
    video.play()
    makeTexture()
  }) 
    
  return video
}
```

In the above code, we wait until the video has started playing to make our texture... this will help prevent errors that might occurred from trying to display the video before it is playing. Here's `makeTexture`, which is almost identical from our image example but contains a couple of additions / variations:

```js
function makeTexture() {
  // create an OpenGL texture object
  texture = gl.createTexture()
  
  // this tells OpenGL which texture object to use for subsequent operations
  gl.bindTexture( gl.TEXTURE_2D, texture )
    
  // since canvas draws from the top and shaders draw from the bottom, we
  // have to flip our canvas when using it as a shader.
  gl.pixelStorei( gl.UNPACK_FLIP_Y_WEBGL, true )

  // how to map when texture element is more than one pixel
  gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_MAG_FILTER, gl.LINEAR )
  // how to map when texture element is less than one pixel
  gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_MIN_FILTER, gl.LINEAR )
  
  // you must have these properties defined for the video texture to
  // work correctly
  gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_WRAP_S, gl.CLAMP_TO_EDGE )
  gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_WRAP_T, gl.CLAMP_TO_EDGE )
  
  // let our render loop know when the texture is ready
  textureLoaded = true
}
```

Differences:
- We removed the call to `gl.textureImage2D`. In our example using a static image, it was fine
to call this when the texture was first created, because the image never changes. For live video, we need to call it once per frame in our render loop instead.  

- We added calls to `gl.texParameteri` to tell our texture how to wrap (clamping)

- We set the value of a variable, `textureLoaded`, to true. In our render loop we will check for this before attempting to load our texture.

Almost there! The last two tasks are to define our `render` function and to make sure we call `getVideo` inside of our `window.onload` function. Put the call to `getVideo()` at the end of `window.onload` now. Then redefine the `render` functions as follows:

```js
// keep track of time via incremental frame counter
let time = 0
render = function() {
  // schedules render to be called the next time the video card requests 
  // a frame of video
  window.requestAnimationFrame( render )
  
  // check to see if video is playing and the texture has been created
  if( textureLoaded === true ) {
    // send texture data to GPU    
    gl.texImage2D( 
      gl.TEXTURE_2D,    // target: you will always want gl.TEXTURE_2D
      0,                // level of detail: 0 is the base
      gl.RGBA, gl.RGBA, // color formats
      gl.UNSIGNED_BYTE, // type: the type of texture data; 0-255
      video             // pixel source: could also be video or image
    )
    
    // draw triangles using the array buffer from index 0 to 6 (6 is count)
    gl.drawArrays( gl.TRIANGLES, 0, 6 )
  }
  
  // update time on CPU and GPU
  time++
  gl.uniform1f( uTime, time )
}
```

Note that we only call `gl.drawArrays` if our texture has loaded, otherwise the screen will be blank. As discussed previously, we are also calling `gl.texImage2D` every frame to upload the current frame of webcam video to the GPU.

OK, you should have live video! No changes to our shaders are required. 
