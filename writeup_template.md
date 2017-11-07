## Project: Perception Pick & Place

---

# Required Steps for a Passing Submission:
1. Extract features and train an SVM model on new objects.
2. Write a ROS node and subscribe to `/pr2/world/points` topic. 
3. Use filtering and RANSAC plane fitting to isolate the objects of interest from the rest of the scene.
4. Apply Euclidean clustering to create separate clusters for individual items.
5. Perform object recognition on these objects and assign them labels (markers in RViz).
6. Calculate the centroid (average in x, y and z) of the set of points belonging to that each object.
7. Create ROS messages containing the details of each object (name, pick_pose, etc.) and write these messages out to `.yaml` files, one for each of the 3 scenarios (`test1-3.world` in `/pr2_robot/worlds/`). 
8. Submit a link to your GitHub repo for the project or the Python code for your perception pipeline and your output.

## [Rubric](https://review.udacity.com/#!/rubrics/1067/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

### Exercise 1, 2 and 3 pipeline implemented
#### 1. Complete Exercise 1 steps. Pipeline for filtering and RANSAC plane fitting implemented.

In the callback function, named as pcl_callback(located No.50 line in the project_template.py), I implemented the folllowing steps.

* Convert ROS message to PCL data by helper function (No.54 line in the project_template.py)
* Remove Outlier point cloud data by PCL’s StatisticalOutlierRemoval filter, which computes the distance to all of its neighbors, and then calculates a mean distance. By assuming a Gaussian distribution, the points outside of an interval defined by the distances mean+standard deviation are considered to be outliers and removed.(No.56-66 lines in the project_template.py)
* Downsample the point cloud data by Voxel Grid technique, with Leaf size at 0.01. (No.68-76 lines in the project_template.py)
* Set Passthrough filter align with Z axis and the range min 0.6 - max 1.1.(No.78-89 lines in the project_template.py)
* Use RANSAC segmentation with max distance = 0.03 to recognize each object and table separately. (No.91-100 lines in the project_template.py)


#### 2. Complete Exercise 2 steps: Pipeline including clustering for segmentation implemented.  

    # TODO: Extract inliers and outliers
    # Call the segment function to obtain set of inlier indices and model coefficients
    inliers, coefficients = seg.segment()
    # Extract inliers
    pcl_cloud_objects = cloud_filtered.extract(inliers, negative=True)
    pcl_cloud_table = cloud_filtered.extract(inliers, negative=False)

    # TODO: Euclidean Clustering
    white_cloud = XYZRGB_to_XYZ(pcl_cloud_objects)
    tree = white_cloud.make_kdtree()

    # TODO: Create Cluster-Mask Point Cloud to visualize each cluster separately
    # Create a cluster extraction object
    ec = white_cloud.make_EuclideanClusterExtraction()
    # Set tolerances for distance threshold
    # as well as minimum and maximum cluster size (in points)
    # NOTE: These are poor choices of clustering parameters
    # Your task is to experiment and find values that work for segmenting objects.
    ec.set_ClusterTolerance(0.015)
    ec.set_MinClusterSize(200)
    ec.set_MaxClusterSize(1500)
    # Search the k-d tree for clusters
    ec.set_SearchMethod(tree)
    # Extract indices for each of the discovered clusters
    cluster_indices = ec.Extract()

    #Assign a color corresponding to each segmented object in scene
    cluster_color = get_color_list(len(cluster_indices))

    color_cluster_point_list = []

    for j, indices in enumerate(cluster_indices):
        for i, indice in enumerate(indices):
            color_cluster_point_list.append([white_cloud[indice][0],
                                            white_cloud[indice][1],
                                            white_cloud[indice][2],
                                            rgb_to_float(cluster_color[j])])

    #Create new cloud containing all clusters each with unique color
    cluster_cloud = pcl.PointCloud_PointXYZRGB()
    cluster_cloud.from_list(color_cluster_point_list)

    # TODO: Convert PCL data to ROS messages
    ros_cloud_objects = pcl_to_ros(pcl_cloud_objects)
    ros_cloud_table = pcl_to_ros(pcl_cloud_table)
    ros_cluster_cloud = pcl_to_ros(cluster_cloud)

    # TODO: Publish ROS messages
    pcl_objects_pub.publish(ros_cloud_objects)
    pcl_table_pub.publish(ros_cloud_table)
    pcl_cluster_pub.publish(ros_cluster_cloud)


#### 2. Complete Exercise 3 Steps.  Features extracted and SVM trained.  Object recognition implemented.


![demo-1](https://user-images.githubusercontent.com/20687560/28748231-46b5b912-7467-11e7-8778-3095172b7b19.png)

### Pick and Place Setup

#### 1. For all three tabletop setups (`test*.world`), perform object recognition, then read in respective pick list (`pick_list_*.yaml`). Next construct the messages that would comprise a valid `PickPlace` request output them to `.yaml` format.


![demo-2](https://user-images.githubusercontent.com/20687560/28748286-9f65680e-7468-11e7-83dc-f1a32380b89c.png)

Spend some time at the end to discuss your code, what techniques you used, what worked and why, where the implementation might fail and how you might improve it if you were going to pursue this project further.  



