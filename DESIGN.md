# DEBS ACM Team Running Notes

## Goal
Detect a high degree of porosity (amount of empty spaces left inside the material) during a simulation, which may severely impact the durability of the object being created.

## Given Data
Each optical tomography image contains, for each point **P = (x, y)**, the temperature **T(P)** at that point, measured layer by layer (**z**). In our input stream, we are given tiles, layer by layer.

## Processing Pipeline
1. **Saturation analysis**: Within each tile, detect all points that surpass a threshold value of **65000**.
2. **Windowing**: For each tile, keep a window of the last three layers.
3. **Outlier analysis**: Within each tile window, for each point **P** of the most recent layer:
   - Compute its local temperature deviation **D** as the absolute difference between:
     - The mean temperature **T(P)** of its close neighbors (Manhattan distance **0 ≤ d ≤ 2** across 3 layers).
     - The mean temperature of its outer neighbors (Manhattan distance **2 < d ≤ 4**).
   - A point is classified as an **outlier** if **D > 5000**.
4. **Outlier clustering**: Using the outliers computed for the last received layer, find clusters of nearby outliers using **DBScan**, with the Euclidean distance between points as the distance metric.

## Output
For each input **tile** received from the stream, the solution should return:
1. The **number of saturated points**.
2. The **centroid** (**x, y** coordinates) and **size** (number of points) of the **top 10 largest clusters**.

## Source of Error
The system receives a stream of optical tomography images indicating the **temperature of the powder bed** for each layer during the manufacturing process. It must compute the **probability of defects** in some areas, enabling **rapid reactions** in case of problems.
