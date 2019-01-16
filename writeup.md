## Project: Follow Me
### Writeup Template: You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

---

[//]: # (Image References)

[image1]: ./imgs/FollowMe-NeuralNetworkStructure.png
[image2]: ./calibration_images/example_grid1.jpg
[image3]: ./calibration_images/example_rock1.jpg 
[image4]: ./calibration_images/example_edge.jpg 
[image5]: ./calibration_images/example_obs1.jpg 


## [Rubric](https://review.udacity.com/#!/rubrics/916/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  

You're reading it!

#### 2. Network analysis
The network consists of mainly 3 parts: encoder, 1x1 convolution, and decoder.
The neural network structure is shown below:

![alt text][image1]
##### Encoder layers
The encoder part is composed of 4 encoder layers. 

The first layer has a filter size of 16, and stride of 2. The output shape of this layer is 64x64x16 (128/stride).

The second layer has a filter size of 32, and stride of 2. The output shape of this layer is 32x32x32.

The third layer has a filter size of 64, and stride of 2. The output shape of this layer is 16x16x64.

The fourth layer has a filter size of 128, and stride of 2. The output shape of this layer is 8x8x128.

The encoder layer is used to extract information from the image. We use various filters to abstract different features of the image, like simple lines, shapes, more complicated shapes, etc, with the deepening of layers. This extraction will cause information loss from the original image as the filter scans through the image and performs convolution(multiply and add) over neighboring pixels.

The encoder layers work to abstract features from the images. With each layer, features from colors and shapes to complex colors and shapes will be learned by the network.
##### 1 x 1 convolution layer
The convolution layer takes the encoder output and perform 1x1 convolution on it. The purpose of this layer is to preserve the spatial information of the encoder layer output and generate categorical probability on each pixel.

Here we use a 1x1 conv layer with filter_depth=3, and strides=1, so that we can preserve the shape of the input layer.

The output shape is 8x8x3

##### Decoder layers
The decoder part consists of 4 decoder layers.

Each layer has an upsample layer, a concatenate layer, and a convolution with batchnorm layer.

The purpose of the upsample layer is to upsample the probabilities from the 1x1 convolution layer to a larger output, eventually, the original image size, so that we can get the categorical probabilities of each pixel in the original image. Here, each decoder layer output is upscaled by 2. 

Therefore, the output shapes of decoder layers are 16x16x64, 32x32x32, 64x64x16, and 128x128x3

The purpose of the concatenate layer is to perform the 'skip' connections so that we add in some encoder layer output to the decoder layer, which helps with precision boundaries in detecting categories.

Finally, the number of decoder layers match that of encoder layers, and we concatenate the layers which have the same dimensions. 

The purpose of decoder layer is to translate the probability of the 'categorized' smaller image back to the original image, by interpolating the probability back into original pixel space. In this case, it is done through upscaling. This is not accurate, as we have previously lost information during encoding, therefore, we use some sort of bypass layers to bypass the less encoded layers to the decode layers to compensate for the information loss.

#### 3. Network parameters
##### Network Depth
Initially, the network depth for encoder and decoder is each set to 3. 
After experimenting with depth of 4, I observed a significant drop in error rate from ~0.8 to ~0.04.
Apparently, the deeper network is able to abstract feautures a little bit better.
However, this does not improve the IOU, I think this is due to overfitting happening earlier for deeper network. Thus, I reduced the number of epoch from 8 to 6.

##### Epoch
Initially, the epoch number is set to 20. However, error plot of training and validation data quickly shows that overfitting happens around epoch 7-9. Thus, after a few trial-and-error, I applied early stopping at epoch 8 for 3 layers, and 6 for 4 layers.

##### Batch Size
The larger the batch size, the more accuracy we gain with each iteration. However, there's a machine limit that we can not set the batch size to be too large. Also, with a large batch size, it means we will overfit earlier, since within each iteration, more data are seen by the network.

##### Steps per epoch
Keeping this at 200 will have a better performance than 100 given the other set parameters. Our data size is in the ~5000 range.

##### Learning Rate
After trial-and-error, I've set the learning rate to be 0.01. Going larger in learning rate makes the model unstable and sometimes not converge. Going smaller makes the model slow to train and may settle in a local optimum. 


#### 4. 1x1 conv layer vs. fullly connected layer
The difference between a fully connected layer and a 1x1 conv layer is that: with fully connnected layer, we aim to come to a conclusion of what this image (or other forms of input) is about, aka, generate a fixed number of categorical labels (in a classification problem); with 1x1 conv layer, we aim to tell what each part of the image is about, aka, the categorical probabilities of each section of an image. With fully connected layer, we collapse the input dimensions, multiply, activate and add to calculate the categorical probabilites. With 1x1 conv layer, we still do the multiply and activate, but do not 'add' or collapse all dimensions into 1, aka, preserve the spatial information about the input so that we can tell where in the image it says about what.

#### 5. Efforts, Results * Observations
##### Efforts
For better training, I have collected additional image, especially for hero to walk in a large crowd, as well as following people that look like the hero (same color clothing, for example)
However, the following other people data collection has caused the trained model to have a lower performance. The false positives of recognizing the hero increases tremendouly. I think this is due to how test set are composed of and a relatively small data set. Since one part of the test set is about recognizing the hero when patrolling over, with a small data set, the network could be learning that the person pattern that looks like a rover is patrolling and following is the hero. Therefore, if the dataset doesn't include following other people images, and the test set doesn't test following other people, the score is pretty high. But this doesn't mean the network is robust in telling the difference between following the hero and following other people. 

##### Results

The model is able to reach an IOU score of 0.419

##### Observations
More data improves the initial IOU score, however, as I collect more data, it does not necessarily improve the test data score, due to the reason explained above regarding test set and dataset similarity.
Collecting individual other people data will help with the classification of both the hero, and other people, since the machine needs to tell the hero from other people, thus both data is needed in large quantities.

If the model will be redeployed to follow a cat, or dog, instead of the hero, it will need to be retrained. However, the network may be easier to train since a cat or dog image is drastically different from other people, or the background. However, if we have other cats and dogs in the scene, it may be just as hard to train.

#### 6. Improvements & Future work
##### Training Network
This project takes a lot of time to train, and figure out what data to collect. As I understand on a high level how each layer works, and how they should be used, sometimes it's hard to interprete the results of improvement efforts. For example, in the lecture it suggests not use skip connections in all the layers, but once I take out one of the skip connections, the model test score falls by a large percent. Also, I would like to read more on how to fine-tune filter size in each layer, how the choice of strides affect the results, and other techniques to fine-tune the network to reach optimal performance.
Another point worth mentioning is the randomization of data set, so that each batch has a good representation of all three scenarios evenly distributed.

##### Data collection
To make the score higher, I noticed that we need to collect more information regarding which IOU score are low in results, and collect the data correspondingly. This has helped improved the score without having to change the network, and watch out for the effect of small data and fixed test set problem.

case#1 for following behind the target, the network is doing pretty good. This is due to the fact that I've collected a good amount of data following target from directly above and at different angles.

case#2 for patrol without target, the network has 69 false positives. I have collected a set of data patrolling the crowd, but more often at some distance but not too close. The false positives is likely due to the fact that in the training data we're missing (or in low quantity) the patrol near other people case, which is true since I took out those data where I collected directly above other people, but not as in test case where it's really close to other people but not directly above.

case#3 for target from far away, number true positives: 135, number false positives: 3, number false negatives: 166. I have collected a good amount of data of hero from far away at the crosswalk and with the grass background. The fact that we have many false negatives may due to the fact that some of the backgrounds in the test are not present in the training data set. For example, near the street, hero with one leg showing in the image, etc.

To further improve the test score, I need to study the test set and the angles and background present in the test set if I only have time to collect small data size. To further improve the model and its robustness, a huge train data is needed with all kinds of images and possibilities, such as images patrolling over other people, walking in different backgrounds, partial and full presence of hero/others in image, etc. Another trick to apply is to utilize the fact that the hero is almost all 'red' in color, thus we can pick out the red channel to build a small network on its own and ensemble that with the RGB network.
