# Chapter 4 — Statistical Outlier Removal (SOR)

**Topic:** Point-Cloud Processing, Statistics, and Noise Filtering

**Primary Resource:** [Interactive PCL Tutorial — Statistical Outlier Removal](https://limhyungtae.github.io/pcl_tutorial/statistical-outlier-removal)

**Additional Resource:** [PCL — Statistical Outlier Removal Documentation](https://pointclouds.org/documentation/tutorials/statistical_outlier.html)

---

## 1. Definition

**Statistical Outlier Removal (SOR)** is a point-cloud filtering algorithm that identifies and removes points whose average distance to their neighboring points is unusually large compared with the rest of the point cloud.

An **outlier** is a measurement that differs significantly from the expected pattern of observations.

In a point cloud, outliers often appear as isolated points located far from surrounding surfaces.

Outliers can result from:

- Sensor measurement errors.
- Inaccurate depth estimates.
- Noise in the measurement process.
- Reflections or difficult surface conditions.
- Other irregularities in sensor data.

**Purpose:** Remove potentially erroneous measurements while preserving the underlying geometry of the point cloud.

Unlike voxelization, Statistical Outlier Removal does not average groups of points into new centroids. Instead, it decides which original points to retain or discard.

## 2. How Statistical Outlier Removal Works

SOR evaluates how densely each point is surrounded by other points.

The algorithm follows these steps:

1. For each point, identify its **K nearest neighbors**.
2. Calculate the distance from the point to each neighbor.
3. Calculate the average neighbor distance for that point.
4. Repeat this process for the entire point cloud.
5. Calculate the global mean and standard deviation of those average distances.
6. Establish a statistical threshold.
7. Classify points whose average neighbor distance exceeds the threshold as outliers.
8. Remove the outliers and retain the remaining points.

### Inliers and Outliers

**Inlier:** A point whose average neighbor distance falls within the accepted statistical threshold.

**Outlier:** A point whose average neighbor distance exceeds the threshold.

An inlier is not necessarily a correct measurement, and an outlier is not necessarily an incorrect measurement. These classifications describe statistical behavior, not absolute measurement accuracy.

---

# Part I — Mathematical Foundations

## 3. Euclidean Distance

Before identifying nearby points, we need a way to calculate the distance between two points in three-dimensional space.

For two points:

$$
A=(x_1,y_1,z_1)
$$

$$
B=(x_2,y_2,z_2)
$$

Their **Euclidean distance** is:

$$
d(A,B)=\sqrt{(x_2-x_1)^2+(y_2-y_1)^2+(z_2-z_1)^2}
$$

This is an extension of the Pythagorean theorem into three dimensions.

### Example

Given:

$$
A=(1,2,3)
$$

$$
B=(4,6,3)
$$

Calculate:

$$
d(A,B)=\sqrt{(4-1)^2+(6-2)^2+(3-3)^2}
$$

$$
=\sqrt{9+16+0}
$$

$$
=\sqrt{25}
$$

$$
\boxed{d(A,B)=5}
$$

If the coordinates are measured in meters, the distance is 5 meters.

SOR uses distances like these to evaluate how isolated a point is from its neighbors.

## 4. K-Nearest Neighbors (KNN)

**K-nearest neighbors** refers to the K points closest to a selected point according to a distance metric.

Here, K represents the number of neighbors being examined.

For example:

- K = 5 means examine the five nearest neighbors.
- K = 10 means examine the ten nearest neighbors.
- K = 30 means examine the thirty nearest neighbors.

The selected point itself is excluded when calculating its neighbor distances.

### Why use multiple neighbors?

Consider two situations:

**Situation A:** A point belongs to a dense surface, and many other points are located nearby.

**Situation B:** A point is isolated from the main surface, with its nearest neighbors much farther away.

By comparing average neighbor distances, SOR can distinguish relatively dense regions from unusually isolated points.

Using multiple neighbors reduces the algorithm's dependence on one particular neighboring point.

### Mean Neighbor Distance

For point $P_i$, suppose its K nearest-neighbor distances are:

$$
d_1,d_2,\ldots,d_K
$$

Its mean neighbor distance is:

$$
\boxed{\bar d_i=\frac{1}{K}\sum_{j=1}^{K}d_j}
$$

Where:

- $\bar d_i$ = Average neighbor distance for point $i$.
- $K$ = Number of neighbors.
- $d_j$ = Distance to the $j$th nearest neighbor.

### Example

Suppose a point has four nearest neighbors at distances:

$$
0.10,\quad0.15,\quad0.20,\quad0.15
$$

Its average neighbor distance is:

$$
\bar d_i=\frac{0.10+0.15+0.20+0.15}{4}
$$

$$
\boxed{\bar d_i=0.15\text{ m}}
$$

SOR repeats this calculation for every point.

**Important:** KNN is also used in machine learning. However, SOR uses nearest-neighbor distances for statistical filtering, not for predicting labels or regression values.

## 5. Arithmetic Mean

The **arithmetic mean** is the average of a collection of numerical values.

For $N$ values:

$$
x_1,x_2,\ldots,x_N
$$

The mean is:

$$
\boxed{\mu=\frac{1}{N}\sum_{i=1}^{N}x_i}
$$

Where $\mu$ (mu) represents the population mean.

In SOR, we calculate the mean of the **per-point average neighbor distances** across the point cloud.

We are not averaging the XYZ coordinates.

### Example

Suppose four points have average neighbor distances:

$$
0.10,\quad0.10,\quad0.30,\quad0.30
$$

Their global mean is:

$$
\mu=\frac{0.10+0.10+0.30+0.30}{4}
$$

$$
\boxed{\mu=0.20\text{ m}}
$$

This gives us a reference value for the typical average neighbor distance in the dataset.

## 6. Variance and Standard Deviation

The mean tells us the center of a collection of values.

However, it does not tell us how spread out those values are.

For this, we use **variance** and **standard deviation**.

### Variance

Variance measures the average squared difference between individual values and their mean.

The population variance formula is:

$$
\boxed{\sigma^2=\frac{1}{N}\sum_{i=1}^{N}(x_i-\mu)^2}
$$

Where:

- $\sigma^2$ = Population variance.
- $N$ = Number of observations.
- $x_i$ = Individual observation.
- $\mu$ = Population mean.

Squaring the differences prevents positive and negative deviations from canceling.

### Standard Deviation

Standard deviation is the square root of variance.

$$
\boxed{\sigma=\sqrt{\frac{1}{N}\sum_{i=1}^{N}(x_i-\mu)^2}}
$$

Standard deviation describes the spread of observations in the original units.

A small standard deviation means the values are relatively close to the mean.

A large standard deviation means the values are more widely distributed.

### Worked Example

Using our previous values:

$$
0.10,\quad0.10,\quad0.30,\quad0.30
$$

**Step 1 — Find the mean**

$$
\mu=0.20
$$

**Step 2 — Subtract the mean**

$$
-0.10,\quad-0.10,\quad0.10,\quad0.10
$$

**Step 3 — Square the differences**

$$
0.01,\quad0.01,\quad0.01,\quad0.01
$$

**Step 4 — Calculate variance**

$$
\sigma^2=\frac{0.01+0.01+0.01+0.01}{4}
$$

$$
\boxed{\sigma^2=0.01\text{ m}^2}
$$

**Step 5 — Calculate standard deviation**

$$
\sigma=\sqrt{0.01}
$$

$$
\boxed{\sigma=0.10\text{ m}}
$$

Thus, our average neighbor distances have a mean of 0.20 m and a standard deviation of 0.10 m.

---

# Part II — Statistical Filtering

## 7. The SOR Threshold

After calculating the mean and standard deviation of the average neighbor distances, SOR establishes a threshold.

The threshold formula is:

$$
\boxed{D_{\text{threshold}}=\mu+\alpha\sigma}
$$

Where:

- $D_{\text{threshold}}$ = Maximum accepted mean neighbor distance.
- $\mu$ = Global mean of the per-point average neighbor distances.
- $\sigma$ = Standard deviation of those averages.
- $\alpha$ = User-selected standard deviation multiplier.

A point is classified as an outlier when:

$$
\boxed{\bar d_i>D_{\text{threshold}}}
$$

Otherwise, the point is retained as an inlier.

### Threshold Example

Suppose:

$$
\mu=0.20\text{ m}
$$

$$
\sigma=0.10\text{ m}
$$

$$
\alpha=2
$$

Then:

$$
D_{\text{threshold}}=0.20+2(0.10)
$$

$$
\boxed{D_{\text{threshold}}=0.40\text{ m}}
$$

A point with an average neighbor distance of 0.50 m would be classified as an outlier.

A point with an average neighbor distance of 0.30 m would be retained.

### Why use standard deviation?

A fixed distance threshold might be appropriate for one dataset but unsuitable for another.

SOR uses the statistical distribution of neighbor distances to determine what constitutes an unusually isolated point.

This makes the threshold relative to the characteristics of the input cloud.

However, the method assumes that the distribution of average neighbor distances can be reasonably characterized by its mean and standard deviation.

That assumption may not work equally well for every environment.

## 8. Standard Deviation Multiplier

The multiplier $\alpha$ controls how permissive the filtering algorithm is.

Using:

$$
\mu=0.20,\qquad\sigma=0.05
$$

| Multiplier | Threshold |
|---|---|
| 0.5 | 0.225 m |
| 1.0 | 0.250 m |
| 2.0 | 0.300 m |
| 3.0 | 0.350 m |

### Smaller Multiplier

A smaller multiplier produces a lower threshold.

Consequently:

- More points are classified as outliers.
- Filtering becomes more aggressive.
- More potentially useful points may be removed.

### Larger Multiplier

A larger multiplier produces a higher threshold.

Consequently:

- Fewer points are classified as outliers.
- Filtering becomes more permissive.
- More noisy measurements may remain.

**Important distinction:** Changing the multiplier does not change the calculated mean or standard deviation of the existing input dataset. It changes the threshold derived from them.

## 9. The Mean K Parameter

The **Mean K** parameter specifies how many nearest neighbors are examined for each point.

For example:

```cpp
sor.setMeanK(30);
```

This tells PCL to calculate each point's average distance to its 30 nearest neighbors.

### Smaller K

A smaller K evaluates a more local neighborhood.

Possible effects include:

- Greater sensitivity to small-scale spatial variation.
- Greater influence from individual neighboring points.
- Less work per neighborhood query.

### Larger K

A larger K evaluates a broader neighborhood.

Possible effects include:

- More stable neighborhood averages in some datasets.
- Increased computational work.
- Greater sensitivity to differences between dense and sparse regions or between surfaces and boundaries.

**Neither larger nor smaller K is automatically better.**

The appropriate value depends on point density, sensor characteristics, obstacle size and filtering requirements.

---

# Part III — PCL Implementation

## 10. Statistical Outlier Removal in C++

The Point Cloud Library provides:

```cpp
pcl::StatisticalOutlierRemoval<pcl::PointXYZ>
```

This class implements statistical outlier filtering for XYZ point clouds.

A basic example is:

```cpp
#include <pcl/point_cloud.h>
#include <pcl/point_types.h>
#include <pcl/filters/statistical_outlier_removal.h>

// Assume input_cloud already contains valid points.

pcl::PointCloud<pcl::PointXYZ>::Ptr filtered_cloud(
    new pcl::PointCloud<pcl::PointXYZ>
);

pcl::StatisticalOutlierRemoval<pcl::PointXYZ> sor;

sor.setInputCloud(input_cloud);

sor.setMeanK(30);

sor.setStddevMulThresh(1.0);

sor.filter(*filtered_cloud);
```

### Explanation

**`pcl::StatisticalOutlierRemoval<pcl::PointXYZ> sor;`**

Creates the statistical filtering object.

**`sor.setInputCloud(input_cloud);`**

Specifies which point cloud will be processed.

**`sor.setMeanK(30);`**

Uses the 30 nearest neighbors when calculating each point's average neighbor distance.

**`sor.setStddevMulThresh(1.0);`**

Sets the statistical threshold multiplier to 1.

**`sor.filter(*filtered_cloud);`**

Executes the filtering algorithm and writes the retained points to the output cloud.

### Extracting Outliers

PCL also supports negative filtering.

```cpp
sor.setNegative(true);
sor.filter(*removed_cloud);
```

This produces the points classified as outliers rather than the retained inliers.

The output points retain their original coordinates; SOR is selecting points, not generating centroids.

## 11. Interactive Tutorial Parameters

The browser-based PCL tutorial lets users experiment with:

**Mean K**

Controls the number of neighboring points used for distance calculations.

**Standard Deviation Multiplier**

Controls how far above the global mean a point's average neighbor distance may lie before rejection.

**Input Dataset**

The tutorial provides different examples, including CAD, indoor and outdoor point clouds.

**Inlier / Outlier Visualization**

The demonstration distinguishes points retained by the filter from those rejected.

### Suggested Experiments

1. Keep Mean K fixed and reduce the standard deviation multiplier.
2. Observe whether the number of rejected points increases.
3. Increase the multiplier and compare the results.
4. Keep the multiplier fixed and change Mean K.
5. Compare how filtering affects different dataset types.

The original C++ tutorial also provides a separate implementation in `lec07_sor.cpp`, which can be studied when learning the PCL API.

---

# Part IV — Mars Rover Applications

## 12. Why SOR Matters for Obstacle Avoidance

Our rover's ZED stereo cameras generate 3D point-cloud measurements from estimated depth.

Some measurements may be inaccurate because of stereo-matching limitations, poor texture, reflections or other sensor conditions.

If erroneous measurements are incorporated into the traversability map, they could lead to incorrect obstacle classifications.

For example, a noisy depth measurement might appear to represent a tall object protruding from otherwise flat ground.

SOR could potentially remove such isolated measurements before terrain-cost calculations.

### Potential Benefits

- Reduces isolated measurement noise.
- Improves the quality of data used for surface analysis.
- May reduce false obstacle detections.
- Provides cleaner inputs for subsequent terrain-processing algorithms.

### Important Limitation

**A statistically isolated point is not necessarily an erroneous point.**

A small rock, narrow protrusion or edge could produce measurements that differ from nearby terrain.

Aggressive filtering could remove genuine obstacle evidence.

This is particularly relevant because our rover must recognize relatively small hazards.

## 13. Relationship to Point Density

Point clouds are not always sampled uniformly.

Point density can vary because of:

- Distance from the sensor.
- Surface orientation.
- Occlusion.
- Sensor measurement quality.
- Environmental characteristics.

SOR compares neighbor distances using statistics calculated across the input cloud.

This means that sparse but valid regions may be more likely to be classified as outliers, particularly when the dataset contains much denser regions elsewhere.

For a rover, this creates a potential problem when comparing nearby terrain with distant terrain.

The filter must be tested under realistic sensor conditions.

## 14. Relationship to Trough Detection

Our obstacle avoidance work includes detecting dangerous drops and troughs.

Statistical outlier removal may help eliminate erroneous depth measurements that could otherwise distort terrain-height calculations.

However, filtering also creates a risk:

A small number of valid points near a terrain discontinuity may be rejected as outliers.

**Removing points does not establish that the remaining region is empty or that a trough exists.**

Missing data must still be distinguished from confirmed terrain hazards.

This relates directly to our proposed approach of retaining unknown grid cells when insufficient reliable measurements are available.

## 15. Real-Time Processing Considerations

SOR requires identifying nearest neighbors and calculating distances across the point cloud.

For large clouds, these operations can be computationally expensive.

Efficient spatial-search structures, such as KD-trees, can accelerate nearest-neighbor queries.

However, the overall cost still depends on:

- Number of input points.
- Mean K setting.
- Data distribution.
- Search implementation.
- CPU/GPU processing architecture.

Our rover may need to evaluate whether SOR provides enough improvement in measurement quality to justify its runtime cost.

A complete implementation should be benchmarked using representative ZED data.

---

# Part V — Comparison and Review

## 16. Comparison with Earlier Chapters

| Algorithm | Main purpose | Method |
|---|---|---|
| Voxelization | Reduce point count | Replace points inside voxels with centroids |
| PassThrough | Select spatial region | Keep or discard points by coordinate interval |
| Transformations | Change coordinate frame | Apply rotation and translation |
| Statistical Outlier Removal | Remove statistical noise | Compare nearest-neighbor distances against a threshold |

These algorithms perform different operations and can be combined into a processing pipeline.

The order in which they are applied matters.

For example, voxelization changes point density, which may affect the neighbor-distance statistics used by SOR.

## 17. Common Mistakes

**Mistake 1: Assuming outliers are always erroneous**

Outliers are statistically unusual measurements. Some may represent real environmental features.

**Mistake 2: Confusing Mean K with the global mean**

Mean K is the number of nearest neighbors examined per point.

The global mean is calculated afterward from the average neighbor distances.

**Mistake 3: Assuming a larger standard deviation multiplier removes more points**

A larger positive multiplier increases the threshold and generally retains more points.

**Mistake 4: Assuming SOR changes the position of retained points**

SOR selects or removes original measurements. It does not average point positions.

**Mistake 5: Ignoring changes in point density**

A region of valid sparse measurements may appear statistically unusual compared with a denser region.

**Mistake 6: Treating filtered-out measurements as evidence of empty terrain**

A removed measurement does not demonstrate that a region is physically empty.

---

## 18. Essential Formulas

### Euclidean Distance

$$
\boxed{
d(A,B)=\sqrt{(x_2-x_1)^2+(y_2-y_1)^2+(z_2-z_1)^2}
}
$$

### Mean Neighbor Distance

$$
\boxed{
\bar d_i=\frac{1}{K}\sum_{j=1}^{K}d_j
}
$$

### Global Mean

$$
\boxed{
\mu=\frac{1}{N}\sum_{i=1}^{N}\bar d_i
}
$$

### Population Variance

$$
\boxed{
\sigma^2=\frac{1}{N}\sum_{i=1}^{N}(\bar d_i-\mu)^2
}
$$

### Standard Deviation

$$
\boxed{
\sigma=\sqrt{\frac{1}{N}\sum_{i=1}^{N}(\bar d_i-\mu)^2}
}
$$

### SOR Threshold

$$
\boxed{
D_{\text{threshold}}=\mu+\alpha\sigma
}
$$

### Outlier Condition

$$
\boxed{
\bar d_i>D_{\text{threshold}}
}
$$

---

## 19. Practice Problems

### Problem A — Mean Neighbor Distance

A point has four nearest-neighbor distances:

$$
0.10,\quad0.20,\quad0.15,\quad0.15
$$

Calculate its mean neighbor distance.

### Problem B — Mean and Standard Deviation

Four points have average neighbor distances:

$$
0.10,\quad0.10,\quad0.30,\quad0.30
$$

Calculate the population mean and standard deviation.

### Problem C — Threshold

Given:

$$
\mu=0.20,\quad\sigma=0.05,\quad\alpha=2
$$

Calculate the SOR threshold.

Would a point with an average neighbor distance of 0.35 m be retained?

### Problem D — Rover Application

A point cloud contains a small rock surrounded by flat terrain.

Explain why an aggressive SOR filter could be dangerous for obstacle detection.

### Answers

- **A:** 0.15 m.
- **B:** Mean = 0.20 m, standard deviation = 0.10 m.
- **C:** Threshold = 0.30 m. The point at 0.35 m would be classified as an outlier.
- **D:** Sparse but valid rock measurements could be removed, potentially hiding a genuine obstacle.

---

## 20. Key Takeaways

1. **Statistical Outlier Removal identifies unusually isolated points by examining nearest-neighbor distances.**
2. Mean K controls the number of neighboring points considered.
3. The global mean and standard deviation describe the distribution of average neighbor distances.
4. The standard deviation multiplier controls how aggressively points are filtered.
5. SOR removes points without creating new centroid positions.
6. Statistical outliers are not necessarily erroneous measurements.
7. For rover obstacle avoidance, SOR may reduce sensor noise but must be tested to ensure genuine hazards are preserved.

### Final Summary

Statistical Outlier Removal is a statistical preprocessing technique that identifies potentially noisy point-cloud measurements using the distances between points and their nearest neighbors.

The algorithm compares each point's mean neighbor distance with a threshold calculated from the global distribution of neighbor distances.

For Mars Rover autonomy, SOR may help improve the quality of sensor measurements before traversability analysis. However, its parameters must be selected carefully because valid terrain features and small obstacles can also appear statistically unusual.