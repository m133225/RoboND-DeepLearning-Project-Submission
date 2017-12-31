## Project: Follow Me

[image1]: ./images/fcn_arch.png
[image2]: ./images/graph.png

### Concepts
#### Convolutional Neural Network
Convolutional Neural Networks (CNNs) are often used in imaging-related applications since the convolution layers emulate the way visual cortex of humans processes vision - by extracting information from local regions. By stacking convolution layers, we attempt to extract higher and higher level of information from the given image - edges/curves, then shapes, etc.. Max pooling layers (and other variations) are also used to improve performance by avoiding overfitting. While its success in image classification is renowned in visual recognition challenges, the traditional CNN architecture lacks the ability of determining positions in objects in the images. In particular, the Fully-Connected Layers, usually placed at the end of the network, remove spatial information that is essential for locating objects.

#### Fully Convolutional Network
To preserve spatial information, as well as to keep the non-linearity of the network, we can replace the Fully-Connected Layers with 1x1 Convolution Layers. There are 2 additional advantages in doing this: one, we can feed images of any size into the resulting trained network, and two, we can easily increase and decrease the complexity of the most compact (usually) representation by varying the number of 1x1 filters.

To make use of the kept spatial information, we make use of bilinear upsampling to predict pixel probabilities. This requires less overall training, as compared to traditional transpose convolutions, since the bilinear upsampling is just linear interpolations of values.

Another powerful tweak we can make to the network is through the use of skip connections. In essence, this means combining the high-level and low-level information to make spatial predictions more robust and accurate. This is done in this project by concatenating a prior layer that has the same height and width, and then perform convolution on this resulting layer.

#### Batch normalization
To optimize network training, we can also make use of batch normalization, which is to normalize the input batch at each layer. This allows us to mitigate vanishing gradient to some extent, as well as provide some level of regularization.

#### Depthwise Separable Convolutions
Instead of the traditional convolution layers, we can instead make use of separable convolution layers. What the layer does is to perform convolution on each of the input channels, and then apply 1x1 convolutions to the result. The advantage is that it uses less parameters which enables fast computation, both in learning and evaluation.

### Hyperparameters
Aside from deciding the complexity of the network architecture, the other important decision required for this project is the choice of hyperparameters. While attempts were made to tune the hyperparameters based on its performance as well as the loss graph, they did not help to exceed the 0.40 final score benchmark. In particular, the general idea when tuning includes:
- When the validation loss fluctuates too much, learning rate is reduced and/or batch size is increased
- When the validation loss towards the end is still decreasing, the number of epochs is increased
- When the final validation loss is too big (than the usual ~0.05), or the score is too low, then # of encoder/decoder layers, # of separable convolution layers, and # of filters in each of the layers are tweaked

With about 50 different runs of training on the AWS instance, the best result obtainable for me was at 0.3904. In the end, collection of more `patrol_with_targ` type of images and some further tuning was needed to exceed the required 0.40 threshold. Towards the end, it became more of a bruteforce of hyperparameters since they seem to have similar performance, although the graphs they produce vary significantly.

### Results
#### Model
![FCN Architecture][image1]

This is the resulting model used for training. Each decoder block included 3 layers of separable convolution to sieve out finer details from prior layers. A 1x1 convolution layer with 96 filters was found to perform well in many different trials and was thus used.

#### Performance
![Training Loss Graph][image2]

After data collection and further tuning, a score of 0.4267 was achieved.
A total of 4892 training images were used. The final model was trained with a 0.001 learning rate, 24 epochs, batch size of 20, and steps of 250. In the decoder block, 3 separable convolution layers were used.

### Segmentation for other objects
If we were to blindly use the trained model to other objects (or even other targets...), of course, it will be unlikely to work. The most important change required is the data used for training. The model we have has been specifically trained to extract information required to identify people from the background, and the hero from the people - colour and shapes, or possibly even type of clothing, and extracting this same information for other objects may not be useful.

Even then, changing of data does not ensure that our model will work for other objects. Depending on the new environment and the type of object of interest, significant changes to the complexity of the model may also be needed to learn different abstractions for identification.