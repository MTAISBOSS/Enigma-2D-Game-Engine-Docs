# Entity Component System (ECS)

The `Physics_Engine.Core.Entity_Component_System` namespace provides a robust, object-oriented Entity-Component architecture. Unlike pure data-driven ECS models, this framework uses a GameObject-style approach (similar to Unity) where an `Entity` acts as a container for modular `Component` objects.

---

## Component Base Class

Every component attached to an entity must inherit from the `Component` abstract class. It provides a standard implementation with lifecycle methods and automatically holds a reference to its parent `Entity`.
```csharp
public abstract class Component
{
public Entity Entity { get; internal set; }
public virtual void Start(){}
public virtual void Update(){}
public virtual bool IsAbleToDuplicate() => true;
}
```

**Key Methods:**

*   `Start()`: Called automatically when the component is added to the `ComponentContainer`.
*   `Update()`: Intended to be called every frame (handled by the `ComponentContainer` and `EntityContainer` update loop).
*   `IsAbleToDuplicate()`: Defaults to `true`. Override this and return `false` if an Entity should only ever have one instance of this component (e.g., a `Transform` or `RigidBody`).

**Example Usage:**
```csharp
public class PlayerController : Component
{
public override void Start()
{
// Initialization logic here
}

public override void Update()
{
// Frame-by-frame logic here
}

public override bool IsAbleToDuplicate() => false; // Only one allowed!
}
```
---

## ComponentContainer

The `ComponentContainer` manages the lifecycle and storage of all components attached to a specific `Entity`. It uses a `Dictionary<Type, List<Component>>` to store components, ensuring fast type-based lookups with an average time complexity of $\mathcal{O}(1)$.

**Key Behaviors:**

*   **Initialization:** When you call `Add<T>()`, the container automatically injects the parent `Entity` reference and calls `Start()` on the base component.

*   **Duplication Safety:** If a component's `IsAbleToDuplicate()` returns `false`, the container will throw an `InvalidOperationException` if you attempt to add a second instance of that exact type.

**API Reference:**

*   `Add<T>(T component)` - Attaches a component to the entity and initializes it.

*   `TryGet<T>(out T component)` - Safely attempts to retrieve the first component of type `T`. Returns `true` if found.
*   `Get<T>()` - Returns the first component of type `T`, or `null` if it doesn't exist.

*   `GetAll<T>()` - Returns an `IEnumerable<T>` containing all components of type `T` attached to the entity.

*   `Remove<T>()` - Completely removes all components of type `T` from the entity.

*   `Update()` - Iterates through all stored components and calls their respective `Update()` methods.

**Example: Adding and Getting Components**
```csharp
Entity player = new Entity("Player");

// Adding components
player.Components.Add(new PlayerController());
player.Components.Add(new Collider());

// You can also first create a component and initialize values
// Let's create a sprite renderer
Sprite playerRenderer = new Sprite();

// Modify the component's values
playerRenderer.Color = Color4.Red; 
playerRenderer.Layer = LayerMask.World;

// Add the component
player.Components.Add(playerRenderer);

// Retrieving components
if (player.Components.TryGet<PlayerController>(out var controller))
{
// Use controller...
}

// Get specific component
player.Components.Get<Collider>();
```
---

## Entity

The `Entity` class is the core object representing an actor, item, or logical grouping in your simulation.

**Properties:**

*   `Name` (`string`): The human-readable name of the entity.

*   `Tag` (`string`): Used for grouping or identifying entities (e.g., "Player", "Enemy").

*   `Transform` (`Transform`): A lazy-loaded property. The first time you access `entity.Transform`, it automatically creates a `Transform` component, attaches it to the `ComponentContainer`, and returns it.

*   `Components` (`ComponentContainer`): The registry of all components attached to this entity.

!!! note "Service Locator Integration & Memory Management"
When an `Entity` is instantiated, it automatically resolves the `EntityContainer` via the `ServiceLocator` and registers itself. You do not need to manually add new entities to the global physics loop. Furthermore, the `Entity` class includes a finalizer (`~Entity()`) that automatically unregisters the entity from the `EntityContainer` upon garbage collection, safely preventing memory leaks.

---

## EntityContainer

The `EntityContainer` acts as the global registry for all active entities in the simulation. It implements the `IService` interface, allowing it to be globally accessible via your `ServiceLocator`.

**Lifecycle:**

*   **Construction:** Automatically registers itself to `ServiceLocator.Instance`.

*   **Destruction:** Automatically unregisters itself via its finalizer (`~EntityContainer()`).

**API Reference:**

*   `RegisterEntity(Entity entity)` - Adds an entity to the tracked internal list (called automatically by the `Entity` constructor).

*   `UnregisterEntity(Entity entity)` - Removes an entity from the tracked list (called automatically by the `Entity` finalizer).

*   `Update()` - Iterates through all registered entities and calls `Update()` on their respective `ComponentContainer`s.

---

## EntityFactory

A static utility struct designed to safely instantiate generic Entities. It uses reflection (`Activator.CreateInstance`) to construct entities that inherit from the base `Entity` class.

**API Reference:**

*   `Create<T>(string name = "", string tag = "")` $\rightarrow$ `T`

**Example: Using the Factory**
```csharp
// Creates a standard entity
Entity genericProp = EntityFactory.Create<Entity>("Box", "Prop");

// Creates a custom entity class that inherits from Entity
MyCustomPlayerEntity player = EntityFactory.Create<MyCustomPlayerEntity>("Hero", "Player");
```