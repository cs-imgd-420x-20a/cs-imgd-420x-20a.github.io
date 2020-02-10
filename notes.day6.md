# A brief history / description of Cellular Automata
  - A system of cell objects  
    - cells live on a grid  
    - each cell has a state  
    - each cell has a neighborhood  
    - cells follow rules over time  
      
  - Created by John Von Neumann  
    - created game theory  
    - worked on manhattan project  
    - Von Neumann architecture  
    - self-replicating automaton  
      - Theory of Self-Reproducing Automata, 1966  

## Examples of 1D automata
- [Rule 30](mathworld.wolfram.com/Rule30.html)  
    - [Cambridge(UK) Train Station using Rule 30](https://boingboing.net/2018/01/04/a-train-station-with-walls-des.html)  
- [Rule 110](mathworld.wolfram.com/Rule110.html)  
  - [Universality in Cellular Automata - Matthew Cook](http://wpmedia.wolfram.com/uploads/sites/13/2018/02/15-1-1.pdf)  
  
## Automata in the arts - Xenakis
- Iannis Xenakis  
  - [Horus](https://www.youtube.com/watch?v=Hf7sGbnsu2E)  
  - [Paper analyzing use of automata in Horus](http://cicm.mshparisnord.org/ColloqueXenakis/papers/Solomos.pdf)
  
## Conways Game of Life
  - [Wikipedia](https://en.wikipedia.org/wiki/Conway's_Game_of_Life)  
  - [1970 Scientific American article by Henry Gardner](https://www.ibiblio.org/lifepatterns/october1970.html), that popularized cellular automata  
  - Any live cell with fewer than two live neighbours dies, as if caused by under-population.  
  - Any live cell with two or three live neighbours lives on to the next generation.  
  - Any live cell with more than three live neighbours dies, as if by over-population.  
  - Any dead cell with exactly three live neighbours becomes a live cell, as if by reproduction.  
  
# Running the Game of Life on the GPU

This tutorial builds off of the [video feedback tutorial from last class](./notes.day5.md); I've pulled the basic concepts into a [template that we can start with](./gol_template.html) that removes the live video texturing and a bunch of associated code. It also breaks our rather long `window.onload` function into separate functions for creating the triangle we'll draw to, for creating our shader programs, and for creating our OpenGL texture objects. The template should run without errors when you open it in the browser, however, nothing will render to the screen. It's currently swapping textures just like our video feedback example, but the shaders aren't rendering anything to screen.

## Initializing State

Our first task is to create an initial state for our simulation. There are lots of different initial states that produce interesting results with the game of life, but for this tutorial we'll stick with a pure random state. Each cell can be either alive or dead, a value of 0 or a value of 1. We need a function that will enable us to write to a particular XY coordinate in a particular texture... it's given below:

```js
function poke( x, y, value, texture ) {   
  gl.bindTexture( gl.TEXTURE_2D, texture )
  
  // https://developer.mozilla.org/en-US/docs/Web/API/WebGLRenderingContext/texSubImage2D
  gl.texSubImage2D( 
    gl.TEXTURE_2D, 0, 
    // x offset, y offset, width, height
    x, y, 1, 1,
    gl.RGBA, gl.UNSIGNED_BYTE,
    // is supposed to be a typed array
    new Uint8Array([ value, value, value, 255 ])
  )
}
```

Now that we have this, we can loop through all the pixels in our texture and set them to a random value. The template includes a `dimensions` object that stores the width and height of the simulation.

```js
function setInitialState() {
  for( i = 0; i < dimensions.width; i++ ) {
    for( j = 0; j < dimensions.height; j++ ) {
      if( Math.random() > .75 ) {
        poke( i, j, 1, textureBack )
      }
    }
  }
}
```

Place a call to `setInitialState` after the call to `makeTextures` in our `window.onload` function. The textures must be initialized before calling the state.

## Editing our simulation shader

The `render()` function is already setup to swap between our textures, now all we have to do is code the simulation in our fragment shader and the simulation will run based on our initial starting condition. 

```glsl
#ifdef GL_ES
precision mediump float;
#endif

uniform float time;
uniform vec2 resolution;

// simulation texture state, swapped each frame
uniform sampler2D state;

// look up individual cell values 
int get(int x, int y) {
  return int( 
    texture2D( state, ( gl_FragCoord.xy + vec2(x, y) ) / resolution ).r 
  );
}

void main() {
  // get sum of all surrounding nine neighbors
  int sum = get(-1, -1) +
            get(-1,  0) +
            get(-1,  1) +
            get( 0, -1) +
            get( 0,  1) +
            get( 1, -1) +
            get( 1,  0) +
            get( 1,  1);
  
  if (sum == 3) {
    // ideal # of neighbors... if cell is living, stay alive, if it is dead, come to life!
    gl_FragColor = vec4( 1. );
  } else if (sum == 2) {
    // maintain current state
    float current = float( get(0, 0) );
    gl_FragColor = vec4( vec3( current ), 1.0 );
  } else {
    // over-population or lonliness... cell dies
    gl_FragColor = vec4( vec3( 0.0 ), 1.0 );
  }
}
```

... and that's basically it. In many ways, this is much less complicated than the video feedback example we looked at in the last class. Not having to deal with initializing the video feed and having an extra texture really cleans the code up.

Lots of things to explore:
- Right now our simulation is bounded... how to make it torroidal?  
- The resolution of our simulation is pretty high, which is cool, but also makes it hard to tell what's happening. How can we *decrease* the resolution, and slow the simulation down?  
- How can we define more interesting initial conditions than pure randomness?  
- What if we have more than two possible states?  
- What tricks could we use to make this more visually interesting?






