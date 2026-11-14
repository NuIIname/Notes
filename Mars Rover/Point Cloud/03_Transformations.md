# Chapter 3 — 3D Transformations

**Topic:** Point Cloud Processing, Linear Algebra, and Coordinate Systems

**Primary Resource:** [Interactive PCL Tutorial — Chapter 3](https://limhyungtae.github.io/pcl_tutorial/)

**Additional Resource:** [Eigen — Geometry Module](https://libeigen.gitlab.io/eigen/docs-nightly/group__TutorialGeometry.html)

---

## 1. Introduction

A **transformation** is a mathematical operation that changes the position or orientation of an object or a collection of points.

In point-cloud processing, transformations allow us to express measurements in different coordinate systems.

Two fundamental operations are:

- **Translation:** Moving a point or object by a specified displacement.
- **Rotation:** Turning a point or object around an axis by a specified angle.

When these operations are combined without scaling or distorting the object, they form a **rigid transformation**.

Rigid transformations preserve the distances between points and the shape of the original object.

### Why are transformations important?

Imagine a camera mounted on a moving rover.

The camera detects a rock three meters away. However, that measurement is relative to the camera's current position and orientation.

The rover's navigation system needs the rock's position relative to the rover or the surrounding world.

Transformations allow us to convert between these coordinate systems.

---

# Part I — Mathematical Foundations

## 2. Cartesian Coordinate Systems

A Cartesian coordinate system describes positions using perpendicular axes.

### Two-dimensional coordinates

In 2D space, a point is described using two coordinates:

$$
P=(x,y)
$$

- $x$ specifies horizontal position.
- $y$ specifies vertical position.

For example:

$$
P=(3,2)
$$

This represents a point three units along the positive X-axis and two units along the positive Y-axis.

The **origin** is the point where the axes intersect:

$$
O=(0,0)
$$

### Three-dimensional coordinates

In 3D space, we introduce a third axis:

$$
P=(x,y,z)
$$

The axes are mutually perpendicular.

In a conventional Z-up coordinate system:

- X represents one horizontal direction.
- Y represents the other horizontal direction.
- Z represents vertical position.

However, **the physical meaning of an axis depends on the coordinate system being used**.

For example, some camera coordinate systems use Z for forward depth rather than height.

This distinction is important when processing ZED camera measurements.

## 3. Points and Vectors

A **point** represents a position in space.

A **vector** represents a displacement, direction, or magnitude along the coordinate axes.

Both can be written using the same coordinate notation.

For example:

$$
P=(2,3,4)
$$

$$
\mathbf{v}=(5,-1,2)
$$

Here, $P$ is a point and $\mathbf{v}$ is a displacement vector.

Adding the vector to the point moves the point:

$$
P'=P+\mathbf{v}
$$

$$
P'=(2+5,\ 3-1,\ 4+2)
$$

$$
\boxed{P'=(7,2,6)}
$$

Vectors are important because both translation and rotation can be described mathematically using them.

### Vector magnitude

The magnitude of a vector is its length.

For a 3D vector:

$$
\mathbf{v}=(x,y,z)
$$

Its length is:

$$
\|\mathbf{v}\|=\sqrt{x^2+y^2+z^2}
$$

For example:

$$
\mathbf{v}=(3,4,0)
$$

$$
\|\mathbf{v}\|=\sqrt{3^2+4^2+0^2}=5
$$

Rigid rotations preserve this length.

## 4. Matrix Fundamentals

A **matrix** is a rectangular arrangement of numbers organized into rows and columns.

For example:

$$
A=
\begin{bmatrix}
1&2\\
3&4
\end{bmatrix}
$$

This is a **2×2 matrix** because it contains two rows and two columns.

A column vector can also be represented as a matrix:

$$
P=
\begin{bmatrix}
3\\
2
\end{bmatrix}
$$

This is a 2×1 matrix.

### Matrix multiplication

To multiply a matrix by a column vector:

1. Take the first row of the matrix.
2. Multiply each entry by the corresponding entry in the vector.
3. Add the products to obtain the first output coordinate.
4. Repeat for each remaining row.

For example:

$$
\begin{bmatrix}
1&2\\
3&4
\end{bmatrix}
\begin{bmatrix}
5\\
6
\end{bmatrix}
$$

First row:

$$
(1)(5)+(2)(6)=17
$$

Second row:

$$
(3)(5)+(4)(6)=39
$$

Therefore:

$$
\boxed{
\begin{bmatrix}
17\\
39
\end{bmatrix}
}
$$

### Matrix dimensions

A matrix with $m$ rows and $n$ columns can multiply a matrix with $n$ rows.

The inner dimensions must match.

For example:

$$
(3\times3)(3\times1)=(3\times1)
$$

This lets a 3×3 rotation matrix transform a 3D coordinate vector.

### Multiplying two matrices

When multiplying matrices, each element in the output is calculated using a **row from the first matrix and a column from the second matrix**.

For example:

$$
A=
\begin{bmatrix}
1&2\\
3&4
\end{bmatrix},
\qquad
B=
\begin{bmatrix}
5&6\\
7&8
\end{bmatrix}
$$

Their product is:

$$
AB=
\begin{bmatrix}
(1)(5)+(2)(7)&(1)(6)+(2)(8)\\
(3)(5)+(4)(7)&(3)(6)+(4)(8)
\end{bmatrix}
$$

$$
\boxed{
AB=
\begin{bmatrix}
19&22\\
43&50
\end{bmatrix}
}
$$

**Important:** Matrix multiplication is not generally commutative.

$$
AB\neq BA
$$

Changing the multiplication order can change the result.

This becomes especially important when combining several coordinate transformations.

## 5. Trigonometry Fundamentals

Rotation matrices rely on two trigonometric functions:

- Sine: $\sin(\theta)$
- Cosine: $\cos(\theta)$

Here, $\theta$ (theta) represents an angle.

### Right-triangle definitions

For a right triangle:

$$
\sin(\theta)=\frac{\text{opposite}}{\text{hypotenuse}}
$$

$$
\cos(\theta)=\frac{\text{adjacent}}{\text{hypotenuse}}
$$

The hypotenuse is the longest side of a right triangle.

The opposite and adjacent sides are identified relative to the chosen angle.

### Connection to rotation

Imagine a point on a circle with radius 1 centered at the origin.

After rotating from the positive X-axis by an angle $\theta$, its coordinates are:

$$
P=(\cos\theta,\sin\theta)
$$

This relationship explains why sine and cosine appear in rotation matrices.

### Common trigonometric values

| Angle | Sine | Cosine |
|---|---|---|
| 0° | 0 | 1 |
| 30° | 0.5 | 0.8660 |
| 45° | 0.7071 | 0.7071 |
| 60° | 0.8660 | 0.5 |
| 90° | 1 | 0 |

At 45°:

$$
\sin45^\circ=\cos45^\circ=\frac{\sqrt2}{2}
$$

### Degrees and radians

Angles can be expressed in degrees or radians.

A full rotation is:

$$
360^\circ=2\pi\text{ radians}
$$

To convert degrees to radians:

$$
\theta_{\text{rad}}=\theta_{\text{deg}}\frac{\pi}{180}
$$

For example:

$$
90^\circ=\frac{\pi}{2}\text{ radians}
$$

**Programming note:** C++ trigonometric functions such as `std::sin()` and `std::cos()` expect radians, not degrees.

---

# Part II — Translation and Rotation

## 6. Translation

Translation moves a point by adding a displacement vector.

For a point:

$$
P=(x,y,z)
$$

And translation vector:

$$
\mathbf{t}=(t_x,t_y,t_z)
$$

The translated point is:

$$
P'=P+\mathbf{t}
$$

Or, written by coordinate:

$$
\begin{aligned}
x'&=x+t_x\\
y'&=y+t_y\\
z'&=z+t_z
\end{aligned}
$$

### Example

Given:

$$
P=(3,5,2)
$$

And:

$$
\mathbf{t}=(-2,4,6)
$$

Calculate:

$$
P'=(3-2,\ 5+4,\ 2+6)
$$

$$
\boxed{P'=(1,9,8)}
$$

Translation preserves the distances between points and their relative orientations.

## 7. Understanding Rotation

Rotation changes a point's position by turning it around an axis.

In two dimensions, we typically rotate around the origin.

In three dimensions, rotation can occur around the X, Y, or Z axes.

### Rotation around the coordinate axes

For rotations around axes passing through the origin:

| Rotation axis | Unchanged coordinate | Rotating coordinates |
|---|---|---|
| X | X | Y and Z |
| Y | Y | X and Z |
| Z | Z | X and Y |

At 90°, the rotating coordinates exchange positions, and one receives a negative sign.

The sign depends on the direction and axis of rotation.

### The right-hand rule

The **right-hand rule** establishes the positive direction of rotation in a right-handed coordinate system.

1. Point your right thumb along the positive direction of the rotation axis.
2. Curl your fingers naturally toward your palm.
3. The direction your fingers curl represents positive rotation.

When viewed from the positive end of the rotation axis looking toward the origin, positive rotation appears counterclockwise.

This convention determines the signs used in rotation matrices.

---

# Part III — Rotation Matrices

## 8. Two-Dimensional Rotation Matrices

A 2D rotation matrix is:

$$
R(\theta)=
\begin{bmatrix}
\cos\theta&-\sin\theta\\
\sin\theta&\cos\theta
\end{bmatrix}
$$

This matrix rotates a point counterclockwise by $\theta$ around the origin when using standard XY coordinates and column vectors.

### Why does the matrix look like this?

Consider the unit vector along the X-axis:

$$
\begin{bmatrix}1\\0\end{bmatrix}
$$

After rotation by $\theta$, it points toward:

$$
\begin{bmatrix}\cos\theta\\\sin\theta\end{bmatrix}
$$

The unit vector along the Y-axis:

$$
\begin{bmatrix}0\\1\end{bmatrix}
$$

rotates to:

$$
\begin{bmatrix}-\sin\theta\\\cos\theta\end{bmatrix}
$$

These two rotated unit vectors become the columns of the rotation matrix.

### Example: 90° counterclockwise

We know:

$$
\cos90^\circ=0,\qquad\sin90^\circ=1
$$

Therefore:

$$
R(90^\circ)=
\begin{bmatrix}
0&-1\\
1&0
\end{bmatrix}
$$

Rotate the point:

$$
P=
\begin{bmatrix}
4\\
-2
\end{bmatrix}
$$

Multiplication gives:

$$
P'=
\begin{bmatrix}
0&-1\\
1&0
\end{bmatrix}
\begin{bmatrix}
4\\
-2
\end{bmatrix}
$$

First coordinate:

$$
x'=(0)(4)+(-1)(-2)=2
$$

Second coordinate:

$$
y'=(1)(4)+(0)(-2)=4
$$

Therefore:

$$
\boxed{P'=(2,4)}
$$

For a positive 90° rotation:

$$
\boxed{(x,y)\rightarrow(-y,x)}
$$

### Example: 45° counterclockwise

At 45°:

$$
\sin45^\circ=\cos45^\circ\approx0.7071
$$

The rotation matrix is:

$$
R(45^\circ)=
\begin{bmatrix}
0.7071&-0.7071\\
0.7071&0.7071
\end{bmatrix}
$$

Given:

$$
P=
\begin{bmatrix}
0\\
2
\end{bmatrix}
$$

Calculate:

$$
x'=(0.7071)(0)+(-0.7071)(2)=-1.4142
$$

$$
y'=(0.7071)(0)+(0.7071)(2)=1.4142
$$

Therefore:

$$
\boxed{P'=(-1.4142,1.4142)}
$$

Unlike a 90° rotation, arbitrary rotations do not simply swap coordinates.

Sine and cosine allow us to calculate the new position for any angle.

## 9. Three-Dimensional Rotation Matrices

In three dimensions, we use 3×3 rotation matrices.

There is a separate basic rotation matrix for each coordinate axis.

### Rotation around Z

$$
R_z(\theta)=
\begin{bmatrix}
\cos\theta&-\sin\theta&0\\
\sin\theta&\cos\theta&0\\
0&0&1
\end{bmatrix}
$$

At +90°:

$$
R_z(90^\circ)=
\begin{bmatrix}
0&-1&0\\
1&0&0\\
0&0&1
\end{bmatrix}
$$

The coordinate transformation is:

$$
\boxed{(x,y,z)\rightarrow(-y,x,z)}
$$

**Example:**

$$
(4,-2,7)\rightarrow(2,4,7)
$$

Z remains unchanged.

### Rotation around X

$$
R_x(\theta)=
\begin{bmatrix}
1&0&0\\
0&\cos\theta&-\sin\theta\\
0&\sin\theta&\cos\theta
\end{bmatrix}
$$

At +90°:

$$
R_x(90^\circ)=
\begin{bmatrix}
1&0&0\\
0&0&-1\\
0&1&0
\end{bmatrix}
$$

The coordinate transformation is:

$$
\boxed{(x,y,z)\rightarrow(x,-z,y)}
$$

**Example:**

$$
(3,5,2)\rightarrow(3,-2,5)
$$

X remains unchanged.

### Rotation around Y

$$
R_y(\theta)=
\begin{bmatrix}
\cos\theta&0&\sin\theta\\
0&1&0\\
-\sin\theta&0&\cos\theta
\end{bmatrix}
$$

At +90°:

$$
R_y(90^\circ)=
\begin{bmatrix}
0&0&1\\
0&1&0\\
-1&0&0
\end{bmatrix}
$$

The coordinate transformation is:

$$
\boxed{(x,y,z)\rightarrow(z,y,-x)}
$$

**Example:**

$$
(6,2,5)\rightarrow(5,2,-6)
$$

Y remains unchanged.

### Important properties of rotation

A proper rotation:

- Preserves distances between points.
- Preserves angles between vectors.
- Does not change the size or shape of objects.
- Can be combined with other rotations.

Rotation order matters. Rotating around X and then Y generally produces a different result from rotating around Y and then X.

---

# Part IV — Homogeneous Transformations

## 10. Combining Rotation and Translation

So far, we have represented rotation and translation separately.

A transformed 3D point can be calculated as:

$$
\boxed{P'=RP+\mathbf{t}}
$$

Where:

- $P$ is the original point.
- $R$ is a 3×3 rotation matrix.
- $\mathbf{t}$ is a translation vector.
- $P'$ is the transformed point.

This means we rotate the point first and then translate it.

However, we can represent both operations using one matrix.

## 11. Homogeneous Coordinates

A standard 3D point contains three coordinates:

$$
P=
\begin{bmatrix}
x\\y\\z
\end{bmatrix}
$$

We extend the point by adding a fourth coordinate:

$$
P_h=
\begin{bmatrix}
x\\y\\z\\1
\end{bmatrix}
$$

This representation is called **homogeneous coordinates**.

The fourth coordinate is not an additional physical spatial dimension.

It allows translation to be expressed through matrix multiplication.

For ordinary 3D points, we use a final homogeneous coordinate of 1.

A direction vector can instead be represented with a final coordinate of 0, ensuring that translation does not affect its direction.

## 12. The 4×4 Transformation Matrix

A homogeneous rigid transformation is represented as:

$$
T=
\begin{bmatrix}
r_{11}&r_{12}&r_{13}&t_x\\
r_{21}&r_{22}&r_{23}&t_y\\
r_{31}&r_{32}&r_{33}&t_z\\
0&0&0&1
\end{bmatrix}
$$

The matrix has three important components:

**Upper-left 3×3 block**

Contains the rotation matrix.

**Rightmost column**

Contains translation values.

**Bottom row**

Allows the homogeneous-coordinate representation to work correctly.

The transformation is applied using:

$$
\boxed{P'_h=TP_h}
$$

### Identity matrix

An identity matrix leaves a vector unchanged when multiplied by it.

In three dimensions:

$$
I_3=
\begin{bmatrix}
1&0&0\\
0&1&0\\
0&0&1
\end{bmatrix}
$$

If we use the identity rotation inside a homogeneous matrix, the transformation performs translation without rotation.

### Example: Translation only

Given:

$$
P=
\begin{bmatrix}
2\\5\\1\\1
\end{bmatrix}
$$

And:

$$
T=
\begin{bmatrix}
1&0&0&3\\
0&1&0&-2\\
0&0&1&4\\
0&0&0&1
\end{bmatrix}
$$

Calculate:

$$
P'=TP
$$

The resulting coordinates are:

$$
\begin{aligned}
x'&=2+3=5\\
y'&=5-2=3\\
z'&=1+4=5\\
w'&=1
\end{aligned}
$$

Therefore:

$$
\boxed{P'=(5,3,5,1)}
$$

### Example: Rotation and translation

Suppose we rotate a point +90° around Z and then translate it by:

$$
\mathbf{t}=(3,2,1)
$$

Our transformation matrix becomes:

$$
T=
\begin{bmatrix}
0&-1&0&3\\
1&0&0&2\\
0&0&1&1\\
0&0&0&1
\end{bmatrix}
$$

Given:

$$
P=
\begin{bmatrix}
4\\2\\3\\1
\end{bmatrix}
$$

First row:

$$
x'=(0)(4)+(-1)(2)+(0)(3)+(3)(1)=1
$$

Second row:

$$
y'=(1)(4)+(0)(2)+(0)(3)+(2)(1)=6
$$

Third row:

$$
z'=(0)(4)+(0)(2)+(1)(3)+(1)(1)=4
$$

Fourth row:

$$
w'=1
$$

The result is:

$$
\boxed{P'=(1,6,4,1)}
$$

This is equivalent to performing the two operations separately:

1. Rotate $(4,2,3)$ into $(-2,4,3)$.
2. Translate by $(3,2,1)$ to obtain $(1,6,4)$.

---

# Part V — Coordinate Frames and Composition

## 13. Coordinate Frames

A **coordinate frame** consists of an origin and a set of coordinate axes.

Different objects or systems can use different frames.

For our rover, three important frames are:

**Camera frame (C)**

Describes positions measured relative to the ZED camera.

**Rover frame (R)**

Describes positions relative to the rover's body or chosen reference point.

**World frame (W)**

Describes positions in a common mapping coordinate system.

The same physical rock can have different numerical coordinates in each frame.

This does not mean the rock is moving. It means its position is being described relative to different origins and orientations.

## 14. Transformation Notation

We use subscripts to identify the coordinate frames involved.

$$
P_C
$$

means a point expressed in camera coordinates.

$$
P_R
$$

means the same point expressed in rover coordinates.

$$
P_W
$$

means the point expressed in world coordinates.

A transformation written as:

$$
T_{RC}
$$

converts coordinates **from Camera (C) to Rover (R)**.

Therefore:

$$
\boxed{P_R=T_{RC}P_C}
$$

Similarly:

$$
T_{WR}
$$

converts Rover coordinates into World coordinates.

Therefore:

$$
\boxed{P_W=T_{WR}P_R}
$$

The first subscript identifies the destination frame, and the second identifies the source frame.

## 15. Applying Multiple Transformations

Suppose our camera detects a point at:

$$
P_C=
\begin{bmatrix}
3\\1\\2\\1
\end{bmatrix}
$$

We want to find the point's position in world coordinates.

### Step 1: Camera to Rover

Our camera-to-rover transformation is:

$$
T_{RC}=
\begin{bmatrix}
0&-1&0&1\\
1&0&0&0\\
0&0&1&0\\
0&0&0&1
\end{bmatrix}
$$

This represents a +90° rotation around Z followed by a translation of 1 unit along rover X.

Calculate:

$$
P_R=T_{RC}P_C
$$

$$
P_R=
\begin{bmatrix}
(0)(3)+(-1)(1)+(1)(1)\\
(1)(3)\\
(1)(2)\\
1
\end{bmatrix}
$$

Therefore:

$$
\boxed{P_R=(0,3,2,1)}
$$

### Step 2: Rover to World

Suppose the rover has no additional rotation, but its origin is located at:

$$
(10,5,0)
$$

The rover-to-world transformation is:

$$
T_{WR}=
\begin{bmatrix}
1&0&0&10\\
0&1&0&5\\
0&0&1&0\\
0&0&0&1
\end{bmatrix}
$$

Calculate:

$$
P_W=T_{WR}P_R
$$

$$
P_W=
\begin{bmatrix}
0+10\\
3+5\\
2\\
1
\end{bmatrix}
$$

Therefore:

$$
\boxed{P_W=(10,8,2,1)}
$$

We have now converted a camera measurement into world coordinates.

## 16. Transformation Composition

Instead of applying transformations individually, we can combine their matrices.

We know:

$$
P_R=T_{RC}P_C
$$

And:

$$
P_W=T_{WR}P_R
$$

Substituting the first equation into the second:

$$
P_W=T_{WR}(T_{RC}P_C)
$$

Therefore:

$$
\boxed{P_W=T_{WR}T_{RC}P_C}
$$

We define the combined transformation:

$$
\boxed{T_{WC}=T_{WR}T_{RC}}
$$

This converts camera coordinates directly into world coordinates.

### Multiplying the two matrices

Our original matrices are:

$$
T_{WR}=
\begin{bmatrix}
1&0&0&10\\
0&1&0&5\\
0&0&1&0\\
0&0&0&1
\end{bmatrix}
$$

$$
T_{RC}=
\begin{bmatrix}
0&-1&0&1\\
1&0&0&0\\
0&0&1&0\\
0&0&0&1
\end{bmatrix}
$$

Calculate:

$$
T_{WC}=T_{WR}T_{RC}
$$

Recall that each output element is found by multiplying a row of the first matrix by a column of the second matrix.

For example, the first output element is:

$$
(1)(0)+(0)(1)+(0)(0)+(10)(0)=0
$$

The first row's fourth element is:

$$
(1)(1)+(0)(0)+(0)(0)+(10)(1)=11
$$

After computing all elements:

$$
\boxed{
T_{WC}=
\begin{bmatrix}
0&-1&0&11\\
1&0&0&5\\
0&0&1&0\\
0&0&0&1
\end{bmatrix}
}
$$

This is our combined camera-to-world transformation.

### Verifying the result

Use the original camera point:

$$
P_C=
\begin{bmatrix}
3\\1\\2\\1
\end{bmatrix}
$$

Calculate:

$$
P_W=T_{WC}P_C
$$

$$
P_W=
\begin{bmatrix}
(0)(3)+(-1)(1)+(11)(1)\\
(1)(3)+(5)(1)\\
(1)(2)\\
1
\end{bmatrix}
$$

Therefore:

$$
\boxed{P_W=(10,8,2,1)}
$$

This matches the result obtained when applying both transformations separately.

### Why does multiplication order matter?

With our column-vector convention, the rightmost transformation is applied first:

$$
P_W=T_{WR}T_{RC}P_C
$$

The camera-to-rover transformation happens before the rover-to-world transformation.

Reversing the order generally produces a different result.

The translation component also depends on the rotation and coordinate frame in which it is expressed.

Therefore, transformations must be composed in the correct order.

---

# Part VI — Practical Application

## 17. Applying Transformations to Point Clouds

A point cloud contains many 3D points.

Once we know the camera-to-world transformation, we can apply it to each point in the cloud.

For example:

$$
P_{W,i}=T_{WC}P_{C,i}
$$

Where $i$ identifies an individual point.

In principle, this operation is repeated for every valid point in the cloud.

If the camera's pose changes, the transformation must be updated accordingly.

### Connection to PCL and C++

The Point Cloud Library provides the `pcl::transformPointCloud()` function for transforming point clouds.

PCL commonly uses Eigen matrices to represent transformations.

Conceptually, the operation looks like:

```cpp
pcl::transformPointCloud(
    input_cloud,
    output_cloud,
    transformation
);
```

Here:

- `input_cloud` contains the original point measurements.
- `output_cloud` receives the transformed points.
- `transformation` is the appropriate transformation matrix.

The transformation must use the correct coordinate convention and frame relationship.

## 18. Application to Mars Rover Obstacle Avoidance

Our Mars Rover uses ZED stereo cameras to obtain depth measurements and point clouds.

A simplified mapping pipeline is:

1. The ZED camera detects points relative to its coordinate frame.
2. The system converts those measurements into the rover's coordinate frame.
3. The rover's estimated pose is used to convert the measurements into world coordinates.
4. The transformed points are assigned to positions in a terrain grid.
5. Terrain characteristics and obstacles can be evaluated within a consistent mapping frame.

This makes it possible to associate observations from different camera frames with their estimated locations in the environment.

### Why is coordinate alignment necessary?

Suppose a stationary rock appears at different positions relative to the camera while the rover moves.

Without correct transformations, the mapping system might treat those measurements as unrelated positions.

By transforming the observations into a common frame, the system can associate repeated measurements with the same physical region.

### Important engineering considerations

**1. Sensor mounting position**

The ZED camera may not be located at the rover's coordinate origin.

Its mounting position and orientation must be accounted for.

**2. Rover orientation**

The rover can turn, tilt or traverse uneven terrain.

These movements change how camera coordinates relate to world coordinates.

**3. Coordinate conventions**

Different libraries may use different axis orientations or handedness conventions.

The chosen conventions must be verified rather than assumed.

**4. Localization error**

Transformations are only as accurate as the estimated positions and orientations used to construct them.

Incorrect pose estimates can cause the resulting map to become misaligned.

**5. Transformation timing**

When processing live observations, the transformation should correspond to the rover's pose at the time the measurements were captured.

---

# Part VII — Review and Reference

## 19. Essential Formulas

### Translation

$$
\boxed{P'=P+\mathbf{t}}
$$

### 2D rotation

$$
\boxed{
R(\theta)=
\begin{bmatrix}
\cos\theta&-\sin\theta\\
\sin\theta&\cos\theta
\end{bmatrix}
}
$$

### Rotation around X

$$
\boxed{
R_x(\theta)=
\begin{bmatrix}
1&0&0\\
0&\cos\theta&-\sin\theta\\
0&\sin\theta&\cos\theta
\end{bmatrix}
}
$$

### Rotation around Y

$$
\boxed{
R_y(\theta)=
\begin{bmatrix}
\cos\theta&0&\sin\theta\\
0&1&0\\
-\sin\theta&0&\cos\theta
\end{bmatrix}
}
$$

### Rotation around Z

$$
\boxed{
R_z(\theta)=
\begin{bmatrix}
\cos\theta&-\sin\theta&0\\
\sin\theta&\cos\theta&0\\
0&0&1
\end{bmatrix}
}
$$

### Rotation followed by translation

$$
\boxed{P'=RP+\mathbf{t}}
$$

### Homogeneous transformation

$$
\boxed{
T=
\begin{bmatrix}
R&\mathbf{t}\\
0&0&0&1
\end{bmatrix}
}
$$

Here, $R$ is a 3×3 matrix and $\mathbf{t}$ is a 3×1 translation vector.

### Transformation composition

$$
\boxed{T_{WC}=T_{WR}T_{RC}}
$$

### Transforming from camera to world coordinates

$$
\boxed{P_W=T_{WC}P_C}
$$

## 20. Common Mistakes

**Mistake 1: Multiplying matrices in the wrong order**

Matrix multiplication is not generally commutative.

The correct transformation order must be maintained.

**Mistake 2: Confusing rotation and translation**

Rotation changes the orientation of a point relative to an axis.

Translation adds displacement.

They are distinct operations, even when combined into one matrix.

**Mistake 3: Assuming Z always represents height**

Different coordinate systems use different axis conventions.

The meaning of each coordinate must be verified.

**Mistake 4: Forgetting negative signs**

A positive 90° rotation generally causes one of the rotating coordinates to change sign.

The sign follows the chosen rotation axis and right-hand convention.

**Mistake 5: Treating homogeneous coordinates as four physical dimensions**

The fourth coordinate is a mathematical representation, not an additional physical spatial axis.

**Mistake 6: Forgetting the point's source frame**

A coordinate such as $(2,3,4)$ has no complete spatial meaning without knowing the reference frame in which it is expressed.

## 21. Practice Problems

### Problem A — Translation

Given:

$$
P=(4,2,7)
$$

And:

$$
\mathbf{t}=(-1,3,-2)
$$

Calculate $P'$.

### Problem B — 2D Rotation

Rotate:

$$
P=(3,1)
$$

By +90° around the origin.

### Problem C — 3D Rotation

Rotate:

$$
P=(5,2,4)
$$

By +90° around the Y-axis.

### Problem D — Homogeneous Transformation

Given:

$$
T=
\begin{bmatrix}
0&-1&0&2\\
1&0&0&3\\
0&0&1&-1\\
0&0&0&1
\end{bmatrix}
$$

And:

$$
P=
\begin{bmatrix}
3\\2\\5\\1
\end{bmatrix}
$$

Calculate $TP$.

### Answers

- **A:** $(3,5,5)$
- **B:** $(-1,3)$
- **C:** $(4,2,-5)$
- **D:** $(0,6,4,1)$

---

## 22. Key Takeaways

1. **Translation** moves points by adding displacement vectors.
2. **Rotation** changes orientation using angles and rotation matrices.
3. **Matrix multiplication** provides a systematic method for applying transformations.
4. **3D rotation matrices** describe rotations around the X, Y and Z axes.
5. **Homogeneous transformation matrices** combine rotation and translation into one 4×4 operation.
6. **Transformation composition** combines several coordinate-frame conversions into one matrix.
7. **Correct coordinate frames are essential** when converting live sensor observations into a consistent world map.

### Final Summary

Transformations allow a robotics system to convert measurements between different coordinate frames while preserving the physical relationships between points.

For Mars Rover autonomy, this mathematics is fundamental to locating camera-detected terrain and obstacles in the coordinate system used by the rover's mapping and navigation software.