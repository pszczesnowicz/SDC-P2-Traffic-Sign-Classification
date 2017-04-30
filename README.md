This is my submission for the Udacity Self-Driving Car Nanodegree Traffic Sign Classification Project. You can find my code in this [Jupyter Notebook](https://github.com/pszczesnowicz/SDCND-P2-TrafficSignClassification/blob/master/TrafficSignClassification.ipynb). The goal of this project was to apply my freshly learned deep neural network and convolutional neural network knowledge to classify images belonging to the [German Traffic Sign Dataset](http://benchmark.ini.rub.de/?section=gtsrb&subsection=dataset). The following write-up recounts the adventure that was building my very first convolutional neural network.

## Dataset Exploration

### Dataset Summary

After loading the files, I got my first glimpse of the datasets.

* Number of training examples = 34799

* Number of validation examples = 4410

* Number of testing examples = 12630

* Image data shape = 32x32x3

* Number of classes = 43

### Exploratory Visualization

To get an idea of what the training dataset examples look like I printed out a random example for each class. This helped me identify the classes for which fake data could be generated by simply flipping the images.

<img src="https://raw.githubusercontent.com/pszczesnowicz/SDCND-P2-TrafficSignClassification/master/readme_graphics/training_dataset_examples.jpg" width="800">

The bar chart below shows the uneven distribution of examples per class in the training dataset. The 50km/h speed limit class has the greatest number of examples (2010), while the 20km/h speed limit, dangerous curve to the left, and go straight or left classes had the least (180). This helped me identify the classes for which I should generate fake data.

<img src="https://raw.githubusercontent.com/pszczesnowicz/SDCND-P2-TrafficSignClassification/master/readme_graphics/training_dataset_distribution.jpg" width="800">

## Design & Test Model Architecture

### Pre-processing & Generating Data

After examining the training dataset examples I learned that color was not a distinct feature of the individual classes, therefore I decided to eliminate a feature that would not help identify a sign's class by converting the images to grayscale. To improve the optimizer efficiency I normalized the images by subtracting 128 from the pixel values and then dividing the result by 128. By doing so the pixel values would be in the range -1 to 1 with a mean of 0. The stop sign below is an example of what an image looks like before and after pre-processing.

<img src="https://raw.githubusercontent.com/pszczesnowicz/SDCND-P2-TrafficSignClassification/master/readme_graphics/stop_original.jpg" width="200"><img src="https://raw.githubusercontent.com/pszczesnowicz/SDCND-P2-TrafficSignClassification/master/readme_graphics/stop_pp.jpg" width="200">

I attempted to even out the distribution of examples per class in the training dataset by generating fake data for classes that met the following criteria:

* Can be flipped horizontally
* Can be flipped vertically
* Can be rotated 180 degrees
* Can be flipped horizontally then labeled as its symmetric counterpart (eg. turn left, turn right)
* Have less than 2000 examples

My goal was to train the model on the same number of examples per class. Unfortunately, as can be seen below, the distribution of examples per class was still uneven after generating fake data.

<img src="https://raw.githubusercontent.com/pszczesnowicz/SDCND-P2-TrafficSignClassification/master/readme_graphics/training_dataset_distribution_gen.jpg" width="800">

I then saved the pre-processed and generated datasets so that this time consuming operation would not have to be run again.

### Model Architecture

The first two layers of my model consist of a 5x5 convolution, a max pooling operation, and a ReLU activation.

For the next layer I implemented an Inception Module with a 1x1, 3x3, and 5x5 convolution as well as a max pooling operation followed by a 1x1 convolution. The 3x3 and 5x5 convolutions are preceded by a 1x1 dimensionality reduction that decreases their input depth and a ReLU for an additional activation. The outputs are then concatenated and activated with a ReLU. Because each convolution in the Inception Module has its own weight and bias, the optimizer can increase or decrease the effect each has on the model error.

The inception module is followed by a dropout operation, a single fully connected layer (instead of two as in the LeNet lab), another dropout operation, and finally, the output layer.

My model consisted of the following layers:

| Layer           | Description                                             |
|:----------------|:--------------------------------------------------------|
| Input           | 32x32x1 grayscale image                                 |
| Convolution     | 5x5 filter, 1x1 stride, valid padding, output = 28x28x16|
| ReLU            | Rectified linear unit activation                        |
| Max pooling     | 2x2 filter, 2x2 stride, output = 14x14x16               |
| Convolution     | 5x5 filter, 1x1 stride, valid padding, output = 10x10x32|
| ReLU            | Rectified linear unit activation                        |
| Max pooling     | 2x2 filter, 2x2 stride, output = 5x5x32                 |
| Inception Module| Output = 5x5x128                                        |
| ReLU            | Rectified linear unit activation                        |
| Dropout         | Training keep probability = 40%                         |
| Flattening      | Output = 3200                                           |
| Fully connected | Output = 1600                                           |
| ReLU            | Rectified linear unit activation                        |
| Dropout         | Training keep probability = 40%                         |
| Output          | 43 logits                                               |

### Model Training

I left the LeNet lab training pipeline unchanged because it yielded good results. The training pipeline uses an Adaptive Moment Estimation (Adam) optimizer which is a type of gradient descent optimization algorithm.

My model's hyperparameters are:

* Number of epochs = 20
* Batch size = 128
* Learning rate = 0.0002
* Dropout keep probability = 40%

### Solution Design

I chose to base my model on the LeNet lab architecture because it was designed to classify handwritten digits which is a good starting point for a traffic sign classifier. I left the first two layers unchanged except for increasing their depths to capture more features and added an Inception Module to further increase the depth of my model. I removed one of the fully connected layers because it was not improving performance. At this point I was overfitting to the training dataset, so I added a dropout operation following the Inception Module and fully connected layer.

I kept the batch size the same as in the LeNet lab, but changed the number of epochs and iterated the learning rate and dropout keep probability before arriving at the final values which yielded my model's highest accuracies.

| Dataset   | Highest Accuracy Achieved|
|:----------|:-------------------------|
| Training  | 99.9%                    |
| Validation| 96.5%                    | 
| Testing   | 95.5%                    |

Had I not been training on a MacBook I probably could have tried more hyperparameter combinations and achieved higher accuracies.

## Test Model on New Images

### Predict Classes

These are the signs and my reasons for choosing them:
* 50km/h and 60km/h speed limit signs: Similarity - A 5 closely resembles a 6
* Right-of-way at next intersection and pedestrian signs: Similarity - If you squint, the intersection and pedestrian symbols look alike
* Turn left ahead and roundabout mandatory: Similarity - Both have a curved arrow pointing left (far stretch, I know)
* Priority road and keep right: Both have something in the background

I wanted to see whether the sign pairs that I chose based on similarity would show up in each others top five softmax probabilities.

As can be seen below, my model correctly predicted the class for each sign in the new image dataset (Success!).

<img src="https://raw.githubusercontent.com/pszczesnowicz/SDCND-P2-TrafficSignClassification/master/readme_graphics/predictions.jpg" width="800">

### Analyze Performance

My model correctly predicted 100% of the signs in the new image dataset, therefore the precision and recall scores were also 100% for each sign. Admittedly, the new images that I chose were not as bad as some in the training dataset. Had the model misclassified an image, the confusion matrix below would look more interesting.

A not very confusing, confusion matrix:

<img src="https://raw.githubusercontent.com/pszczesnowicz/SDCND-P2-TrafficSignClassification/master/readme_graphics/confusion_matrix.jpg" width="400">

The 100% accuracy on the new image dataset is comparable to the 95.5% accuracy on the testing dataset because the new image dataset contains fewer examples.

### Softmax Probabilities

My hypothesis that similar signs would show up in each others top five softmax probabilities was correct for the 50km/h and 60km/h speed limit signs and right-of-way at next intersection and pedestrian signs.

<img src="https://raw.githubusercontent.com/pszczesnowicz/SDCND-P2-TrafficSignClassification/master/readme_graphics/probabilities.jpg" width="800">

## Conclusion

This being my very first convolutional neural network I am quite satisfied with a testing accuracy of 95.5%. But I definitely would like to revisit this challenge after I have gained more knowledge and achieve a perfect accuracy.

## References

[Udacity Self-Driving Car ND](http://www.udacity.com/drive)

[Udacity Self-Driving Car ND - Term 1 Starter Kit](https://github.com/udacity/CarND-Term1-Starter-Kit)

[Udacity Self-Driving Car ND - Traffic Sign Classifier Project](https://github.com/udacity/CarND-Traffic-Sign-Classifier-Project)

[Udacity Self-Driving Car ND - LeNet Lab](https://github.com/udacity/CarND-LeNet-Lab)

[Traffic Sign Recognition with Multi-Scale Convolutional Networks](http://yann.lecun.com/exdb/publis/pdf/sermanet-ijcnn-11.pdf)

[An overview of gradient descent optimization algorithms](https://arxiv.org/pdf/1609.04747.pdf)

[Going Deeper with Convolutions](https://www.cs.unc.edu/~wliu/papers/GoogLeNet.pdf)

[Inception modules: explained and implemented](https://hacktilldawn.com/2016/09/25/inception-modules-explained-and-implemented/)
