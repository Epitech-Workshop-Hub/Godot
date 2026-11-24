# Godot en 3D

## ATTENTION À LIRE AVANT DE COMMENCER

**Ce workshop part du principe que vous maîtrisez déjà les bases de Godot et/ou que vous avez participé au workshop `Godot Introduction`**

Vous devriez déjà avoir installé la [dernière version de **Godot .NET**](https://godotengine.org/download/linux/)  
Consultez la [documentation officielle de Godot](https://docs.godotengine.org/en/stable/about/introduction.html)

# Introduction

Godot utilise des nodes similaires entre la 2D et la 3D, cependant, opérer avec une dimension en plus apporte de la complexité.

Nous ne allons pas revenir sur les Scènes, les Nodes, les Signaux ou les bases du C#, déjà vus dans le workshop `Godot Introduction`. Ce workshop se concentre uniquement sur ce qui change en 3D.

Notre objectif sera de créer un mini **FPS** : un joueur qui se déplace, regarde autour de lui, et tire sur des cibles.

---

# Rappel : de la 2D à la 3D

Avant de commencer, quelques différences essentielles à garder en tête :

* Les Nodes 2D (`Node2D`, `Sprite2D`, `Area2D`, ...) ont leur équivalent en 3D : `Node3D`, `MeshInstance3D`, `Area3D`, ...
* On travaille maintenant avec des `Vector3` au lieu de `Vector2`. L'axe **Y** représente désormais la hauteur (au lieu de la profondeur en 2D).
* La rotation d'un Node2D est un simple angle (`float`). En 3D, un Node3D peut tourner sur 3 axes à la fois : on manipule sa `Transform3D`, composée d'une position et d'une `Basis` (l'équivalent d'une matrice de rotation/échelle).

> Pas besoin de maîtriser toutes les maths derrière la `Basis` pour ce workshop, on va simplement l'utiliser pour orienter les déplacements du joueur.

---

# Le décor

Avant d'avoir un joueur, il nous faut un endroit où le faire évoluer.

Créez une nouvelle scène avec un Node `Node3D` comme racine, renommé `Main`.

Ajoutez :

* Un `WorldEnvironment` pour définir l'ambiance générale de la scène (ciel, éclairage ambiant).
* Un `DirectionalLight3D` pour simuler une lumière type "soleil".
* Un sol : `StaticBody3D` avec un `MeshInstance3D` (un `BoxMesh` ou `PlaneMesh` suffit) et un `CollisionShape3D` associé.

Ajoutez ensuite quelques obstacles (des `StaticBody3D` avec des `BoxMesh`) pour donner un peu de relief à la scène et servir de repères visuels une fois en jeu.

> Les nodes `CSG*` (`CSGBox3D`, `CSGSphere3D`, ...) sont une alternative pratique pour prototyper rapidement un décor sans avoir à gérer manuellement Mesh + CollisionShape séparément.

---

# Le joueur FPS

C'est le cœur de ce workshop.

## Se déplacer

Créez une nouvelle scène pour le joueur, avec un `CharacterBody3D` comme racine, renommé `Player`. Ajoutez-lui un `CollisionShape3D` (une `CapsuleShape3D` est adaptée à un personnage).

Contrairement au joueur 2D qui utilisait `Position += velocity * Speed * delta`, un `CharacterBody3D` gère lui-même les déplacements et les collisions avec la physique via la méthode `MoveAndSlide()`.

Le script du joueur devra gérer :

* La lecture des entrées clavier (`move_left`, `move_right`, `move_forward`, `move_back`) via l'Input Map, comme en 2D.
* La direction du déplacement, calculée à partir de la `Basis` du joueur (pour que "avancer" corresponde toujours à la direction regardée).
* La gravité, à appliquer chaque frame sur la vélocité verticale (`Velocity.Y`) tant que le joueur n'est pas au sol (`IsOnFloor()`).
* Le saut, via une nouvelle action `jump` dans l'Input Map.

## Regarder autour de soi

Ajoutez un Node `Node3D` enfant du `Player` (par exemple `Head`, positionné à hauteur des yeux), puis un `Camera3D` enfant de `Head`.

On capture la souris avec `Input.MouseMode = Input.MouseModeEnum.Captured`, puis on utilise `_UnhandledInput()` pour lire les mouvements de souris (`InputEventMouseMotion`).

Deux rotations distinctes sont nécessaires :

* La rotation horizontale (**yaw**) s'applique sur le `Player` lui-même (elle doit affecter la direction de déplacement).
* La rotation verticale (**pitch**), qui permet de lever/baisser la tête, s'applique uniquement sur `Head` (et doit être `Clamp`ée pour éviter de se retourner la tête à 180°).

---

# Tirer sur des cibles

Ajoutez un `RayCast3D` en enfant de la `Camera3D`, pointant droit devant (`TargetPosition = new Vector3(0, 0, -100)` par exemple).

Lorsque le joueur appuie sur une action `shoot` :

* On vérifie si le `RayCast3D` touche quelque chose (`IsColliding()`).
* Si l'objet touché fait partie d'une cible, on émet un signal custom (comme vu avec `Hit` en 2D) pour prévenir que la cible a été touchée.
* La cible réagit à ce signal en disparaissant (`QueueFree()`) ou en jouant un petit effet.

Créez une scène `Target` (par exemple un `StaticBody3D` avec un `MeshInstance3D` en forme de sphère ou de cube) et placez-en plusieurs dans le décor.

---

# Le HUD

Bonne nouvelle : le `CanvasLayer` fonctionne exactement de la même manière qu'en 2D, qu'on soit dans une scène 2D ou 3D.

Ajoutez un `CanvasLayer` à votre scène principale, avec :

* Un viseur au centre de l'écran (`TextureRect` ou simplement un `Label` avec un `+`).
* Un `Label` affichant le nombre de cibles touchées.

---

## En Autonomie :

Pour aller plus loin, essayez d'ajouter :

* Un `Timer` qui fait réapparaître nos cibles à une position aléatoire après un délai.
* Un score et/ou un chrono affiché sur le HUD.
* Un feedback visuel ou sonore lorsqu'une cible est touchée.

## Auteur(s)
| [<img src="https://github.com/Kapwiing.png?size=85" width=85><br><sub>Guillaume CBV</sub>](https://github.com/EdenComp)
| :---:
