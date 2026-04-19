# Collision System

The engine's collision system is a component-based framework designed to detect and resolve collisions between entities. It consists of two main parts:

1.  **Colliders**: Components that define the physical shape of an entity for collision detection.
2.  **CollisionResolver**: A static utility class that calculates the physical response (impulse) when two colliders intersect.

---

## Collider (Base Class)

The `Collider` is an abstract base class that all specific collider shapes inherit from. You attach a `Collider` derivative to an `Entity` to give it a physical presence in the world.

**Key Properties:**

*   `Material` (`PhysicMaterial`): Defines the physical properties of the surface, such as restitution (bounciness) and friction.

!!! note "Restitution (specifically the coefficient of restitution) is a measure of ***bounciness*** or elasticity in a collision. It determines how much kinetic energy is retained or lost when two objects hit each other.Typically represented by a value between 0 and 1"

!!! note "Static friction is the force that prevents an object from starting to move. When two objects are at rest relative to each other, their surfaces interlock. You must apply a certain amount of force to break this lock before the object will slide. "

!!! note "Dynamic friction is the force that resists the motion of an object that is already moving (sliding)."

*   `IsTrigger` (`bool`): If set to `true`, the collider will detect intersections and fire `OnTriggerEnter` events, but it will not produce a physical collision response. This is useful for creating detection zones, like checkpoints or damage areas.

**Key Methods:**

*   `Intersects(Collider other, out CollisionInfo collisionInfo)`: The core function used by the physics engine to check if this collider is intersecting with another. It returns `true` if they overlap and provides detailed information about the collision in the `collisionInfo` output parameter.

**Example: Modifying Collider Properties**
```csharp
// Get the entity's collider component
var playerCollider = playerEntity.Components.Get<PolygonCollider>();

// Make the collider a trigger
playerCollider.IsTrigger = true;

// Adjust its physical material
playerCollider.Material.Restitution = 0.2f; // Make it less bouncy
playerCollider.Material.DynamicFriction = 0.5f;
---
```
## Collider Types

The engine provides several primitive collider shapes.

### CircleCollider

A circular collider, defined by a radius. It is computationally efficient and perfect for objects that can be approximated as a circle, such as balls, player characters in some top-down games, or projectiles.

**Key Property:**

*   `CircleArea.Radius` (`float`): The radius of the collision circle, derived from the entity's `Transform.Scale.x`.

**Usage:**
```csharp
// Create a new entity for a ball
var ball = EntityFactory.Create<Entity>("Ball");

// Add and configure a CircleCollider
var circleCollider = new CircleCollider();
ball.Components.Add(circleCollider);

// The radius is automatically set from the entity's scale
ball.Transform.Scale = new Vector2(10, 10); // Radius will be 10
```
### PolygonCollider

A collider for convex polygon shapes. The current implementation is optimized for creating four-sided polygons (boxes/rectangles), making it ideal for crates, walls, platforms, and other rectangular objects.

**Key Property:**

*   `BoxArea` (`BoxArea`): Contains the `Width` and `Height` of the collider, derived from the entity's `Transform.Scale`.

**Usage:**
```csharp
// Create a new entity for a wall
var wall = EntityFactory.Create<Entity>("Wall");

// Add a PolygonCollider to it
var boxCollider = new PolygonCollider();
wall.Components.Add(boxCollider);

// The size is automatically set from the entity's scale
wall.Transform.Scale = new Vector2(100, 20); // Creates a 100x20 box collider
```
---

## CollisionResolver

The `CollisionResolver` is a static class that handles the physics response after a collision has been detected. It applies impulses to the colliding bodies to change their linear and angular velocity. You can choose a resolution method based on the complexity and realism your game requires.

### ResolveBasic

*   **What it does:** This is the simplest resolver. It calculates a bounce response based on the collision normal but **does not account for rotation**. Objects will change direction but will not start spinning after a collision.

*   **Best for:** Simple, fast-paced arcade games where rotational physics are unnecessary. Examples include Pong, Arkanoid, or simple top-down shooters.

### ResolveWithRotation

*   **What it does:** This resolver adds rotational physics to the collision response. It calculates how an off-center impact should apply torque to the objects, causing them to spin realistically.

*   **Best for:** The majority of physics-based games where objects are expected to rotate. Examples include billiards games, platformers with physics-based props, or any game where tumbling objects add to the experience.

### ResolveWithRotationWithFriction

*   **What it does:** This is the most realistic resolver. It builds upon the rotational resolver by adding **static and dynamic friction**. This allows objects to slide against each other, lose momentum from friction, and come to a rest.

*   **Best for:** Games that require more nuanced physical interactions. Examples include top-down racing games where tire grip is important, puzzle games involving stacking blocks, or simulations where surface materials should affect object movement.

!!! warning "Performance"
Based on your game you can choose between the 3 collision resolve methods, but keep in mind that your choice will **impact on performance**.