# DEBS ACM Team Running Notes

## Members

1. Prathmesh Sonawane
2. Owen Mariani
3. Tanish Bhowmick

## Problem Statement 

L-PFB is a manufacturing method where layers of fine metal powder are melted on top of each other to create objects. The problem is that in some cases, due to impurities in the material or simply calibration errors, the object can have high levels of porosity (or empty space) in some areas, leading the object to be defected. Though objects can be assessed after the L-PFB process finishes, a lot of time, energy, and material could have been saved by detecting defects and shutting down the process earlier. Thus, there is a need to implement real-time monitoring systems that can detect defects as they are made, resulting in the L-PFB process stopping as defects are found. Such a solution would inhibit a system from continuing the L-PBF process when a defect is already likely, leading to saved time and resources that can be spent to create a new, un-defected object. 

In short, we hope to create a solution that halts the L-PFB manufacturing process in real time if defects are found, saving valuable time and resources. Finding such a solution will save manufacturers using the L-PFB technique time, energy, and materials, decreasing costs that may then be sold to consumers for a lower price. Thus, both manufacturers and consumers of L-PFB products can benefit from such a solution. 

## Goal
Detect a high degree of porosity (amount of empty spaces left inside the material) during a simulation, which may severely impact the durability of the object being created.

## Given Data
Each optical tomography image contains, for each point **P = (x, y)**, the temperature **T(P)** at that point, measured layer by layer (**z**). In our input stream, we are given tiles for each layer, layer by layer, where each time contains points, each with a T(P).

## Proposed Solution/Processing Pipeline
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
