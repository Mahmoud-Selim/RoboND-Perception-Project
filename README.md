## Writeup / README
### Project: Perception Pick & Place
The goal of the project is to make a perception pipeline to locate objects and identify them in a cluttered environment.
---
[//]: # (Image References)

[image1]: ./images/voxel_grid.PNG
[image2]: ./images/pass_through.PNG
[image3]: ./images/extracted_inliers.PNG
[image4]: ./images/extracted_outliers.PNG
[image5]: ./images/accuracy.PNG
[image6]: ./images/world1.PNG
[image7]: ./images/world2.PNG
[image8]: ./images/world3.PNG


### RGB-Camera
RGB-Data camera is the sensor used to perform the perception pipeline. It's hybrid sensor which contains 2 sensors. A passive sensor and an active sensor. The passive sensor is a normal RGB camera from which we can read RGB images. the active sensor is a depth infrared sensor. The infrared (IR) sensor has 2 parts: a projector and a receiver. IR transmitter projects light out n a predefined pattern and calculating the depth by interpreting the deformation in that pattern caused by the surface of target objects. These patterns range from simple stripes to unique and convoluted speckle patterns. Depth is perceived by interpreting the deformation in the pattern caused by the surface of target objects. The advantage of using RGB-D cameras for 3D perception is that, unlike stereo cameras, they save a lot of computational resources by providing per-pixel depth values directly instead of inferring the depth information from raw image frames. In addition, these sensors are inexpensive and have a simple USB plug and play interface. RGB-D cameras can be used for various applications ranging from mapping to complex object recognition.

### Point Cloud
In the previous sections we've explored various techniques used by 3D sensors to infer depth information from a scene. Here, we will look at how we can represent that information as a point cloud, which is an abstract data type responsible for storing data from our RGB-D sensor. Nearly all 3D scanners or Lidars output data as high accuracy Point Clouds. Data from stereo cameras and RGB-D cameras can also be easily converted into Point Clouds. Point Clouds are used in numerous applications where 3D spatial information is a key component of the data. 

---

### Here i will discuss the steps performed in the pipeline 
#### Pipeline for filtering and RANSAC plane fitting implemented.
First, we get a raw RGB-D image from the sensor. the first thing to be done in a real world is to calibrate the sensor. Other than that, a noise removal should be done. After the noise removal, i did sample from the 3D point cloud to speed up the processing. I used a VoxelGrid downsampling filter to do that. It may turn out to affect the accuracy but for the purpose of the project it worked well. The VoxelGrid sample is shown below

![alt text][image1]

After the down sampling, a pass through filter was used to pass only the portion of interest. The pass through filter was used on the Z-axis and passed only the table with the objects above it. 

![alt text][image2]

After filtering, the table was removed from the cloud to extract the objetcs. That was done using the Random Sample Consensus or "RANSAC". RANSAC assumes point cloud data is comprised of inliers and outliers. Depending on the model defined, inliers are the points in the point cloud which fit the model and thereby outliers can be defined as the points which do not fit the model. The model we define can be in any form like plane, cylindrical or any common shape. In this case, we know the table is rectangular and so here we use PCL's RANSAC plane fitting. Since table here is the single most prominent plane in the scene after the pass through filter. So, A RANSAC was used and the table and the objects were sperated as shown below

![alt text][image3]

![alt text][image4]

#### 2. Pipeline including clustering for segmentation implemented.  

So far we have performed RANSAC plane fitting and seperated the table from the objects above it. So we have a PointCloud where only the objects of interest exist. Our 3D point cloud data has some other details like color, spatial information or any such rich features, which can be used for our segmentation task. Now, the seperation fo objects are required. In the unsupervised learning literature, this can easily be done. A clustering algorithm was used to seperate the objects into different groups. In this project, Density-Based Spatial Clustering of Applications with Noise was used. It is also called "DBSCAN Algorithm" or “Euclidean Clustering”. That's because it seperates the objects depending on the eculedian distance between them. There are 3 important parameters affecting overall performance: ClusterTolerance, which is the allowed spatial cluster tolerance as a measure in the L2 Euclidean space. After several trials, I decided to keep a value of 0.05 as cluster tolerance. Second and Third parameters are Min and Max cluster sizes, which are set based on the minimum and maximum size of target objects in the scene we are trying to segment. For this project, using minClusterSize = 50 and maxClusterSize = 5000. Using this configuration, all the objects were seperated successfully with each group has the XYZRGB PointCloud of each seperate object

#### 2. Features extracted and SVM trained.  Object recognition implemented.
To segment the specific object from clustered objects, 3D object recognition pipeline was used. Seperate features was used to train our algorithm. One such feature available in our point cloud data is color information RGB. To eleminate the sensitivity to lighting conditions, a different colour space was used. The image was transformed from RGB domain to the HSV one and used the appropriate channels. Color information is converted into normalized histogram form to accommodate varying image sizes for a target object. Another feature which can be used is the partial information of 3D shapes available in the object point cloud data. Object surface can be described by its distribution of surface normals and this in normalized form can be used a feature. Details for feature extraction in this project can be seen features.py

Until now, we have a ready to train features. The only thing remaining is the training itself. The algorithm used was the SMV algorithm. It was used with many kernels such as rbf, linear and sigmoid. the performance didn't imporve much in the rbf or the sigmoid case. So, a linear kernel was used for the training. Here is an image of the confusion matrix and the normalized confusion matrix below

![alt text][image5]

### Pick and Place Setup

#### 1. For all three tabletop setups (`test*.world`), perform object recognition, then read in respective pick list (`pick_list_*.yaml`). Next construct the messages that would comprise a valid `PickPlace` request output them to `.yaml` format.

Here, the 3 configuration of the worlds was used. A 100% was achived in both the first and the second one. However, one object was classified wrong in the third case. an imporvment that could be done is to lower the sampling rate and use more features for training. The images for the results are shown below

![alt text][image6]

![alt text][image7]

![alt text][image8]

