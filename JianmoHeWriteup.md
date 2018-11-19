**Vehicle Detection Project**

The goals / steps of this project are the following:

* Perform a Histogram of Oriented Gradients (HOG) feature extraction on a labeled training set of images and train a classifier Linear SVM classifier
* Optionally, you can also apply a color transform and append binned color features, as well as histograms of color, to your HOG feature vector. 
* Note: for those first two steps don't forget to normalize your features and randomize a selection for training and testing.
* Implement a sliding-window technique and use your trained classifier to search for vehicles in images.
* Run your pipeline on a video stream (start with the test_video.mp4 and later implement on full project_video.mp4) and create a heat map of recurring detections frame by frame to reject outliers and follow detected vehicles.
* Estimate a bounding box for vehicles detected.

[//]: # "Image References"
[image1]: ./examples/car_not_car.png
[image2]: ./examples/HOG_example.jpg
[image3]: ./examples/sliding_windows.jpg
[image4]: ./output_images/pipeline1.png
[image5]: ./examples/bboxes_and_heat.png
[image6]: ./examples/labels_map.png
[image7]: ./examples/output_bboxes.png
[image8]: ./output_images/heatmap.png


## [Rubric](https://review.udacity.com/#!/rubrics/513/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Vehicle-Detection/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Histogram of Oriented Gradients (HOG)

#### 1. Explain how (and identify where in your code) you extracted HOG features from the training images.

The code for this step is contained in second code cell of the IPython notebook (proj.ipynb).  

I started by reading in all the `vehicle` and `non-vehicle` images.  Here is an example of one of each of the `vehicle` and `non-vehicle` classes:

![alt text][image1]

I then explored different color spaces and different `skimage.hog()` parameters (`orientations`, `pixels_per_cell`, and `cells_per_block`).  I grabbed random images from each of the two classes and displayed them to get a feel for what the `skimage.hog()` output looks like.

I also tried using RGB and HSV. In previous projects, I used the the saturation channel of HSV to find features, but this did not work as well as a combination of all features of YCrCb in this project. In addition, I tested out many combinations of different HOG parameters. In training, `spatial_size = (8, 8)` and `hist_bins = 64` worked really well and had about 0.99 accuracy on the test set. However this did not work well in practice. I found that `spatial_size = (32, 32)` and `hist_bins = 16` is a bit worse in testing, but better in practice, most likely due to better generalization.

#### 2. Explain how you settled on your final choice of HOG parameters.

In addition to my explanation above, my process was mostly trial and error.

#### 3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).

I trained a linear SVM using the given vehicle and non-vehicle image database. After each image is loaded and labeled (1 for vehicle, 0 for non-vehicle), image features were extracted, including spacial features, histogram features, and HOG features. These features are stacked into one feature vector per image. Next, the feature vector database is shuffled and split using train_test_split: 80% used to train, 20% used to test. Then the data is normallized using StandardScaler. Finally the model is fit. This implementation can be seen in the code cell 13 of proj.ipynb.

### Sliding Window Search

#### 1. Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?

I used a hog sub-sampling sliding window search, which is benefitial because it calculates the hog features once and sub-samlples the dataset when searching for smaller and smaller features. I used a scale of 2 because this still keeps the visual information while making the algorithm run fast. Lines 35-67 in code cell 14 implemented the sliding window search.

![alt text][image3]

#### 2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?

Ultimately, I searched on one scale using YCrCb 3-channel HOG features plus spatially binned color and histograms of color in the feature vector, which provided a working result. Using HOG sub-sampling and a higher scaling factor really improved speed, as there is less processing per image. Here is an example image:

![alt text][image4]

---

### Video Implementation

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)

The result of processing the video is in the github repo, the file name is called project_video_cars.mp4.


#### 2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

I recorded the positions of positive detections in each frame of the video.  From the positive detections I created a heatmap and then thresholded that map to identify vehicle positions.  I then used `scipy.ndimage.measurements.label()` to identify individual blobs in the heatmap. I then assumed each blob corresponded to a vehicle.  I constructed bounding boxes to cover the area of each blob detected.  I implement a pixel-hit threshold of 2. While I am not including more filters in my submission (due to the amount of time I have available to work on the project), I did play around with some filters and have ideas for what I would do given more time. For example, I can keep track of heat map labels from frame to frame and filter out transient detections, filtering out false positives. In addition, I can increase the window overlap of my sliding search window and raise the threshold to generate more vehicle detections where there are actual vehicles and reduce false positives.

Here's an example result showing the heatmap:

![alt text][image8]

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

There were some problems that I faced with this project. But with doing it over and over again, I solved them all. At first it's the vehicle detection. My model worked well on the testing set but not very good on new images. But from the behavioral cloning project, I know that was overfitting. Next, I only recognized part of the car. The bounding boxes were drawn part of the car. I tuned the parameters and it seems better. In addition, as can be seen in my final video, the white car seems to get lost, and I think this might have to do with it's angle. It is recognized again, but I'd like to do more digging into why it's lost for a bit.

My pipeline is not perfect for sure. I could see there're defects which I could see in the video. I will play around with the parameters to see if it could be better. Also, it took a long time to get the result. Maybe it would be more efficient with simplifying the algorithm.

