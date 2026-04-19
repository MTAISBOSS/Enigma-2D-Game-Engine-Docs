# Math Utilities

The physics engine relies on a robust and optimized set of mathematical structures and functions to calculate movements, forces, and collisions. The core of this system is built upon the `Vector2` struct and the static `Mathematics` helper class.

---

## Vector2

The `Vector2` struct is the fundamental building block for spatial representation in the 2D physics engine. It is used to represent positions, velocities, forces, normals, and any other data requiring both a magnitude and a 2D direction.

### Properties

*   `x` (`float`): The position or magnitude along the horizontal X-axis.

*   `y` (`float`): The position or magnitude along the vertical Y-axis.

### Static Constants
For convenience, `Vector2` provides pre-defined static vectors for common directions and values:

*   `Zero`: $(0, 0)$ - Represents the origin or no movement.

*   `Right`: $(1, 0)$ - Unit vector pointing right.

*   `Left`: $(-1, 0)$ - Unit vector pointing left.

*   `Up`: $(0, 1)$ - Unit vector pointing up.

*   `Down`: $(0, -1)$ - Unit vector pointing down.

*   `One`: $(1, 1)$ - Useful for scaling operations.

### Operators
The struct overloads standard mathematical operators to allow intuitive vector arithmetic:
*   **Addition (`+`) / Subtraction (`-`)**: Component-wise addition/subtraction. Useful for calculating relative positions.

*   **Multiplication (`*`)**: Scales the vector by a `float` scalar ($v \cdot s$).

*   **Division (`/`)**: Divides the vector by a `float` scalar. *Note: Includes a built-in safeguard that throws an exception and beeps if a division by zero is attempted.*

*   **Negation (`-`)**: Reverses the direction of the vector.

*   **Equality (`==`, `!=`)**: Compares two vectors. It uses a small tolerance (epsilon of $0.0001$) under the hood to account for floating-point inaccuracies.

### Important Methods

#### `Translate(Vector2 vector2, Pose2D pose2D)`
This is a crucial spatial transformation function. It takes a local `vector2` and transforms it into world space using a given `Pose2D` (which contains position and rotation data).
It mathematically applies a 2D rotation matrix followed by a translation:
$$x_{new} = (x \cdot \cos(\theta) - y \cdot \sin(\theta)) + P_x$$
$$y_{new} = (x \cdot \sin(\theta) + y \cdot \cos(\theta)) + P_y$$
Where $\theta$ is the rotation from the pose, and $P_x, P_y$ are the pose's position.

---

## Mathematics

The `Mathematics` struct is a static utility class containing essential physics calculations, ranging from vector algebra to floating-point precision handlers.

### Vector Operations

*   **`Normalize(Vector2 a)`**
    Returns a unit vector (a vector with a length of exactly $1$) pointing in the same direction as the input vector. Calculated as $\frac{a}{||a||}$. This is heavily used for collision normals and direction vectors where magnitude is irrelevant.

*   **`DotProduct(Vector2 a, Vector2 b)`**
    Calculates the dot product (scalar product) of two vectors: $a \cdot b = a_x \cdot b_x + a_y \cdot b_y$. 
    *Usage:* Extensively used in physics to find the projection of one vector onto another, determine if objects are moving towards each other, or calculate collision impulses.

*   **`CrossProduct(Vector2 a, Vector2 b)`**
    Calculates the 2D cross product. While a true cross product requires 3D vectors, in 2D physics, this returns a scalar representing the magnitude of the theoretical Z-axis vector: $a \times b = a_x \cdot b_y - a_y \cdot b_x$.
    *Usage:* Crucial for calculating angular velocity, torque, and determining the rotational direction of a collision response.

### Magnitudes & Distances

*   **`Length(Vector2 a)`**
    Returns the true magnitude (length) of the vector using the Pythagorean theorem: $\sqrt{x^2 + y^2}$.

*   **`Distance(Vector2 a, Vector2 b)`**
    Returns the true Euclidean distance between two points: $\sqrt{(a_x - b_x)^2 + (a_y - b_y)^2}$.

*   **`LengthSq(Vector2 a)`** & **`DistanceSq(Vector2 a, Vector2 b)`**
    These functions return the *squared* length ($x^2 + y^2$) and *squared* distance.
    *Performance Tip:* The square root function ($\sqrt{x}$) is a computationally expensive operation. Whenever you only need to *compare* lengths or distances (e.g., checking if an object is within a certain radius), you should compare the squared distance to the squared radius. Use these methods in performance-critical loops like the Broad Phase or Narrow Phase.

### Utility & Precision

*   **`Clamp(float current, float min, float max)`**
    Restricts a value so it does not fall below a minimum or exceed a maximum. Useful for capping velocities, forces, or physical properties.
    
*   **`IsNearlyEqual(...)`** *(Overloads for `float` and `Vector2`)*
    Due to the way computers handle floating-point numbers, calculating $0.1 + 0.2$ might result in $0.30000004$. Checking for absolute equality (`a == b`) can lead to bugs. This function safely checks if two numbers or vectors are functionally identical by ensuring their difference is smaller than a tiny threshold (an epsilon of $0.0005$).
