# Display a visual value with GFX.Viewer

`GFX.Viewer` opens a native window for a visual value produced by a Silex
program. It owns the presentation application and hides all Window, GPU,
Rendering, Scene2D, ECS, and shader composition.

[Lire cette documentation en français.](../FR/README.md)

## Install the package

```text
silex install GFX.Viewer
```

GFX.Viewer requires Silex 0.39.0 or newer.

## Display an image

After creating or loading an `image` value, display it at its natural size:

```sx
use GFX.Viewer

Viewer.show(image)
Viewer.show(image, Viewer.ImageSettings(title:"Generated image"))
```

This fragment assumes an existing compatible image. `ImageSettings` controls
the window, background, and texture sampling when the short form is not enough:

```sx
Viewer.show(image, Viewer.ImageSettings(
    title:"Nearest-neighbour preview",
    width:960,
    height:640,
    smooth:false,
))
```

Small images remain centered at their native resolution. Larger images scale
down while preserving their aspect ratio.

## Display a Canvas

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
    title:"Generated chart",
))
```

The positional `Viewer.show(drawing, width, height, title)` form remains the
shortest path for examples and generated plots.

When layout depends on the window, pass a drawing function. Viewer calls it at
startup and after every resize so the producer can recompute its layout:

```sx
func responsive(width:int, height:int) Canvas {
    return build_interface(width, height)
}

Viewer.show(responsive, Viewer.CanvasSettings(
    width:1180,
    height:760,
    title:"Responsive interface",
))
```

This fragment assumes an application-owned `build_interface` function.

## Present interactive content

Interactive Canvas content implements `Canvas.Session`:

```sx
class Preview:Canvas.Session {
    func drawing(width:int, height:int) Canvas { return build(width, height) }
    func update(delta:float) bool { return advance_animations(delta) }
    func handle(event:@GFX.Input.Event) bool { return update_state(event) }
    func accepts_text_input() bool { return editing_text() }
}

Viewer.show(Preview(), Viewer.CanvasSettings(width:1180, height:760))
```

The functions called in this fragment belong to the content. Viewer forwards
frame deltas and input events to the session. A `true` result from `update` or
`handle` requests a new retained drawing; `false` keeps idle content from being
rebuilt. `accepts_text_input` synchronizes system text input.

`Viewer.show_until(session, close_when, settings)` evaluates a predicate after
every update and closes its window when it returns `true`. An ordinary session
remains open until the user closes it.

`Viewer.CanvasSession` remains a compatibility alias for `Canvas.Session`. The
contract belongs to Canvas, not its presenter. All Viewer calls block until the
window closes; their Application plumbing remains private.
