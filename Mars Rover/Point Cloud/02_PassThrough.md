# Chapter 2 — PassThrough Filtering

**Topic:** Point-cloud processing  
**Resource:** [Interactive PCL Tutorial — Chapter 2](https://limhyungtae.github.io/pcl_tutorial/)

## 1. Definition

**PassThrough filtering** is a point-cloud processing technique that selects or removes points based on whether their coordinate along a chosen axis falls within a specified interval.

The filter operates on one spatial axis at a time: X, Y, or Z.

Its primary purpose is to isolate a region of interest and discard points that are irrelevant to subsequent processing.

For example, an autonomous rover might use PassThrough filtering to ignore measurements outside its useful observation range.

## 2. How PassThrough filtering works

1. Select the spatial axis (X, Y, or Z) along which filtering will occur.
2. Define a minimum and maximum coordinate value.
3. Examine each point's coordinate along the selected axis.
4. Keep points whose coordinate falls within the interval.
5. Discard the remaining points.

Unlike voxelization, PassThrough filtering **does not average points or create centroids**. It simply retains or removes existing points.

### Mathematical representation

Suppose we filter along the Z-axis.

A point is retained when:

$$
z_{\min} \leq z_i \leq z_{\max}
$$

Where:

- $z_i$ = Z-coordinate of the current point.
- $z_{\min}$ = Minimum accepted Z-coordinate.
- $z_{\max}$ = Maximum accepted Z-coordinate.

The same principle applies to the X-axis and Y-axis.

### Example

Suppose the filter operates along Z with the interval:

$$
-2 \leq z \leq 0
$$

Consider the following points:

| Point | Coordinates (X, Y, Z) | Result |
|---|---|---|
| A | (1, 2, -1) | Keep |
| B | (3, 1, 2) | Remove |
| C | (2, 4, -3) | Remove |
| D | (5, 2, 0) | Keep |

Only points A and D are retained because their Z-coordinates lie within the specified interval.

## 3. Filter parameters

### Axis

The axis determines which coordinate is evaluated.

- **X-axis:** Filters points according to their X-coordinate.
- **Y-axis:** Filters points according to their Y-coordinate.
- **Z-axis:** Filters points according to their Z-coordinate.

The physical meaning of an axis depends on the coordinate system used by the sensor or application.

For example, Z may represent height in a Z-up coordinate system, but it may represent forward depth in a camera coordinate system.

### Minimum and maximum

These parameters define the accepted coordinate interval.

Narrower intervals retain fewer points when they exclude more of the cloud.

Wider intervals generally retain more points.

### Negative filtering

PassThrough filtering also supports an inverted selection mode.

When negative filtering is enabled, the algorithm **keeps points outside the specified interval** instead of inside it.

For the interval:

$$
-2 \leq z \leq 0
$$

Normal filtering retains points inside the interval.

Negative filtering retains points satisfying:

$$
z < -2 \quad \text{or} \quad z > 0
$$

This is useful when removing a particular spatial region rather than selecting it.

## 4. Tutorial observations

The interactive PCL demonstration allows filtering along the X, Y, or Z axes.

It also provides CAD, indoor, and outdoor example datasets.

In the outdoor example shown:

- Original point cloud: 124,668 points.
- Filtered point cloud: 100,623 points.
- Points removed: 24,045.
- Reduction: approximately 19.3%.
- Selected axis: Z.
- Minimum: -2 meters.
- Maximum: 0 meters.

The PT display controls affect the visual appearance of rendered points, not the filtering interval.

## 5. C++ implementation using PCL

The Point Cloud Library provides `pcl::PassThrough` to perform axis-aligned interval filtering.

Example:

```cpp
pcl::PassThrough<pcl::PointXYZ> pt;

pt.setInputCloud(src);

pt.setFilterFieldName("z");

pt.setFilterLimits(-2.0, 0.0);

pt.filter(*out);
```

### Explanation

**`pcl::PassThrough<pcl::PointXYZ> pt;`**

Creates a PassThrough filter that operates on XYZ point-cloud data.

**`pt.setInputCloud(src);`**

Specifies the input point cloud.

**`pt.setFilterFieldName("z");`**

Selects the Z-coordinate as the filtering field.

**`pt.setFilterLimits(-2.0, 0.0);`**

Configures the accepted coordinate interval.

**`pt.filter(*out);`**

Executes the filter and writes the resulting points to the output cloud.

Negative filtering can be enabled using:

```cpp
pt.setNegative(true);
```

This reverses which points are retained.

## 6. Application to Mars Rover obstacle avoidance

Our rover's ZED stereo cameras generate 3D point clouds containing terrain and environmental measurements.

PassThrough filtering could help remove irrelevant measurements before more computationally expensive processing.

### Potential applications

**1. Limit observation distance**

Exclude points beyond a specified distance along the camera's forward axis.

This can reduce processing on terrain too distant to be immediately relevant.

An axis-aligned filter is not equivalent to a true radial-distance filter.

**2. Limit vertical range**

Exclude measurements above or below a selected height interval.

For example, measurements far above the rover might be irrelevant to ground traversability.

This requires knowing how camera coordinates correspond to real-world height.

**3. Select a region of interest**

Restrict processing to a particular portion of the environment.

Multiple PassThrough filters could be applied sequentially to constrain several axes.

### Limitations

- Filtering is based on coordinate values, not object classification.
- An obstacle outside the selected interval will be discarded.
- Incorrect coordinate-system assumptions may remove important measurements.
- A single-axis filter cannot fully represent an arbitrary three-dimensional region.
- Filtering does not determine whether retained terrain is safe.

Therefore, filter boundaries must be selected according to the rover's sensing and navigation requirements.

## 7. Comparison with voxelization

| Property | Voxelization | PassThrough |
|---|---|---|
| Primary purpose | Downsample points | Select a spatial interval |
| Changes point coordinates | Yes, through centroids | No |
| Removes existing points | Yes | Yes |
| Uses voxel dimensions | Yes | No |
| Uses coordinate bounds | Internally for cells | Explicitly for filtering |
| Can reduce computation | Yes | Yes |

Both techniques can be used as preprocessing steps, but they solve different problems.

Voxelization reduces spatial redundancy, while PassThrough removes points outside a selected coordinate interval.

## 8. Key takeaway

**PassThrough filtering selects or removes points according to whether their coordinate along a chosen axis falls within a specified range.**

Unlike voxelization, it preserves the coordinates of retained points without averaging them.

For Mars Rover obstacle avoidance, it could help restrict point-cloud processing to relevant spatial regions, provided the chosen boundaries do not discard important obstacles.