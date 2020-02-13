# Noise + Modular GLSL with glslify

In this lecture we'll learn how to modularize our shader code, and perhaps make our webgl code a little easier to read as well. To do this, we'll be using node.js and the node package manager (aka npm) to install libraries import shader functionality into our sketches. We'll also look at using an simple abstraction to replace our fairly complex WebGL code, and create a server that will automatically reload our webpages whenever we make changes to our shaders our JavaScript files.

## Installing node.js / npm

If you haven't already installed it, [install node.js](http://nodejs.org/). This will install NPM by default. Once node is installed, you should be able to run `node -v` and `npm -v` to verify the versions of node/npm that were installed.

## Installing browserify

Browserify is a library that resolves JS file dependencies by concatenating all linked JS files into one single file, while taking care to ensure that potential problems like circular dependencies are correctly resolved. With a little bit of extra wrapping (glslify) we can instead used browserify to conctenate all of our GLSL code into a single file as well, in addition to importing other glsl functions.

To install browserify globably on your computer, we'll use npm:

`npm install -g browserify`

You can check the version installed by running: `browserify --version`.

## Installing glslify

glslify is a *transform* of browserify, basically an extra set of instructions to the browserify program that customizes it for use with shaders. We can either install it to use when we call the `browserify` application, or we can install it globally to slightly simplify the command line invocation. The first option is simply `npm install glslify`, which will install the transform in the current project directory for use with browserify. The second option, `npm install -g glslify` will create a `glslify` command that we can invoked from the command line interface.

Go ahead and run the first version now, as we'll be using `glslify` as a `browserify` transform.

## Basic setup 

OK, with that out of the way, let's start with [our basic webgl template](./webgl_template.html) and make some changes to get glslify involved. The main change is going to be using separating out our JavaScript and shaders from our html.

1. First, take all of our JavaScript and place it into a single file, we'll call it `main.js`. 
2. Next take the fragment shader code out of the `<script>` tag in our html and place it in a separate file named `frag.glsl`. Delete the `<script>` tag after doing this.
3. Do the same thing with our vertex shader and name the file `vert.glsl`.
4. We want to import our `main.js` file, but first we need to run it through `browserify` in order to load our shaders. By convention, we will call the output `bundle.js`, to reflect that `browserify` is bundling many files into one. Go ahead and add a `<script>` tag to import this: `<script src='./bundle.js'></script>`
5. Next we need to load our shaders into our `main.js` file, to do this we need to import the `glslify` function. We can do this with `require()` function. Add the following line to the top of you `./main.js`:

`const glslify = require( 'glslify' )`

6. Now we need to replace our previous commmands to load our shader. Do a quick search for `shaderSource` in our `main.js` file and remove any code referencing our old `<script>` elements. Instead, we'll use the `glslify.file()` function to import a file. For example, for our vertex shader use:
  
```js
  let shaderSource = glslify.file( './vert.glsl' ) 
```

... and for our fragment shader a bit further down the page:

```js
  shaderSource = glslify.file( './frag.glsl' )
```

The file paths won't resolve correctly without the `./` in front of their names, so make sure you include them.

7. Last but not least, let's run `browserify` on our main.js file. Remember that we want to output a file named `bundle.js`, and we want to include our glslify transform. On the command line run:

`browserify -t glslify main.js -o bundle.js`

You should now have a file named `bundle.js` in your project directory. If you open it up, you can see that your shader files have been inlined where the `shaderSource` variables are defined. And if you open your HTML page in the browser, you should now see your shader correctly running. We have now taken the first step towards modularizing our GLSL.

## Bring in the noise
We can use glslify to easily import a wide variety of glsl functions that we can then call in our shaders; you can browse many of these in the [stack.gl documentation](http://stack.gl/packages/) under "Shader Components" in the menu on the left.

We're going to bring in some 3D simplex noise, where we'll use time as our third dimension (in addition to our x/y frag coordinates). First we need to install the `glsl-noise` module from the command line:

`npm i glsl-noise`

Next we'll replace our `frag.glsl` with the following code:

```c
#ifdef GL_ES
precision mediump float;
#endif

// below is the line that imports our noise function
#pragma glslify: snoise3 = require(glsl-noise/simplex/3d)

uniform float time;
uniform vec2 resolution;

void main() {
  vec2 uv = gl_FragCoord.xy / resolution;
  float noise = snoise3( vec3(uv.x*100., uv.y*100., time/50.) );
  
  gl_FragColor = vec4( noise, noise, noise, 1. );
}
```

Run our same browserify command again:

`browserify -t glslify main.js -o bundle.js`

... and now you should have time-varying 2D simplex noise if you open your HTML file. You can easily change this to 3D Worley noise (cellular noise) just by switch the glslify command in `frag.glsl` to:

```

## ASCII

Just for fun... let's add another function that converts an input texture to ASCII. We'll need to lower the resolution of our simplex noise to get this to work well.

```c
#ifdef GL_ES
precision mediump float;
#endif

#pragma glslify: snoiform float time;
uniform vec2 resolution;

void main() {
  vec2 uv = gl_FragCoord.xy / resolution;
  float noise = snoise3( vec3(uv.x*5., uv.y*5., time/150.) )3(noise), uv );
  gl_FragColor = vec4( asc ascse3 = require(glsl-noise/simplex/3d)
ut,asc
ut = asc
ut,asc
re(glsl-ascepragma glslify: asc
ut, 1. );
}
```

## Bye-bye WebGL

So, as you have seen in this course, it turns out creating a fullscreen fragment shader to run simulations and create various types of other effects is a pretty common task. `gl-toy` abstracts away all of the WebGL / OpenGL ES we've been using to create out fullscreen fragment shader... for example, all the shader program compilation, generating the vertex array buffer, defining our uniforms etc. We can keep our last fragment shader, but replace all the code in `main.js` with the following:

```js
const glslify = require( 'glslify' )
const toy     = require( 'gl-toy' )

const shader = glslify( './frag.glsl' )

let count = 0
toy( shader, (gl, shader) => {
  // this function runs once per frame
  shader.uniforms.resolution = [ gl.drawingBufferWidth, gl.drawingBufferHeight ]
  shader.uniforms.time = count++
})
```

You also need to run `npm install gl-toy`, and then re-run `browserify` with the glslify transform enabled. 

## Live shader reloading
We can use `budo` to enable live reloading of our shader. `budo` will start small server delivering our files, and also rerun browserify whenever we make changes to our file and automatically reload the results in any page accessing our server. We can install budo with `npm i budo`. We can then run it with `npx budo main.js --live -- -t glslify`. The `--live` option gets us the live reloading while the `-t glslify` adds the glslify transform to recompilation.

Once you run the server it should tell you what port it launched on and provide a URL. 
