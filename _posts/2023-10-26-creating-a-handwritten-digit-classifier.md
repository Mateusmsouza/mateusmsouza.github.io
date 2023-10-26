---
layout: post
title: Creating a handwritten digit classifier
categories: [Machine Learning, Deep Learning]
---

As you start your journey through the extensive world of machine learning, perhaps you need a complete walk through a problem solved by neural network. A handwritten digit classifier could be considered as a great “hello world” in this case, which we can apply not a very hard neither too easy code to solve. 

## Dataset

For this code we will be using the MNIST dataset contains 70,000 images of handwritten digits ranging from 0 to 9.

The Keras API makes it easy to acquire this dataset.

```python
from tensorflow.keras.datasets import mnist

(train_images, train_labels), (test_images, test_labels) = mnist.load_data()

print(f"The type of the dataset is {type(train_images)}")
print(f"An example of a stored image and its class is: \n"
f"Label: {train_labels[0]} \nImage: {train_images[0]}")
```

The image may not look like an actual image when printed like this because it is stored as a tensor, which is a fundamental data type for storing text, images, and more.

We can visualize the image using matplotlib.

```python
import matplotlib.pyplot as plt
plt.imshow(train_images[0])
```

## Data preprocessing

During the preprocessing phase we want to manipulate the data so that we have better results in our experiments, such as increasing accuracy, decreasing time required for the network to converge.

In the following code, the images are reshaped so that the tensors can be inputted to the network, proceeding to normalize all pixels of the image that have a value between 0 and 255, and by dividing them to 255 we end up with all pixels being 0 or 1. Lastly, the labels are converted to one hot vector.

```python
import numpy as np
from tensorflow.keras.utils import to_categorical

print(f"The old shape is: {train_images.shape} {test_images.shape}")
train_images = np.reshape(train_images, (-1, 784))
test_images = np.reshape(test_images, (-1, 784))
print(f"The new shape is: {train_images.shape} {test_images.shape}")
train_images = train_images.astype('float32') / 255
test_images = test_images.astype('float32') / 255

train_labels = to_categorical(train_labels)
test_labels = to_categorical(test_labels)
```

## Creating the architecture of the neural network

The architecture of the really simple neural network is explicitly created below.

Note that the InputLayer has a shape of 784, exactly how we reshaped the data in the preprocessing step.

```python
from tensorflow.keras.models import Sequential
from tensorflow.keras.layers import InputLayer
from tensorflow.keras.layers import Dense

MLP = Sequential()
MLP.add(InputLayer(input_shape=(784, ))) # input layer
MLP.add(Dense(256, activation='relu')) # hidden layer 1
MLP.add(Dense(256, activation='relu')) # hidden layer 2
MLP.add(Dense(10, activation='softmax')) # output layer

# summary
MLP.summary()
```

## Training and testing

```python
MLP.fit(train_images, train_labels,
        epochs=20, batch_size=128)
```

```python
test_loss, test_acc = MLP.evaluate(test_images, test_labels,
                                   batch_size=128)
print(f"Test loss: {test_loss}")
print(f"Test accuracy: {(test_acc * 100):.2f}%")
```

With this code we should achieve training accuracy around 99% and 97% for testing.

### References

[https://towardsdatascience.com/creating-a-multilayer-perceptron-mlp-classifier-model-to-identify-handwritten-digits-9bac1b16fe10](https://towardsdatascience.com/creating-a-multilayer-perceptron-mlp-classifier-model-to-identify-handwritten-digits-9bac1b16fe10)

[https://www.deeplearningbook.com.br/funcao-de-ativacao/](https://www.deeplearningbook.com.br/funcao-de-ativacao/)

Citation: Deng, L., 2012. The mnist database of handwritten digit images for machine learning research. IEEE Signal Processing Magazine, 29(6), pp. 141–142.
Source: [http://yann.lecun.com/exdb/mnist/](http://yann.lecun.com/exdb/mnist/)
License: Yann LeCun (Courant Institute, NYU) and Corinna Cortes (Google Labs, New York) hold the copyright of the MNIST dataset which is available under the Creative Commons Attribution-ShareAlike 4.0 International License (CC BY-SA). You can learn more about different dataset license types here.