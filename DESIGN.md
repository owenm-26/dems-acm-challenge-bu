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

The following "processing pipeline" is given to us by the DEBS 2025 website. 

    1. Saturation analysis: Within each tile, detect all points that surpass a threshold value of 65000.

    2. Windowing: For each tile, keep a window of the last three layers.

    3. Outlier analysis: Within each tile window, for each point P of the most recent layer:
        - Compute its local temperature deviation D as the absolute difference between:
            - The mean temperature T(P) of its close neighbors (Manhattan distance 0 ≤ d ≤ 2 across 3 layers).
            - The mean temperature of its outer neighbors (Manhattan distance 2 < d ≤ 4).
        - A point is classified as an outlier if D > 5000.

    4. Outlier clustering: Using the outliers computed for the last received layer, find clusters of nearby outliers using DBScan, with the Euclidean distance between points as the distance metric.

    -----------
    For each input (tile) received from the stream, your solution should return:

    The number of saturated points.
    The centroid (x, y coordinates) and size (number of points) of the top 10 largest clusters.

In summary, by finding the number of saturated points (or those that pass our temperature threshold) and our cluster centers/sizes for points that are outliers according to step 3 in our processing pipeline, defects can be detected. Our understanding is that thermal outliers are directly linked to defects. Thus, by finding thermal outliers, we can find when defects are likely to occur. 

Though DEBS gives us a sample solution written in Python, our group has decided to use Kafka and Flink to funnel and manipulate our streaming data. These tools have been taught in class and have the necessary functionality to aggregate/manipulate streaming data with a variety of operators. 



### Technical Tools for Each Step: 

**Note: The DEBS sample solution was not working until 2/18 when the DEBS organizers pushed a fix. This limited our testing capabilities for the design document.**

1. To detect all points that surpass a threshold for each tile, we can use the filter operator and filter out data points that don't meet our threshold. Thus, all output data would pass our threshold and could be counted for each tile, giving us our first out put requirement. After, we can simply key-by the tileID and run a count function. 

2. Flink/Kafka automatically supports the usage of windows. For this step, we could use sliding windows after key-by using tileID of size three layers and step one layer. 

3. Using our window of size 3, we can then compute the average temperature for each point in each tile for the latest layer, finding the local and outer averages. Then, we can simply see if the deviation is large enough, and use a filter function to pass those points on. 

4. Lastly, with all the deviated points, we can keyby tileID, plot those points using DBScan, find our top 10 clusters, and feed those to our sink. 

## Output
For each input **tile** received from the stream, the solution should return:
1. The **number of saturated points**.
2. The **centroid** (**x, y** coordinates) and **size** (number of points) of the **top 10 largest clusters**.

## Source of Error
The system receives a stream of optical tomography images indicating the **temperature of the powder bed** for each layer during the manufacturing process. It must compute the **probability of defects** in some areas, enabling **rapid reactions** in case of problems.

## Experimental Plan
### Data & Simulation:
- Use synthetic datasets and historical optical tomography images to simulate defect scenarios.
- Inject controlled defects into the data stream to validate threshold filtering, windowing, outlier detection, and clustering.

### Performance & Fault Testing:
- Measure throughput and latency using both local setups and a Kubernetes deployment.
- Deploy our solution via the DEBS competition’s pipeline, which benchmarks Kubernetes clusters and simulates network and pod failures.

### Monitoring & Metrics:
- Collect system metrics (CPU, memory, throughput, latency) using Kafka/Flink built-in tools
- Validate detection accuracy by comparing outputs (saturated point counts, cluster centroids/sizes) against ground-truth data.

### Deployment Pipeline:
- Utilize the DEBS deployment pipeline to upload our Kubernetes clusters, enabling automated benchmarking and fault injection tests (network and pod failures).
- Run iterative experiments through this pipeline to fine-tune performance, scalability, and resilience under realistic conditions.

This plan will confirm our hypotheses by ensuring the system accurately detects defects while maintaining performance and scalability under realistic operational conditions.
## Success Indicators
By the conclusion of the semester, this project will be able to performantly handle the task stated above when deployed as a task in the Kubernetes cluster. Our markers of progress will be the following: 

#### Progress Markers 
For what "performantly" and "unperformantly" mean please refer to the **Metrics** section. In general "performantly means that the solution has both a competitive latency and throughput in relation to the other competitors and is able to replicate those scores repeatedly.

1. Basic Solution that accomplishes the task locally (unperformantly)
The system is setup such that it can take in images and output confirmation that it has received each image
    * The solution is able to correctly identify high saturation points in images (check against sample solution)
    * The solution is able to correctly compute outliers (check against sample solution)
    * The solution is able to correctly cluster those outliers (check against sample solution)
2. Basic Solution that accomplishes the task unperformantly in the deployed environment
3. Solution that outperforms the template solution in the deployed environment
4. Solutions that perform increasingly well on the leaderboard and have better metrics than previous runs in the deployed environment
5. Solution is cleaned up and scores well in the following metrics

#### Metrics 
Our measures for success are directly tied to the metrics by which our project will be evaluated: **throughput and latency**. However, even if these two metrics are the most important, we aim to also keep in mind the following characteristics as nice-to-haves:
* **Configurability**: the ability to input different parameters to optimize the solution for workloads of different characterstics or clusters with different amounts of resources
* **Horizontal scalability**: the ability to enhance the performance of the solution by adding more hardware via efficient paralellization
* **Operational Reliability/Resilience**: the solution is able to overcome network and pod failures gracefully 
* **Accessibility of source code**: there is good documentation and guiding tutorials for how to best use the solution
* integration with standard (tools/protocols)
* **Security Mechanisms**: protections against malicious payloads being uploaded as injections 
* **Deployment Support**: ability to and assistance in running the solution on premise for those who want to manage their own solution
* **Portability/Maintainability**: The solution can be used in multiple environments and on multiple platforms. It is also well structures/commented for long term maintainability
* **Support of Special Hardware** (e.g., FPGAs, GPUs, SDNs, ...)

## Task Assignment

Specific Tasks to be completed:
- Outlier Detection & Clustering
- Monitoring & Fault Tolerance:
- Deployment Pipeline Integration:
- Validation & Testing:


Tasks will be divided so that ingestion, detection, and monitoring can be developed in parallel once initial setup is complete. 
Some tasks are dependent on each other:
- Data ingestion must be completed before outlier detection.
- Deployment integration depends on functional pipeline components.
- Validation runs iteratively across modules.

Although we all have similar strengths and weaknesses as students, we’ll distribute tasks based on interest and maintain regular coordination to ensure smooth and efficient progress throughout the project.