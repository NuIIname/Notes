# Chapter 1 — Voxelization

**Topic:** Point-cloud processing  
**Resource:** [Interactive PCL Tutorial — Chapter 1](https://limhyungtae.github.io/pcl_tutorial/)

## 1. Definition

**Voxelization** is the process of dividing three-dimensional space into regularly sized volumetric cells called *voxels*.

A voxel is the 3D equivalent of a pixel, although it represents a volume rather than an area.

**Voxel-grid downsampling** reduces the number of points in a point cloud by representing the points inside each occupied voxel with one representative point.

The purpose is to reduce computational cost while preserving the approximate geometry of the original point cloud.

## 2. How voxel-grid downsampling works

1. Divide the point cloud's 3D space into equally sized cubic cells.
2. Determine which voxel contains each point based on its XYZ coordinates.
3. Group the points occupying the same voxel.
4. Calculate the centroid of each group.
5. Replace each occupied voxel's original points with one point located at that centroid.

Empty voxels do not produce output points.

### The centroid

The **centroid** is the arithmetic mean position of all points within a group.

For \(n\) points:

$$
C=\left(
\frac{\sum_{i=1}^{n}x_i}{n},
\frac{\sum_{i=1}^{n}y_i}{n},
\frac{\sum_{i=1}^{n}z_i}{n}
\right)
$$

**Example:**

Suppose three points occupy the same voxel:

- A = (2, 4, 6)
- B = (4, 6, 8)
- C = (6, 8, 10)

Their centroid is:

\[
C=(4,6,8)
\]

All three points are replaced by one representative point at `(4, 6, 8)`.

The centroid is **not necessarily the geometric center of the voxel**, nor does it need to coincide with an original point.

## 3. Leaf Size

**Leaf size** determines each voxel's dimensions along the X, Y and Z axes.

For example, a leaf size of `0.004 m` produces cubes measuring 4 mm along each axis.

- **Smaller leaf size:** More points retained, greater geometric detail, higher processing cost.
- **Larger leaf size:** Fewer points retained, reduced detail, generally lower processing cost.

A voxel grid with too large a leaf size may remove small geometric features that are important for downstream algorithms.

Leaf size must therefore be chosen according to the application's required resolution.

## 4. Tutorial observations

The interactive PCL demonstration provides CAD, indoor and outdoor point-cloud datasets.

These are different example datasets, not different point-cloud data structures.

In the Stanford bunny example shown:

- Original cloud: 1,889 points
- Filtered cloud: 1,811 points
- Reduction: approximately 4.1%
- Leaf size: 0.004 m

The *PT* display controls alter the visual size of rendered points, whereas the *leaf size* controls the actual voxel-grid filtering.

## 5. C++ implementation using PCL

The Point Cloud Library provides `pcl::VoxelGrid` for performing this operation.

The tutorial demonstrates the following operations:

- Create a `pcl::VoxelGrid<pcl::PointXYZ>` filter.
- Use `setInputCloud()` to specify the original point cloud.
- Use `setLeafSize()` to configure the voxel dimensions.
- Call `filter()` to generate the downsampled output cloud.

The resulting point cloud contains approximately one centroid for each occupied voxel.

## 6. Application to Mars Rover obstacle avoidance

The rover's ZED stereo cameras generate 3D measurements representing visible terrain and objects.

Voxel-grid downsampling could reduce the number of points that the obstacle avoidance system needs to process.

**Potential benefits:**

- Reduced memory consumption.
- Fewer points for subsequent geometric calculations.
- Faster downstream processing, depending on the algorithms used.

**Potential problems:**

- Small rocks or obstacles might lose geometric detail.
- Averages can obscure extreme height measurements.
- Excessive downsampling may affect terrain classification accuracy.

Voxel size should be selected with the rover's physical dimensions, obstacle-detection requirements and sensor resolution in mind.

Voxelization is not itself an obstacle-detection algorithm. It is a preprocessing technique that can help make later processing more efficient.

## 7. Key takeaway

**Voxel-grid downsampling trades geometric detail for computational efficiency by replacing all points in each occupied voxel with their centroid.**

The appropriate leaf size depends on how much geometric detail the application needs to preserve.