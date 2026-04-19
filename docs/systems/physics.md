# Physics Engine Core

The core of the physics simulation is managed by the `PhysicsContext` class, which orchestrates the entire process of movement, collision detection, and collision resolution. The simulation loop is designed for efficiency and accuracy, breaking the complex problem of collision handling into two distinct stages: the **Broad Phase** and the **Narrow Phase**.

The overall behavior of the simulation can be globally tuned using the static `PhysicsSetting` class.

---

## The Simulation Loop (PhysicsContext)

The `PhysicsContext` is the central service that tracks and manages every `Rigidbody2D` in the scene. Its primary responsibility is to run the main simulation loop each frame. This loop consists of three key steps:

1.  **Move:** Update the position and rotation of every dynamic rigidbody based on its velocity and applied forces.

2.  **Detect:** Perform collision detection to find all pairs of overlapping objects.

3.  **Resolve:** Calculate and apply the physical reactions (impulses) for all detected collisions to realistically alter their velocity.

To perform step 2 efficiently, the engine uses a two-phase approach.

### What is the Broad Phase?

The **Broad Phase** is a quick, initial filtering step. Its goal is to efficiently discard pairs of objects that are obviously not colliding, leaving a much smaller list of potential candidates.

*   **How it works:** Instead of checking the precise, complex shapes of objects, the engine first calculates a simple, non-rotated rectangle called an 
**Axis-Aligned Bounding Box (AABB)** that completely encloses each object. Checking for overlap between two simple rectangles is extremely fast.

*   **Purpose:** This step dramatically reduces the number of expensive, detailed collision checks that need to be performed. If the AABBs of two objects aren't touching, their actual shapes can't be touching either.

### What is the Narrow Phase?

The **Narrow Phase** is the second, more precise stage. It takes the smaller list of potentially colliding pairs from the broad phase and performs accurate collision detection on them.

*   **How it works:** For each pair, the engine uses the actual `Collider` shapes (e.g., `CircleCollider`, `PolygonCollider`) to determine if they are truly intersecting. If they are, it calculates detailed `CollisionInfo`, such as the collision normal (the direction of the impact) and the penetration depth.

*   **Collision Resolution:** Once a true collision is confirmed, the `NarrowPhase` is responsible for calling a **Collision Resolver** method. This is where you can define the physical behavior of the impact. The engine can use different methods to calculate the response, from simple bounces to complex reactions involving rotation and friction.

> **Note:** The specific `CollisionResolver` methods (`ResolveBasic`, `ResolveWithRotation`, etc.) and their use cases are discussed in detail in the **Collision System** documentation.

---

## PhysicsSetting

The `PhysicsSetting` class is a crucial static class that contains global constants and configuration values for the entire physics simulation. Adjusting these values allows you to tune the performance, stability, and overall "feel" of the engine's physics.

**Key Settings:**

*   `GravityDirection`: A `Vector2` that defines the direction and magnitude of the global gravity force. By default, it is `(0, -9.81)`, simulating a downward pull. You can change this to create top-down gravity or zero-gravity environments.

*   `Iterations`: This is one of the most important settings for simulation stability. The physics engine solves collisions iteratively. A higher number of `Iterations` per frame results in a more accurate and stable simulation, preventing fast-moving objects from "tunneling" through thin walls. However, this comes at a higher performance cost. The default is `2`, which is a good balance for many games.

*   `MinMass` / `MaxMass`: A hard clamp on the final calculated mass of any object, ensuring all rigidbodies stay within a reasonable and stable range.
