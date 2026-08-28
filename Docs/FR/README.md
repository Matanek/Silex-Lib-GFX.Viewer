# Afficher une valeur visuelle avec GFX.Viewer

`GFX.Viewer` ouvre une fenêtre native pour une valeur visuelle produite par un
programme Silex. Il possède l’application de présentation et masque toute la
composition Window, GPU, Rendering, Scene2D, ECS et shaders.

[Read this documentation in English.](../EN/README.md)

## Installer le package

```text
silex install GFX.Viewer
```

GFX.Viewer demande Silex 0.39.0 ou une version plus récente.

## Afficher une image

Après avoir créé ou chargé une valeur `image`, affichez-la à sa taille
naturelle :

```sx
use GFX.Viewer

Viewer.show(image)
Viewer.show(image, Viewer.ImageSettings(title:"Generated image"))
```

Ce fragment suppose une image compatible déjà disponible. `ImageSettings`
contrôle la fenêtre, l’arrière-plan et l’échantillonnage de texture lorsque la
forme courte ne suffit pas :

```sx
Viewer.show(image, Viewer.ImageSettings(
    title:"Nearest-neighbour preview",
    width:960,
    height:640,
    smooth:false,
))
```

Les petites images restent centrées à leur résolution native. Les images plus
grandes sont réduites en conservant leurs proportions.

## Afficher un Canvas

Un Canvas reste vectoriel et suit le viewport de la fenêtre :

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

La forme positionnelle `Viewer.show(drawing, width, height, title)` reste le
parcours le plus court pour les exemples et graphiques générés.

Lorsque le layout dépend de la fenêtre, passez une fonction de dessin. Viewer
l’appelle au démarrage puis après chaque redimensionnement afin que le
producteur recalcule sa disposition :

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

Ce fragment suppose une fonction `build_interface` propre à l’application.

## Présenter un contenu interactif

Un contenu Canvas interactif implémente `Canvas.Session` :

```sx
class Preview:Canvas.Session {
    func drawing(width:int, height:int) Canvas { return build(width, height) }
    func update(delta:float) bool { return advance_animations(delta) }
    func handle(event:@GFX.Input.Event) bool { return update_state(event) }
    func accepts_text_input() bool { return editing_text() }
}

Viewer.show(Preview(), Viewer.CanvasSettings(width:1180, height:760))
```

Les fonctions appelées dans ce fragment appartiennent au contenu. Viewer
transmet les deltas de frame et événements d’input à la session. Un résultat
`true` de `update` ou `handle` demande un nouveau dessin retenu ; `false` évite
de reconstruire un contenu inactif. `accepts_text_input` synchronise l’input
texte du système.

`Viewer.show_until(session, close_when, settings)` évalue un prédicat après
chaque mise à jour et ferme sa fenêtre lorsqu’il retourne `true`. Une session
ordinaire reste ouverte jusqu’à la fermeture par l’utilisateur.

`Viewer.CanvasSession` reste un alias de compatibilité de `Canvas.Session`. Le
contrat appartient à Canvas, pas à son presenter. Tous les appels Viewer
bloquent jusqu’à la fermeture de la fenêtre ; leur plomberie Application reste
privée.
