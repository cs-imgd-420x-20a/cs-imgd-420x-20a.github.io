
# Running Simulations on the GPU & Multi-pass Rendering
In this assignment we will learn how to run the reaction diffusion model on the graphics card of our computers, opening it up to parallel processing that will speed up performance and enable us to run multiple iterations of the equation per frame of video, even while running the simulation at full (or close to full) resolution. The big difference in this simulation vs. our previous game of life simulation is that we will do multipass rendering, which allows us to run our simulation multiple times per frame of video, which *greatly* speeds up simulations. Reaction diffusion is notoriously slow to run, so it really only becomes interesting for realtime interaction once you’ve implemented it on the GPU with multipass rendering.  

Place the assignment on your course website when complete. Since I am away this week, you are welcome to seek help from your classmates on this assignment. 

### Resources:
• Here’s the primary reference we’ll use for the RD equation:
Karl Sims tutorial on reaction-diffusion:  [https://karlsims.com/rd.html](https://karlsims.com/rd.html) 

• This is a fantastic video tutorial that covers RD. Although it’s done on the CPU using p5.js (hah! look how slow it runs!) the instructor does a great job explaining the RD equation: 
Coding Train Reaction Diffusion Tutorial -  [https://www.youtube.com/watch?v=BV9ny785UNc](https://www.youtube.com/watch?v=BV9ny785UNc) 

• One cool example of RD in action:
DIFFUSION -  [https://vimeo.com/145251635](https://vimeo.com/145251635) 

• Alan Turing’s classic paper on the subject:
The Chemical Basis of Morphogenesis:  [http://www.dna.caltech.edu/courses/cs191/paperscs191/turing.pdf](http://www.dna.caltech.edu/courses/cs191/paperscs191/turing.pdf) 

## Create our starter HTML file.
```html
<!doctype html> 
<html lang='en'> 
<head> 
  <style> 
    body{ 
      margin:0;
      background:black; 
    } 
    canvas{ 
      width:100%; height:100%; position:relative; 
    } 
  </style> 
</head>

<body>
  <canvas></canvas>
</body>
<script>

</script>
</html>
```

## Setup our canvas and define an array buffer to hold our quad
This is similar to our last assignment. We want to draw a fullscreen quad to the screen, via two triangles. We’ll pass the vertices for our triangles in pairs. Remember in the OpenGL coordinate systems coordinates are measured from {-1,1}. After defining our vertices, we then create vertex array buffer that we can use to pass all of our vertices up to the graphics card. 

We’ll also define two sizes to use. The first, /stateSize/, will determine the width and the height of our reaction-diffusion simulation. Because we’ll be reading and writing state for the simulation into OpenGL textures, the size we chooses needs to be a power of two ( 2,4,8,16,32,64,128 etc.) The equation given below for the stateSize variable guarantees that. Our /renderSize/variable will determine the size we display our simulation output at; by default this will fill the width of the screen, stretching the simulation output as needed.

Place the code below in the `<script>` tag.

```js
window.onload = function() { 
    let canvas = document.querySelector( 'canvas' ) 
    canvas.width = window.innerWidth
    canvas.height = window.innerHeight 
    let gl = canvas.getContext( 'webgl' ) 
    let stateSize = Math.pow( 2, Math.floor(Math.log(canvas.width)/Math.log(2)) )
    
    let verts = [ 
      1, 1, 
      -1, 1, 
      -1,-1, 
      1, 1, 
      -1, -1, 
      1, -1, 
    ]

    let vertBuffer = gl.createBuffer() 
    gl.bindBuffer( gl.ARRAY_BUFFER, vertBuffer ) 
    gl.bufferData( gl.ARRAY_BUFFER, new Float32Array(verts), gl.STATIC_DRAW ) 
    gl.vertexAttribPointer( 0, 2, gl.FLOAT, false, 0, 0 ) 
    gl.enableVertexAttribArray( 0 )
}
```

## Add script tags to hold the three shaders we will use.
Script tags are a convenient way to use shaders. We could also store the shaders as strings in separate JavaScript files, or dynamically load the files using fetch(). These script tags can be placed above our below the script tag containing the bulk of our JavaScript.

Our first shader is our vertex shader. In many ways this is the “defaut” vertex shader, that simply accepts an input vertex and then outputs the result. These vertices are then used to determine the position of each pixel every time our fragment shader runs. We’ll also add a note to the compiler that our shader will use floats with medium level precision.

Our second shader will draw our simulation to the screen, inverting the colors along the way for good measure.

Our final shader will run our simulation. This will look very familiar to you from our previous assignments running the shader on the CPU. We’ll include a function, `get()`, that will let us lookup pixel values with a given offset from our current location. We can use this to lookup the neighbors to our current cell so can perform our Laplace transforms. These lookups are done using OpenGL textures, using the `texture2D()` function, which enables us to read a pixel from an OpenGL `sampler2D` uniform (note: uniforms in GLSL are variables that are passed from the CPU up to the GPU; they can be changed interactively on the CPU as the shader runs and the changes can then update the running shader).

```html
<script id="vshader" type="whatever"> 
  precision mediump float; 
  attribute vec2 a_position; 
  void main() { 
    gl_Position = vec4( a_position, 0, 1.0); 
  } 
</script>

<script id="fshader_draw" type="whatever"> 
  precision mediump float;
  uniform sampler2D state; 
  uniform vec2 scale; 
  
  void main() { 
    vec4 color = texture2D(state, gl_FragCoord.xy / scale); 
    gl_FragColor = vec4( 1.-color.x, 1.-color.x, 1.-color.x, 1. ); 
  }
</script>

<script id="fshader_render" type="whatever"> 
  precision mediump float;
  uniform sampler2D state; 
  uniform vec2 scale;
  //float f=.0545, k=.062, dA = 1., dB = 0.; // coral preset 
  float f = .0457, k = .0635, dA = 1., dB = .5;
  
  vec2 get(int x, int y) { 
    return texture2D( state, ( gl_FragCoord.xy + vec2(x, y) ) / scale ).rg; 
  } 
  
  vec2 run() { 
    vec2 state = get( 0, 0 ); 
    float a = state.r; 
    float b = state.g; 
    float sumA = a * -1.; 
    float sumB = b * -1.; 
    
    sumA += get(-1,0).r * .2; 
    sumA += get(-1,-1).r * .05; 
    sumA += get(0,-1).r * .2; 
    sumA += get(1,-1).r * .05; 
    sumA += get(1,0).r * .2; 
    sumA += get(1,1).r * .05; 
    sumA += get(0,1).r * .2; 
    sumA += get(-1,1).r * .05;
    
    sumB += get(-1,0).g * .2; 
    sumB += get(-1,-1).g * .05; 
    sumB += get(0,-1).g * .2; 
    sumB += get(1,-1).g * .05; 
    sumB += get(1,0).g * .2; 
    sumB += get(1,1).g * .05; 
    sumB += get(0,1).g * .2; 
    sumB += get(-1,1).g * .05; 
    
    state.r = a + dA 
      * sumA - 
      a * b * b + 
      f * (1. - a); 
      
    state.g = b + dB * 
      sumB + 
      a * b * b - 
      ((k+f) * b);
      
    return state; 
  } 
  void main() { 
    vec2 nextState = run(); 
    gl_FragColor = vec4( nextState.r, nextState.g, 0., 1. ); 
  } 
</script>
```

## Compile and link our shaders to use.
Here we grab our script tags and the text inside each of them, compile fragment and vertex shaders, and then link them into /programs/ that we can alternate between using. We’ll create two programs: one to run our shader and one to render it to the screen. Both will use the same vertex shader but paired with different fragment shaders. We’ll add some `console.log()` statements to make sure everything compiles OK. We’ll also define a uniform, `scale`to pass our simulation size up to the shader, and define an attribute, `a_position` to reference our vertex positions. The code below should go in the end of our `window.onload` function.

```js
    let shaderScript = document.getElementById( 'vshader' )
    let shaderSource = shaderScript.text 
    const vertexShader = gl.createShader( gl.VERTEX_SHADER ) 
    gl.shaderSource( vertexShader, shaderSource ) 
    gl.compileShader( vertexShader )
    console.log( gl.getShaderInfoLog( vertexShader ) ) // create fragment shader to run our simulation
    
    shaderScript = document.getElementById( 'fshader_render' ) 
    shaderSource = shaderScript.text 
    const fragmentShaderRender = gl.createShader( gl.FRAGMENT_SHADER ) 
    gl.shaderSource( fragmentShaderRender, shaderSource ) 
    gl.compileShader( fragmentShaderRender ) 
    console.log( gl.getShaderInfoLog( fragmentShaderRender ) ) // create shader program const
      
    programRender = gl.createProgram() 
    gl.attachShader( programRender, vertexShader ) 
    gl.attachShader( programRender, fragmentShaderRender )
    gl.linkProgram( programRender )
    gl.useProgram( programRender )
    
    // create pointer to vertex array and uniform sharing simulation size 
    const position = gl.getAttribLocation( programRender, 'a_position' )
    gl.enableVertexAttribArray( position ) 
    gl.vertexAttribPointer( position, 2, gl.FLOAT, false, 0,0 ) 
    let scale = gl.getUniformLocation( programRender, 'scale' ) 
    gl.uniform2f( scale, stateSize, stateSize )
      
    // create shader program to draw our simulation to the screen 
    shaderScript = document.getElementById( 'fshader_draw' ) 
    shaderSource = shaderScript.text 
    fragmentShaderDraw = gl.createShader( gl.FRAGMENT_SHADER ) 
    gl.shaderSource( fragmentShaderDraw, shaderSource )
    gl.compileShader( fragmentShaderDraw ) 
    console.log( gl.getShaderInfoLog( fragmentShaderDraw ) ) 
      
    // create shader program
    programDraw = gl.createProgram() 
    gl.attachShader( programDraw, vertexShader ) 
    gl.attachShader( programDraw, fragmentShaderDraw ) 
    gl.linkProgram( programDraw )
    gl.useProgram( programDraw )

    scale = gl.getUniformLocation( programDraw, 'scale' ) 
    gl.uniform2f( scale, canvas.width,canvas.height ) 
    const position2 = gl.getAttribLocation( programDraw, 'a_position' ) 
    gl.enableVertexAttribArray( position2 )
    gl.vertexAttribPointer( position2, 2, gl.FLOAT, false, 0,0 )
```

## Create our textures and fill them with the initial values for our simulation.
We’ll use OpenGL textures to read and write our simulation state. One limitation of OpenGL is that you can’t read from a texture and write to it using the same shader, so we’ll create two different textures to bounce between. You can think of this as being similar to using two different JS arrays to store our state in from previous simulations we’ve done, just swapping between the two as we go along.

We’ll then populate our textures using a function named /reset(),/which we can call later if we want to interactively reset the contents of the simulation. Our reset function will fill the textures using the gl.texSubImage2D function, which lets us place an array of values into a texture at a given offset (here we’ll use {0,0} for the the offset.

```js
    // enable floating point textures in the browser
    gl.getExtension('OES_texture_float'); 
    
    let texFront = gl.createTexture() 
    gl.bindTexture( gl.TEXTURE_2D, texFront ) 
    gl.texParameteri( gl.TEXTURE_2D, gl.TEXTURE_WRAP_S, gl.REPEAT ) 
    gl.texParameteri( gl.TEXTURE_2D, gl.TEXTURE_WRAP_T, gl.REPEAT ) 
    gl.texParameteri( gl.TEXTURE_2D, gl.TEXTURE_MIN_FILTER, gl.NEAREST )
    gl.texParameteri( gl.TEXTURE_2D, gl.TEXTURE_MAG_FILTER, gl.NEAREST ) 
    gl.texImage2D( gl.TEXTURE_2D, 0, gl.RGBA, stateSize, stateSize, 0, gl.RGBA, gl.FLOAT, null ) 
    
    let texBack = gl.createTexture() 
    gl.bindTexture( gl.TEXTURE_2D, texBack ) 
    gl.texParameteri( gl.TEXTURE_2D, gl.TEXTURE_WRAP_S, gl.REPEAT ) 
    gl.texParameteri( gl.TEXTURE_2D, gl.TEXTURE_WRAP_T, gl.REPEAT ) 
    gl.texParameteri( gl.TEXTURE_2D, gl.TEXTURE_MIN_FILTER, gl.NEAREST ) 
    gl.texParameteri( gl.TEXTURE_2D, gl.TEXTURE_MAG_FILTER, gl.NEAREST ) 
    gl.texImage2D( gl.TEXTURE_2D, 0, gl.RGBA, stateSize, stateSize, 0, gl.RGBA, gl.FLOAT, null )

    const pixelSize = 4 
    const feedSize = 48
    const initState = new Float32Array( stateSize * stateSize * pixelSize ) 
    const reset = function() { 
      for( let i = 0; i < stateSize; i++ ) { 
          for( let j = 0; j < stateSize * pixelSize; j+= pixelSize ) { 
          // this will be our 'a' value in the simulation 
          initState[ i * stateSize * pixelSize + j ] = 1 
          // selectively add 'b' value to middle of screen 
          if( i > stateSize / 2 - stateSize / feedSize  && i < stateSize / 2 + stateSize / feedSize ) { 
            const xmin = j > (stateSize*pixelSize) / 2 - stateSize / feedSize
            const xmax = j < (stateSize*pixelSize) / 2 + (stateSize*pixelSize) / feedSize 
            if( xmin && xmax ) { 
              initState[ i * stateSize * pixelSize + j + 1 ] = 1 
            } 
          } 
        } 
      } 
      
      gl.texSubImage2D( 
        gl.TEXTURE_2D, 0, 0, 0, stateSize, stateSize, gl.RGBA, gl.FLOAT, initState, 0 
      ) 
    }
    
    reset()
```

## Create our framebuffers and a function to bounce between them.
OpenGL renders to /framebuffer/objects; there’s a default one enabled whenever you create an OpenGL scene. Here we create a function that bounces back and forth between two frame buffer objects, one that runs a shader reading one texture and writing the result to another texture (remember: you can’t read and write to the same texture in a single shader pass). By assigning our framebuffer object to a texture, we can draw to that texture instead of to the screen (which is the default behavior if the framebuffer is not assigned to a texture).

```js
    const fb = gl.createFramebuffer() 
    const fb2 = gl.createFramebuffer() 
  
    const pingpong = function() {
      gl.bindFramebuffer( gl.FRAMEBUFFER, fb ) 
      // use the framebuffer to write to our texFront texture
      gl.framebufferTexture2D( gl.FRAMEBUFFER, gl.COLOR_ATTACHMENT0, gl.TEXTURE_2D, texFront, 0 ) 
      // set viewport to be the size of our state (reaction diffusion simulation) 
      // here, this represents the size that will be drawn onto our texture 
      gl.viewport(0, 0, stateSize, stateSize ) 
      // in our shaders, read from texBack, which is where we poked to 
      gl.bindTexture( gl.TEXTURE_2D, texBack ) // run shader 
      gl.drawArrays( gl.TRIANGLES, 0, 6 ) 
    
      gl.bindFramebuffer( gl.FRAMEBUFFER, fb2 )
      gl.framebufferTexture2D( gl.FRAMEBUFFER, gl.COLOR_ATTACHMENT0, gl.TEXTURE_2D, texBack, 0 ) 
      // set our viewport to be the size of our canvas 
      // so that it will fill it entirely 
      gl.viewport(0, 0, canvas.width, canvas.height )
      // select the texture we would like to draw the the screen. 
      // note that webgl does not allow you to write to / read from the 
      // same texture in a single render pass. Because of the swap, we're 
      // displaying the state of our simulation ****before**** this render pass (frame) 
      gl.bindTexture( gl.TEXTURE_2D, texFront ) 
      // put simulation on screen 
      gl.drawArrays( gl.TRIANGLES, 0, 6 ) 
    }
```

## (almost) Last but not least, draw!
Our draw function starts by telling OpenGL to use our /programRender/ shader, which runs our simulation. We then call the /pingpong/function as many times as we want (or as our graphics card supports) to do multipass rendering, running the reaction diffusion simulation multiple times per frame of video. After we’ve run the simulation, we remove our framebuffer binding so that we can draw to the screen again (as opposed to a texture). We then draw whatever is currently in our /texBack/texture, using it to fill the screen.

```js
const draw = function() { 
      gl.useProgram( programRender ) 
      for( let i = 0; i < 12; i++ ) pingpong()
 
      // use the default framebuffer object by passing null 
      gl.bindFramebuffer( gl.FRAMEBUFFER, null ) 
    
      // set our viewport to be the size of our canvas 
      // so that it will fill it entirely 
      gl.viewport(0, 0, canvas.width, canvas.height )
      // select the texture we would like to draw the the screen. 
      gl.bindTexture( gl.TEXTURE_2D, texBack ) 
      // use our drawing (copy) shader 
      gl.useProgram( programDraw ) 
      // put simulation on screen
      gl.drawArrays( gl.TRIANGLES, 0, 6 ) 
        
      window.requestAnimationFrame( draw ) 
    }
     
    draw()
```

## GUI time.
Using your own code, add a simple GUI that lets you control aspects of the simulation while it is running. I highly recommend using the dat.GUI project:

 [https://workshop.chromeexperiments.com/examples/gui/#1--Basic-Usage](https://workshop.chromeexperiments.com/examples/gui/#1--Basic-Usage) 

… but you can also use regular HTML widgets if you’d like. Remember, we can move values between the CPU and the GPU using /uniforms/. To define a uniform in a shader, simply declare it as follows:

`uniform float f;`

In JavaScript, you we could define and update the uniform value as follows:

```js
// get pointer to uniform in shader
let f = gl.getUniformLocation( programRender, ‘f’ )
// send value from CPU to GPU pointer location
gl.uniform1f( f, .0457 )
```

You want to have your renderProgram shader enabled when you send your uniforms up to the shader, so make sure you do this after the call to:

` gl.useProgram( programRender )`

… in our draw() method.
