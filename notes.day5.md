# Video Feedback
This class we’ll discuss video feedback, first looking at feedback more generally as an artistic concept, and then discussing it’s technical implementations via multiple textures and frame buffer objects (fbos).

James Crutchfield describes video feedback both practically and mathematically (I suppose that can be practical to) in his paper [Space-Time Dynamics in Video Feedback](https://pdfs.semanticscholar.org/1937/e742405499bade7501c04682575a44a72ed9.pdf)., where he discusses the relationship of video feedback to models / simulations that we’ll be exploring next week, such as cellular automata and reaction diffusion. Video feedback can simply be achieved by taking a camera, pointing it at a monitor, and then using the camera as an input to the monitor, so that a feedback loop is created. Putting a synthesizer like the Vidiot in the middle of the loop enables all sorts of interesting transformations and dynamic control elements. 

Many video artists have experimented with video feedback. Some excellent  examples can be found at [this page on the subject](https://softology.com.au/videofeedback/videofeedback.htm), but many other pioneering artists, such as Nam Junk Paik and Steina & Woody Vasulka were active experimenters.

## Creating Video Feedback in GLSL / WebGL
This tutorial assumes that we start [with a file capturing where we left of last class](./video_to_texture_invert.html), with video being copied to a texture that we could then manipulate on the GPU.  We’ll build from there.

The basic process is:
1. Create an additional **frame buffer object** (FBO)  to render to, in addition to the default FBO that WebGL provides.  
2. Create two additional texture objects to read and write from/to. We can't read and write to the same texture from a single shader, so instead we'll pingpong between reading and writing to two different textures.  
3. Create a fragment shader / shader program that generates video feedback and writes it to our extra FBO.  
4. Create an additional fragment shader that simply copies the contents of our extra FBO to the screen.  

### Shader Setup
1. We’ll keep our original shader that we used from last class, but we’ll also need to define a new shader program. This new shader program will use a different fragment shader, but the same vertex shader that simply passes the vertices through the shader without modification. Place the following code immediately after creating our first shader in the `window.onload` method:  

```js
shaderScript = document.getElementById('feedback')
shaderSource = shaderScript.text
const feedbackFragmentShader = gl.createShader( gl.FRAGMENT_SHADER )
gl.shaderSource( feedbackFragmentShader, shaderSource )
gl.compileShader( feedbackFragmentShader )
console.log( gl.getShaderInfoLog( feedbackFragmentShader ) )

// create feedback shader program
feedbackProgram = gl.createProgram()
gl.attachShader( feedbackProgram, vertexShader )
gl.attachShader( feedbackProgram, feedbackFragmentShader )

gl.linkProgram( feedbackProgram )
gl.useProgram( feedbackProgram )

uRes = gl.getUniformLocation( feedbackProgram, 'resolution' )
gl.uniform2f( uRes, gl.drawingBufferWidth, gl.drawingBufferHeight )

uFeedback = gl.getUniformLocation( feedbackProgram, 'feedbackAmount' )
gl.uniform1f( uFeedback, .925 )

// find a pointer to the uniform "time" in our fragment shader
uTime = gl.getUniformLocation( feedbackProgram, 'time' )

// one texture for feedback, one for video. There will actually be
// a third texture involved, but we'll only need to access two in our
// feedback shader in any given frame of video.
uFeedbackTexture = gl.getUniformLocation( feedbackProgram, 'feedbackTexture' )
uVideoTexture = gl.getUniformLocation( feedbackProgram, 'videoTexture' )

position = gl.getAttribLocation( feedbackProgram, 'a_position' )
gl.enableVertexAttribArray( feedbackProgram )
gl.vertexAttribPointer( position, 2, gl.FLOAT, false, 0,0 )
```

2. Next we need to create another `<script>` tag for our extra fragment shader. We'll give it the `id` of `feedback`.  
  
```html
<script id='feedback' type='x-shader/x-fragment'>
  #ifdef GL_ES
  precision mediump float;
  #endif

  uniform float time;
  uniform float feedbackAmount;
  uniform vec2  resolution;
  
  // for our live video feed
  uniform sampler2D videoTexture;
  // get access to the last frame of video
  uniform sampler2D feedbackTexture;

  void main() {
    vec2 pos = gl_FragCoord.xy / resolution;
    vec3 video = texture2D( videoTexture, pos ).rgb;
    vec3  = texture2D( feedbackTexture, pos ).rgb;
    
    // our final output is a combination of the live video signal
    // and our feedback
    gl_FragColor = vec4( (video * .125 + prior * feedbackAmount), 1. );
  }
</script>
```  

3. We'll take our previous fragment shader, which has an `id` attribute of `fragment`, and greatly simplify it. All this shader needs to do is copy data from a texture to its output. Of course, you could also perform additional manipulations either here or in the feedback shader... invert it, blur it, etc.  

```html
<script id='render' type='x-shader/x-fragment'>
  #ifdef GL_ES
  precision mediump float;
  #endif

  uniform sampler2D uSampler;
  uniform vec2 resolution;

  void main() {
    // copy color info from texture
    gl_FragColor = vec4( texture2D( uSampler, gl_FragCoord.xy / resolution ).rgb, 1. );
  }
</script>
```

OK, that's most of our shader work! The only remaining task will be to send uniforms to the GPU representing our textures, which we'll do in our `render()` function a bit later.

### Texture + Framebuffer Setup

We'll need two additional textures to read and write from in our render loop. We will be *reading* the last frame of video + feedback from one texture, and then *writing* the result of adding this to the current frame of video to a different texture. It's not possible to both read and write to the same texture in one frame / shader pass, so we'll have to do some pingponging in our render loop to switch between these textures.

We'll also create a framebuffer object (FBO) here. FBOs are the primary render targets for OpenGL / WebGL. In fact, we've been using one all along that WebGL sets up for us by default to render to... this is the FBO that gets displayed on screen. Now we need an additional FBO for offscreen rendering, so that we can keep each frame of video to use it in the next frame.

Add the following code to our `makeTextures()` function. It's basically running the same code we used to create our live video texture two more times, with an additional line to create our framebuffer.

```js
textureBack = gl.createTexture()
gl.bindTexture( gl.TEXTURE_2D, textureBack )
gl.texParameteri( gl.TEXTURE_2D, gl.TEXTURE_WRAP_S, gl.CLAMP_TO_EDGE )
gl.texParameteri( gl.TEXTURE_2D, gl.TEXTURE_WRAP_T, gl.CLAMP_TO_EDGE )
gl.texParameteri( gl.TEXTURE_2D, gl.TEXTURE_MIN_FILTER, gl.LINEAR )
gl.texParameteri( gl.TEXTURE_2D, gl.TEXTURE_MAG_FILTER, gl.LINEAR )
gl.texImage2D( gl.TEXTURE_2D, 0, gl.RGBA, size, size, 0, gl.RGBA, gl.UNSIGNED_BYTE, null )

textureFront = gl.createTexture()
gl.bindTexture( gl.TEXTURE_2D, textureFront )
gl.texParameteri( gl.TEXTURE_2D, gl.TEXTURE_WRAP_S, gl.CLAMP_TO_EDGE )
gl.texParameteri( gl.TEXTURE_2D, gl.TEXTURE_WRAP_T, gl.CLAMP_TO_EDGE )
gl.texParameteri( gl.TEXTURE_2D, gl.TEXTURE_MIN_FILTER, gl.LINEAR )
gl.texParameteri( gl.TEXTURE_2D, gl.TEXTURE_MAG_FILTER, gl.LINEAR )
gl.texImage2D( gl.TEXTURE_2D, 0, gl.RGBA, size, size, 0, gl.RGBA, gl.UNSIGNED_BYTE, null )

// Create a framebuffer and attach the texture.
framebuffer = gl.createFramebuffer()
```

### Render loop
OK, this part is a bit tricky. Here's how this goes:

1. First, we render to our offscreen framebuffer, which will read from texture A and write to texture B.  
2. Next, we need to copy our live video stream to our video texture (let's call this texture V).  
3. With our offscreen frame buffer bound, we call `gl.drawArrays()` to render to the FBO.  
4. Now we want to read from the texture we just wrote to, so we swap the pointers for the read / write textures.  
5. We bind our default framebuffer, so that the next time we draw it will render to our screen.  
6. We switch the active shader to be the one that copies our texture to the screen.  
7. We call `gl.drawArrays()` to render to screen.  

This is complicated! However, it's the basis for most of the simulations we'll be doing in this course for the remainder of the term, so take some time to look at the code below and make sure it makes sense... and then ask questions until it does.

```js
let time = 0
function render() {
  // schedules render to be called the next time the video card requests 
  // a frame of video
  window.requestAnimationFrame( render )
  
  if( textureLoaded === true ) { 
    // use our feedback shader
    gl.useProgram( feedbackProgram )  
    // update time on CPU and GPU
    time++
    gl.uniform1f( uTime, time )     
    gl.bindFramebuffer( gl.FRAMEBUFFER, framebuffer )
    // use the framebuffer to write to our texFront texture
    gl.framebufferTexture2D( gl.FRAMEBUFFER, gl.COLOR_ATTACHMENT0, gl.TEXTURE_2D, textureFront, 0 )
    // this defines the size of the data that will be drawn onto our texture
    gl.viewport(0, 0, size,size )
    
    gl.activeTexture( gl.TEXTURE0 )
    gl.bindTexture( gl.TEXTURE_2D, videoTexture )
    gl.uniform1i( uVideoTexture, 0 )
    gl.texImage2D( 
      gl.TEXTURE_2D,    // target: you will always want gl.TEXTURE_2D
      0,                // level of detail: 0 is the base
      gl.RGBA, gl.RGBA, // color formats
      gl.UNSIGNED_BYTE, // type: the type of texture data; 0-255
      video             // pixel source: could also be video or image
    )

    // in our shaders, read from texBack, which is where we poked to
    gl.activeTexture( gl.TEXTURE1 )
    gl.bindTexture( gl.TEXTURE_2D, textureBack )
    gl.uniform1i( uFeedbackTexture, 1 )
    // run shader
    gl.drawArrays( gl.TRIANGLES, 0, 6 )

    // swap our front and back textures
    let tmp = textureFront
    textureFront = textureBack
    textureBack = tmp

    // use the default framebuffer object by passing null
    gl.bindFramebuffer( gl.FRAMEBUFFER, null )
    gl.viewport(0, 0, size, size )
    // select the texture we would like to draw to the screen.
    // note that webgl does not allow you to write to / read from the
    // same texture in a single render pass. Because of the swap, we're
    // displaying the state of our simulation ****before**** this render pass (frame)
    gl.activeTexture( gl.TEXTURE0 )
    gl.bindTexture( gl.TEXTURE_2D, textureFront )
    // use our drawing (copy) shader
    gl.useProgram( drawProgram )
    // put simulation on screen
    gl.drawArrays( gl.TRIANGLES, 0, 6 )
  }
}
```

OK, that should do it... hopefully if you test this now you'll have some feedback to play with. Experiment with different values for the `feedbackAmount` uniform mixed in with the live video signals. One final thing, we made a bunch of global variables in our functions (most of our uniforms etc.) Add the following declarattions to the top of your `<script>` tag to clean things up:
  
```js
let feedbackProgram, 
    uTime, uVideoTexture, uFeedbackTexture, uFeedback,
    videoTexture, textureBack, textureFront, textureLoaded,
    video
```