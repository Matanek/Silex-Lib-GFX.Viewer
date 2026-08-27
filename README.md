# GFX.Viewer

`GFX.Viewer` displays visual values in a native window with one `show` call.
It is intended for generated images, authored Canvas drawings, Plot figures,
examples, and exploratory programs.

```text
silex install GFX.Viewer
```

```silex
use GFX.Viewer

Viewer.show(image)
Viewer.show(canvas, 960, 640, "Generated chart")
Viewer.show(responsive_canvas, Viewer.CanvasSettings(width:1180, height:760))
Viewer.show(interactive_session, Viewer.CanvasSettings(width:1180, height:760))
```

The package creates and runs the required GFX application internally. Its
Window, GPU, Rendering, Scene2D, ECS, and plugin composition are not part of
the public Viewer API.

`Canvas.Session` is the stateful form: Viewer forwards frame deltas and GFX
input events, rebuilds its retained Canvas only after a reported visual change
and synchronizes OS text input. `Viewer.CanvasSession` remains a compatibility
alias; the contract itself belongs to GFX.Canvas rather than its presenter.
`Viewer.ImageSettings` and `Viewer.CanvasSettings` expose presentation choices
only when the short forms are insufficient. Future model, animated-sprite,
gallery, and slideshow viewers can add their own typed settings while keeping
`Viewer.show(...)` as the single intent.

See [Docs/Viewer.md](Docs/Viewer.md) for the current image and Canvas behavior.
