gl-surface-plot
===============
Draws a surface plot

## Example

```javascript
var shell = require("gl-now")({ clearColor: [0,0,0,0] })
var camera = require("game-shell-orbit-camera")(shell)
var createSurfacePlot = require("gl-surface-plot")
var ndarray = require("ndarray")
var fill = require("ndarray-fill")
var diric = require("dirichlet")
var glm = require("gl-matrix")
var mat4 = glm.mat4

var surface

shell.on("gl-init", function() {
  var gl = shell.gl
  gl.enable(gl.DEPTH_TEST)

  //Set up camera
  camera.lookAt(
    [0, 0, 2],      //Eye position
    [256, 256, 64], //Eye target
    [0, 0, 1])      //Up direction

  //Create field
  var field = ndarray(new Float32Array(512*512), [512,512])
  fill(field, function(x,y) {
    return 128 * diric(10, 10.0*(x-256)/512) * diric(10, 10.0*(y-256)/512)
  })
  surface = createSurface(gl, field)
})

shell.on("gl-render", function() {
  surface.draw({
    view: camera.view(),
    projection:  mat4.perspective(new Array(16), Math.PI/4.0, shell.width/shell.height, 0.1, 10000.0)
  })
})
```

Here is what this should look like:

<img src="plot.png">

[Test it in your browser (requires WebGL)](http://mikolalysenko.github.io/gl-surface-plot/)

## Install

```
npm install gl-surface-plot
```

## API

```javascript
var createSurfacePlot = require("gl-surface-plot")
```

### `var surface = createSurfacePlot(gl, field[, params])`
Creates a surface plot object

* `gl` is a WebGL context
* `field` is a 2D ndarray
* `params` is an optional collection of arguments that contains any of the following:

    + `colormap` - the name of the color map to use for the surface (default "jet")
    + `pickId` is the picking id for the surface

**Returns** A surface object

### `surface.update(params)`
Updates the surface.  The parameter object may contain any of the following properties:

* `field` a new 2D field encoded as an ndarray
* `colormap` the name of the new color map for the surface
* `pickId` is the picking id for the surface
* `ticks` is a pair of arrays of ticks representing the spacing of the points for the axes of the surface
* `showSurface` if set, draw the surface
* `showContour` if set, draw contour lines
* `contourWidth` the width fo the contour lines
* `contourTint` the amount of tint of the contour lines
* `contourColor` the color of the contour line tint
* `levels` an array of arrays representing the level of the isolines.
* `dynamicWidth` the width of the dynamic isolines
* `dynamicColors` the color of the dynamic isolines
* `dynamicTint` the tint of the dynamic isolines

### `surface.draw([params])`
Draws the surface.  Accepts the following parameters

* `model` the 4x4 model matrix (in gl-matrix format)
* `view` the 4x4 view matrix
* `projection` the 4x4 projection matrix

### `surface.dispose()`
Destroys the surface, releases all associated WebGL resources

### `surface.bounds`
A pair of 3D arrays representing the lower/upper bounding box for the surface plot.

### `surface.clipBounds`

A pair of arrays which bound the coordinates of the surface plot in 3D.

## Interactivity/picking

### `surface.drawPick(camera)`
Draws the surface for point picking mode

### `surface.pick(selection)`
Test if the given selection is contained in the surface.  If true, returns an object encoding the selected point.

**Returns** An object encoding the selected point on the surface with the following properties:

* `position` which is the position of the selected point on the surface
* `index` a vector encoding the [x,y] index of the closest data point
* `uv` the uv coordinate of the selection
* `levels` the closest levels to the selection

### `surface.dynamic(levels)`
Toggles drawing level isolines and their projections.

* `levels` is an array of 3 arrays representing the x/y/z levels to draw

## Lighting parameters

You can also tweak the lighting parameters for the surface using the following variables

### `surface.lightPosition`
The position of the light source relative to the viewer in clip coordinates

### `surface.ambientLight`
The fraction of light which is ambiently lit

### `surface.diffuseLight`
The amount of diffuse light to apply to the surface

### `surface.specularLight`
THe amount of speculare light to apply to the surface

### `surface.roughness`
How rough the surface is  (must be between 0 and 1)

### `surface.fresnel`
The amount of rim lighting to apply.  Higher values = more intense rim light.

## License
MIT License.