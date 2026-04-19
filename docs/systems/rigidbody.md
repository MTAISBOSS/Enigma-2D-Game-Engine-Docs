# Rigidbody System

The Rigidbody system is responsible for bringing entities to life with dynamic physics. By attaching a `Rigidbody2D` component to an entity, you place it under the control of the physics engine, allowing it to be affected by gravity, forces, and collisions.

The system is divided into three main parts:

1.  **RigidbodyData**: The pure data container holding the physical properties.

2.  **Rigidbody2D**: The ECS Component that applies physics to the entity's Transform.

3.  **RigidbodyBuilder**: A fluent interface for easily constructing and configuring rigidbodies.

---

## RigidbodyData

The `RigidbodyData` class stores all the physical attributes of a rigidbody. It is accessed via the `Body` property on the `Rigidbody2D` component.

**Properties:**

*   **`LinearVelocity` ($v$)**: A `Vector2` representing the current speed and direction of the object in 2D space.

*   **`AngularVelocity` ($\omega$)**: A `float` representing how fast the object is rotating.

*   **`Mass` ($m$)**: The "weight" or amount of matter in the object. Higher mass requires more force to move. Default is $1$.

*   **`InverseMass` ($\frac{1}{m}$)**: A pre-calculated value heavily used in physics resolution. If the object is static, this evaluates to $0$ (acting as infinite mass).

*   **`Inertia` ($I$)**: The rotational equivalent of mass. It determines how hard it is to start or stop the object from spinning. This is automatically calculated based on the entity's `Collider` shape when the component starts.

*   **`InverseInertia` ($\frac{1}{I}$)**: Used for resolving rotational physics. Evaluates to $0$ if static.

*   **`IsStatic`**: A boolean flag. If `true`, the object is treated as an immovable object (like a floor or wall). It will not be affected by gravity, forces, or collisions.

*   **`HasGravity`**: A boolean flag determining if the global physics gravity should pull on this specific body.

---

## Rigidbody2D (Component)

`Rigidbody2D` is the core component that links the physics data (`RigidbodyData`) to your `Entity`. It automatically synchronizes the physics simulation with the entity's `Transform` and ensures `Collider` caches are updated whenever the object moves.

### Direct Position and Rotation

You can directly get the position and rotation of the rigidbody. Doing so automatically updates the underlying Entity's `Transform` and invalidates the collider cache so collision detection remains accurate.

*   `Position`: Gets the `Vector2` position.
*   `Rotation`: Gets the float angle.

### Movement & Rotation Methods

If you need to move the object manually (e.g., for a kinematic character controller or teleportation), use these explicit methods instead of modifying the Transform directly:

*   `MoveByAmount(Vector2 amount)`: Translates the object relative to its current position.

*   `MoveToExactPosition(Vector2 position)`: Teleports the object to a specific coordinate.

*   `RotateByAmount(float amount)`: Rotates the object by the given angle.

*   `RotateToExactAngle(float angle)`: Sets the object's rotation to a specific angle.

### Physics Functions

*   `AddForce(Vector2 force)`: Applies a push to the rigidbody. The force is accumulated and applied during the next `Simulate` step, altering the linear velocity based on the object's mass ($a = \frac{F}{m}$).

*   `Simulate(float time)`: The core integration step called by the physics engine. It calculates new velocities based on forces and gravity, updates the position and rotation, resets the applied force to zero, and updates collider caches.

---

## Using the RigidbodyBuilder

To create and configure a `Rigidbody2D` cleanly, the engine uses a Fluent Builder pattern. This allows you to chain configuration methods together. The builder automatically handles registering the new rigidbody with the `PhysicsContext`.

### Example Usage
```csharp
// 1. Create your entity
var playerEntity = EntityFactory.Create<Entity>("Player");

// 2. Add a collider (Required for Inertia calculation!)
playerEntity.Components.Add(new PolygonCollider());

// 3. Build the Rigidbody
var rb = new RigidbodyBuilder.Builder<Rigidbody2D>()
.WithOwner(playerEntity)           // Assign the entity
.WithMass(50f)                     // Set mass to 50
.WithGravityState(true)            // Enable gravity
.WithState(false)                  // Not static (it can move)
.WithAngularVelocity(50)           // Sets the initial angular velocity
.WithLinearVelocity(Vector2.One)   // Sets the initial linear velocity
.Build();                          // Finalize and register

// The builder returns the configured Rigidbody2D component.
// Note: You still need to add it to the Entity's component list if your ECS requires it,
// though the Builder registers it with the PhysicsContext directly.
playerEntity.Components.Add(rb);
```
### Builder Methods Reference

*   `.WithOwner(Entity)`: **Required.** Links the component to its parent entity.

*   `.WithMass(float)`: Sets the mass.

*   `.WithState(bool)`: Sets whether the object is static (`true`) or dynamic (`false`).

*   `.WithGravityState(bool)`: Enables/disables gravity.

*   `.WithLinearVelocity(Vector2)`: Gives the object an initial pushing speed.

*   `.WithAngularVelocity(float)`: Gives the object an initial spin.

*   `.Build()`: Validates, creates, registers, and returns the `Rigidbody2D` instance.
