## Project: Follow Me
### Writeup Template: You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

---

[//]: # (Image References)

[image1]: ./imgs/auto_mode_run.PNG
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
##### Encoder layers
The encoder part is composed of 3 encoder layers. 
The first layer has a filter size of 32, and stride of 2. The output shape of this layer is 64x64x32 (128/stride)
The second layer has a filter size of 64, and stride of 2. The output shape of this layer is 32x32x64 
The third layer has a filter size of 128, and stride of 2. The output shape of this layer is 16x16x128

The encoder layers work to abstract features from the images. With each layer, features from colors and shapes to complex colors and shapes will be learned by the network.
##### 1 x 1 convolution layer
The convolution layer takes the encoder output and perform 1x1 convolution on it. The purpose of this layer is to preserve the spatial information of the encoder layer output and generate categorical probability on each pixel.

Here we use a 1x1 conv layer with filter_depth=3, and strides=1, so that we can preserve the shape of the input layer.

The output shape is 16x16x3

##### Decoder layers
The decoder part consists of 3 decoder layers.

Each layer has an upsample layer, a concatenate layer, and a convolution with batchnorm layer.

The purpose of the upsample layer is to upsample the probabilities from the 1x1 convolution layer to a larger output, eventually, the original image size, so that we can get the categorical probabilities of each pixel in the original image. Here, each decoder layer output is upscaled by 2. 

Therefore, the output shape of first decoder layer is 32x32xh, the second decoder layer output is 64x64xh, the third 128x128xh

The purpose of the concatenate layer is to perform the 'skip' connections so that we add in some encoder layer output to the decoder layer, which helps with precision boundaries in detecting categories.

Finally, the number of decoder layers match that of encoder layers, and we concatenate the layers which have the same dimensions. 


#### 3. Network parameters
##### Network Depth

##### Epoch
Initially, the epoch number is set to 20. However, error plot of training and validation data quickly shows that overfitting happens around epoch 7-9. Thus, after a few trial-and-error, I have set the epoch to be 8. 

##### Batch Size
The larger the batch size, the more accuracy we gain with each iteration. However, there's a machine limit that we can not set the batch size to be too large. Also, with a large batch size, it means we will overfit earlier, since within each iteration, more data are seen by the network.

##### Learning Rate
After trial-and-error, I've set the learning rate to be 0.01. Going larger in learning rate makes the model unstable and sometimes not converge. Going smaller makes the model slow to train and may settle in a local optimum. 


#### 4. 1x1 conv layer vs. fullly connected layer
The difference between a fully connected layer and a 1x1 conv layer is that: with fully connnected layer, we aim to come to a conclusion of what this image (or other forms of input) is about, aka, generate a fixed number of categorical labels (in a classification problem); with 1x1 conv layer, we aim to tell what each part of the image is about, aka, the categorical probabilities of each section of an image. With fully connected layer, we collapse the input dimensions, multiply, activate and add to calculate the categorical probabilites. With 1x1 conv layer, we still do the multiply and activate, but do not 'add' or collapse all dimensions into 1, aka, preserve the spatial information about the input so that we can tell where in the image it says about what.

#### 5. Efforts & Results
For better training, I have collected additional image, especially for hero to walk in a large crowd. 
This improves the initial IOU score, as I collect more data, it does not particularly improve the test data score.
The model is able to reach an IOU score of 
If the model will be redeployed to follow a cat, or dog, instead of the hero, it will need to be retrained. However, the network may be easier to train since a cat or dog image is drastically different from other people, or the background. However, if we have other cats and dogs in the scene, it may be just as hard to train.

#### 6. Improvements & Future work
This project takes a lot of time to train. As I understand on a high level how each layer works, and how they should be used, sometimes it's hard to interprete the results of improvement efforts. For example, in the lecture it suggests not use skip connections in all the layers, but once I take out the any skip connection, the model test score falls by a large percent. Also, I would like to read more on how to fine-tune filter size in each layer, how the choice of strides affect the results, and other techniques to fine-tune the network to reach optimal performance.
