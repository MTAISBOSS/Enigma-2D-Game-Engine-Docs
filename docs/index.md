# Welcome to the Physics Engine ⚙️

The Physics Engine is a lightweight, C#-based framework designed for building robust physics simulations and games.

## Key Features

*   **Object-Oriented ECS:** A flexible and intuitive Entity Component System.
*   **Modular Physics Components:** Easily add `RigidBody`, `Collider`, and custom behaviors.
*   **Service Locator:** A clean architecture for accessing global systems like `EntityContainer`.
*   **Extendable Core:** Built to be customized for your specific project needs.

!!! tip "Ready to start?"
    Dive into the [Entity Component System](core/ecs.md) documentation to understand the core architecture or head to the [Quick Start Guide](getting-started/quickstart.md).

## Architecture at a glance

The engine operates on a fixed-step update loop to ensure stable physics calculations:

$$ \Delta t = \text{CurrentTime} - \text{PreviousTime} $$
