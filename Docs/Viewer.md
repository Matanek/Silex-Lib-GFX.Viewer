# Display visual values

`GFX.Viewer` opens a native window for visual values produced by Silex
programs. It owns the presentation application and hides its Window, GPU,
Rendering, Scene2D, ECS, and shader plumbing.

Display an image with its natural size:

```sx
use GFX.Viewer

Viewer.show(image)
Viewer.show(image, Viewer.ImageSettings(title:"Generated image"))
```

`ImageSettings` controls the window, background, and texture sampling when the
short form is not enough:

```sx
Viewer.show(image, Viewer.ImageSettings(
    title:"Nearest-neighbour preview",
    width:960,
    height:640,
    smooth:false
))
```

Small images remain centered at their native resolution. Larger images scale
down while preserving their aspect ratio.

A Canvas remains vectorial and follows the window viewport:

```sx
use GFX.Canvas
use GFX.Viewer

var drawing = Canvas()
drawing.paint(func(painter:&Canvas.Painter) {
    // Paint shapes and text.
})

Viewer.show(drawing, Viewer.CanvasSettings(
    width:960,
    height:640,
    title:"Generated chart"
))
```

The positional `Viewer.show(drawing, width, height, title)` form remains the
shortest path for examples and generated plots.

Both forms block until their window closes. Application plugins and resources
used to implement the viewer are package-private; callers only choose what to
show and how it should be presented.

Future content kinds such as models, GLB assets, animated sprites, galleries,
and slideshows will add typed settings for their own presentation choices
without turning the common API into an application-building interface.
