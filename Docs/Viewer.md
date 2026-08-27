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

When layout depends on the available window size, pass a drawing function.
Viewer calls it initially and after each resize, producing retained vector
commands while allowing the producer to recompute its layout:

```sx
func responsive(width:int, height:int) Canvas {
    return build_interface(width, height)
}

Viewer.show(responsive, Viewer.CanvasSettings(
    width:1180,
    height:760,
    title:"Responsive interface"
))
```

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

Viewer forwards frame deltas and the window's input events to the session. A
`true` result from `update` or `handle` requests a new retained Canvas drawing.
Returning `false` keeps an idle session from rebuilding its vector content. It
also starts and stops OS text input from `accepts_text_input`. The session owns
its state; Viewer does not insert that state into an application's game ECS
world.

`Viewer.CanvasSession` remains available as a compatibility alias for
`Canvas.Session`. New reusable producers should name the Canvas-owned contract
so they do not describe themselves in terms of one presenter.

The positional `Viewer.show(drawing, width, height, title)` form remains the
shortest path for examples and generated plots.

All forms block until their window closes. Application plugins and resources
used to implement the viewer are package-private; callers only choose what to
show and how it should be presented.

Future content kinds such as models, GLB assets, animated sprites, galleries,
and slideshows will add typed settings for their own presentation choices
without turning the common API into an application-building interface.
