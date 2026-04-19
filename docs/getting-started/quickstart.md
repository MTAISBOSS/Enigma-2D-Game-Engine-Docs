# Quick Start Guide

This guide will show you how to initialize the Enigma Engine and open a basic window.

## 1. Engine Initialization

Include the core engine header and initialize the system.

````cpp title="main.cpp"
#include <Enigma/Enigma.h>

int main() {
Enigma::Application app;

// Initialize with width, height, and title
app.Init(1280, 720, "My Enigma Game");

app.Run();
app.Shutdown();

return 0;
}

!!! warning "Important"
Always ensure `app.Shutdown()` is called to free memory allocated by the core allocator.

## 2. Adding a Game Loop

If you want manual control over the loop instead of using `app.Run()`, you can do this:

cpp
while (app.IsRunning()) {
app.PollEvents();
app.Update();
app.Render();
}


#### `docs/systems/physics.md` (Physics System - Showing Math formatting)
```markdown
# Physics & Collision

The Enigma Engine uses a custom 3D physics solver.

## Rigid Body Dynamics

To apply forces to an entity, we use Newton's second law of motion, where Force $F$ is equal to mass $m$ multiplied by acceleration $a$:

$$ F = m \times a $$

You can apply a force directly in code:
```cpp
Enigma::Entity player = scene.GetEntity("Player");
player.GetComponent<RigidBody>().ApplyForce(Vector3(0.0f, 9.81f, 0.0f));

## Velocity and Position

The engine calculates the new velocity $v_{f}$ over the delta time $\Delta t$ using:

$$ v_{f} = v_{i} + (a \times \Delta t) $$

??? note "Custom Gravity"
You can change the global gravity multiplier in the `PhysicsSettings` struct before initializing the physics system.


### Next Steps
1. Create all the folders and blank `.md` files matching the `nav` list in your `docs/` folder.
2. Paste these templates into their respective files.
3. Run `mkdocs serve -a localhost:8080` to see your complete site structure! Send your codes whenever you are ready to fill in the rest.
````
