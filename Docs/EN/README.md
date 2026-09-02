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
use STD.Math

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

`anchor` places the drawing origin relative to the viewport and `position`
adds a logical offset. A circle drawn around `(0, 0)` can therefore remain
unchanged and be centered in the window:

```sx
Viewer.show(drawing, Viewer.CanvasSettings(
    width:800,
    height:800,
    anchor:Math.Vec2(0.5),
))
```

The positional `Viewer.show(drawing, width, height, title)` form remains the
shortest path for examples and generated plots.

Text uses hinted coverage by default, which is suitable at small sizes. An
application that needs to control its representation explicitly selects the
mode in the Canvas settings:

```sx
Viewer.show(drawing, Viewer.CanvasSettings(
    text_mode:Viewer.CanvasTextMode.vector
))
```

`coverage` preserves raster coverage and `coverage_density` controls its
additional density; `vector` requires an outline for every non-empty glyph and
retains scalable meshes; `automatic` tries outlines and falls back to coverage
when a glyph has none. This choice affects text only: the other Canvas commands
keep their retained vector geometry.

Image fills, shadowed or blurred groups, and Scene2D placement shaders recorded
in the Canvas follow the same fixed, responsive, animated, and interactive
paths. Viewer therefore has no second effects API: it forwards the retained
intent to the Scene2D renderer. Shadowed `GFX.Font` text remains a `GlyphRun`;
in `vector` mode its outlines remain meshes, while `coverage` mode rasterizes
only the hinted R8 glyphs. The effect never builds a CPU-side RGBA texture for
the complete line.

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

## Animate a retained Canvas

Pass a frame callback when the drawing evolves over time without requiring a
complete interactive session. Viewer creates one Canvas, calls the callback
once with a zero delta, then forwards the time and logical dimensions of every
frame:

```sx
use GFX.Canvas
use GFX.Color
use GFX.Viewer
use STD.Math

var elapsed = 0.0

Viewer.show(
    func(frame:@Viewer.CanvasFrame, drawing:&Canvas) {
        elapsed += frame.delta
        let radius = 24.0 + Math.sin(elapsed) * 6.0

        drawing.clear()
        drawing.paint(func(painter:&Canvas.Painter) {
            painter.fill(
                Canvas.Circle(Math.Vec2(), radius),
                Canvas.Fill.solid(Color.amber_400())
            )
        })
    },
    Viewer.CanvasSettings(
        width:640,
        height:480,
        anchor:Math.Vec2(0.5)
    )
)
```

The callback always receives the same Canvas instance. Viewer neither clears
nor replaces it: the producer decides when its commands change. A frame that
does not mutate the drawing therefore keeps its revision and all retained
preparation. After an intentional mutation, Canvas and Scene2D reuse their
ordinary incremental geometry and allocations.

Retention also applies to filtered groups and image fills: a frame that does
not mutate a group keeps its prepared surface, while a mutation invalidates
only that group and its dependencies.

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
