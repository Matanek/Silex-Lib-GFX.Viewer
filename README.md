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
```

The package creates and runs the required GFX application internally. Its
Window, GPU, Rendering, Scene2D, ECS, and plugin composition are not part of
the public Viewer API.

`Viewer.ImageSettings` and `Viewer.CanvasSettings` expose presentation choices
only when the short forms are insufficient. Future model, animated-sprite,
gallery, and slideshow viewers can add their own typed settings while keeping
`Viewer.show(...)` as the single intent.

See [Docs/Viewer.md](Docs/Viewer.md) for the current image and Canvas behavior.
