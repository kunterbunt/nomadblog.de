+++
draft = true
title = "Neural Networks Tutorial"
date = 2019-08-02T16:58:16+02:00
summary = ""
tags = ["tech", "machine-learning"]
showLogo = true
logo = "/imgs/logos/nerdy/neural_networks_tutorial2.png"
hasMath = true
+++

Artificial Neural Networks (ANNs) are *the* hype in Computer Science, and I have jumped on the bandwagon.
I will surely write about how I want to apply ANNs to the field of communication networks in some later post, but for now I want to present what I have gathered about these lovely little thingies, and show it in a way that would have been very useful to me when I first started looking into ANNs:

**I will showcase a brief, hands-on example of how ANNs work, and provide some TensorFlow / Keras code to back up my claims.**

The examples are based on [**Andrew Ng's course on Machine Learning on coursera.org**](https://www.coursera.org/learn/machine-learning/home/welcome) - I can certainly recommend this course. The Python code is written by me, and uses `tensorflor v1.14` and its built-in Keras library.

To keep things short, I won't introduce everything about ANNs; you can easily find info on this on thousands of other sites, online courses and books.
Instead, the practical example of *learning logic operators through ANNs* from the above-mentioned online course is given.

Learning the logical `AND`
===
You probably know the `AND` operator. Its truth table fully defines its function (output 1 only if both inputs are 1):

<table class="mdl-data-table mdl-js-data-table mdl-shadow--2dp">
  <thead>
    <tr>
      <th>x1</th>
      <th>x2</th>
      <th>x1 AND x2</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <td>0</td>
      <td>1</td>
      <td>0</td>
    </tr>
    <tr>
      <td>1</td>
      <td>0</td>
      <td>0</td>
    </tr>
	<tr>
      <td>1</td>
      <td>1</td>
      <td>1</td>
    </tr>
  </tbody>
</table>

We can approximate the `AND` function with a single-layer Neural Network. Learning the truth table above for a number of epochs, it would learn the following weights:
![Neural Network](/imgs/neural-network-tutorial/ann_AND.png)

and by setting the activation function as the *Sigmoid function*
$$ g(z) = \frac{1}{1 + e^{-\Theta^T z}} $$
the ANN is capable of learning the `AND` function, as its prediction becomes
$$ h_\theta(x) = g(-30 + 20x_1 + 20x_2) $$
which leads to 

<table class="mdl-data-table mdl-js-data-table mdl-shadow--2dp">
  <thead>
    <tr>
      <th>x1</th>
      <th>x2</th>
      <th>x1 AND x2</th>
	  <th>prediction</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>0</td>
      <td>0</td>
	  <td>g(-30) ≈ 0</td>
    </tr>
    <tr>
      <td>0</td>
      <td>1</td>
      <td>0</td>
	  <td>g(-10) ≈ 0</td>
    </tr>
    <tr>
      <td>1</td>
      <td>0</td>
      <td>0</td>
	  <td>g(-10) ≈ 0</td>
    </tr>
	<tr>
      <td>1</td>
      <td>1</td>
      <td>1</td>
	  <td>g(10) ≈ 1</td>
    </tr>
  </tbody>
</table>

And now in Python
---

```
import tensorflow.compat.v1 as tf
from tensorflow.keras import layers
import numpy as np

def construct_single_layer_ann(learning_rate, num_inputs, num_outputs):
	model = tf.keras.Sequential()	
	model.add(layers.Dense(units=num_outputs, activation='sigmoid', input_shape=(num_inputs,)))
	# Gradient Descent + Mean Squared Error loss function.
	model.compile(optimizer=tf.train.GradientDescentOptimizer(learning_rate), loss='mse', metrics=['accuracy'])
	return model

def get_training_data_AND():
	data = [[(0,0), (0,1), (1,0), (1,1)]]
	labels = [0, 0, 0, 1]
	return data, labels

def print_layer_weights(model, num_layer):
	layer_weights = model.layers[num_layer].get_weights()[0]
	bias_weights = model.layers[num_layer].get_weights()[1]	
	print("Learned weights are", end=' ')	
	for i in range(len(layer_weights)):
		for j in range(len(layer_weights[i])):
			print("n_" + str(num_layer+1) + "," + str(i+1) + "_" + str(j) + "=" + str(layer_weights[i][0]), end=' ')
	for i in range(len(bias_weights)):
		print("b_" + str(num_layer+1) + "," + str(i) + "=" + str(bias_weights[i]), end=' ')
	print()

if __name__ == "__main__":
	learning_rate = 10.0
	num_epochs = 50
	verbose = False

	# Logical AND model.	
	model_AND = construct_single_layer_ann(learning_rate, 2, 1)
	# Provide perfect training data.
	data_AND, labels_AND = get_training_data_AND()
	# Fit the model to the data.
	history = LossHistory()
	model_AND.fit(data_AND, labels_AND, epochs=num_epochs, verbose=verbose, callbacks=[history])
	# Print weights and loss.
	print("Logical AND ANN:\n================")
	print_layer_weights(model_AND, 0)
	print("Final loss is " + str(history.losses[-1]))	
	predictions = model_AND.predict(data_AND)
	for i in range(predictions.size):		
		print(str(data_AND[0][i]) + " -> " + str(predictions[i][0]) + " ≈ " + str(round(predictions[i][0])))	
	print()
```

which outputs

```
Logical AND ANN:
================
Learned weights are n_1,1_0=3.8320987 n_1,2_0=3.8320732 b_1,0=-5.8527803 
Final loss is 0.012041996
(0, 0) -> 0.002863679 ≈ 0.0
(0, 1) -> 0.117045894 ≈ 0.0
(1, 0) -> 0.11704853 ≈ 0.0
(1, 1) -> 0.85953003 ≈ 1.0
```

We can see that instead of learning the weights -30, 20, 20 it learns -5.8, 3.8, 3.8. The *relations* are the same though
$$\frac{-30}{20} = -1.5 \approx{} \frac{-5.8}{3.8} = -1.53$$

This concludes this very brief introduction. The practical example of the `AND` function seemed intuitive to me. All an ANN does is adapt its weights until the loss (prediction - target) is adequately small. Since the Sigmoid function is non-linear, the ANN can approximate non-linear functions. And Keras gives us a way of implementing this in very few lines of code. :)