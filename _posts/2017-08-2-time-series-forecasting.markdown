---
layout: posts
title:  "Deep Learning Recurrent Neural Net for Time Series Forecasting"
date:   2017-08-2 20:46:34 +0100
categories: [Kaggle]
tags: [data science,notebook]
share: true
---

The Notebook described here supersedes my basic Recurrent Neural Net (RNN) for predicting multiple time series for the Wikipedia page forecasting competition. Instead, this RNN is constructed using the GRUCell rather than the BasicRNNCell which provides the model with persistence in the patterns observed over the series. 

The introductory paper by Junyoung Chung, Caglar Gulcehre, KyungHyun Cho, Yoshua Bengio proposing a GRU architecture can be found here: <a href="https://arxiv.org/abs/1412.3555">https://arxiv.org/abs/1412.3555</a>.

For a full description of the competition, timescales, milestones, evaluation go to <a href="https://www.kaggle.com/c/web-traffic-time-series-forecasting">https://www.kaggle.com/c/web-traffic-time-series-forecasting</a>.

Time series prediction problems are a difficult type of predictive modelling problems. Unlike regression predictive modelling, time series adds the complexity of a sequence dependence among the input variables. This interdependence can be modelled using a recurrent neural network.

The model presented here is an RNN that takes a sequence of iputs and productes a sequence of outputs. The network is fed values over the last N time steps and is trained to predict values shifted by one, N+1, time steps into the future. 

<b>Findings</b>. The model is then cycled forwards by 90 time steps to make predictions. However, the model shown here fails to converge on an acceptably low Mean Square Error and therefore we must be sceptical about the results. Furthermore, whilst the model runs in polynomial time, the large number of rows means that this deep learning approach would be best suited to a distributed architecture.

<b>Anyway, let's dive into the code</b>. First, we'll need to import tensorflow and numpy.

{% highlight python %}
import tensorflow as tf
import numpy.random as rnd
import numpy as np
{% endhighlight %}

Now, onto the construction phase. We'll construct the RNN with 10 inputs (one for each Wiki page), 6 layers, 50 neurons per layer, and 39 time steps. The input X will have shape [-1, 39, 10]. The output y will have the same shape, and we'll make use of tf.contrib.rnn.OutputProjectionWrapper to reduce the 39 * 10 outputs at each layer and the final layer to the 10 outputs we need. Using the OutputProjectionWrapper is computationally expensive and this will be something to come back to and improve. We'll also make use of the Mean Square Error as the loss function.

{% highlight python %}
n_steps = 39
n_inputs = 10 #same as rows
n_neurons = 50
n_outputs = 10 #same as rows
n_layers = 6

learning_rate = 0.001

X = tf.placeholder(tf.float32, [None, n_steps, n_inputs])

#y has the same shape
y = tf.placeholder(tf.float32, [None, n_steps, n_outputs])

#create the cells and the layers of the network
layers = [tf.contrib.rnn.OutputProjectionWrapper(tf.contrib.rnn.GRUCell(num_units=n_neurons, activation=tf.nn.relu),output_size=n_outputs) for layer in range(n_layers)]
multi_layer_cell = tf.contrib.rnn.MultiRNNCell(layers, state_is_tuple=True)
outputs, states = tf.nn.dynamic_rnn(multi_layer_cell,X,dtype=tf.float32)

print(outputs.shape)
print(y.shape)

#define the cost function and optimization
loss = tf.reduce_mean(tf.square(outputs - y))
optimizer = tf.train.AdamOptimizer(learning_rate=learning_rate)
training_op = optimizer.minimize(loss)

#Adds an op to initialize all variables in the model
init = tf.global_variables_initializer()
{% endhighlight %}

Next, we'll load in the data for the first 10 Wikipedia pages. We'll also need to strip off some rows from the time series data to reshape the data to the desired dimensions.

{% highlight python %}
#Import training data
import csv
from numpy import genfromtxt
import numpy as np
import tensorflow as tf
import pandas as pd
import random
from sklearn.preprocessing import Imputer

#read the csv into a dataframe
df=pd.read_csv('../input/train_1.csv', sep=',', error_bad_lines=False, warn_bad_lines=True)

#clean the data
df.drop(['Page'], axis=1, inplace = True)
df = df.replace(np.nan, 0, regex=True)

#grab the relevant rows
X_test = df.values.astype(int)
rows = print(len(X_test))

#number of rows to read 
rows = 10

#fetch the rows
X_test = df.values.astype(int)
X_test = X_test[:rows,0:]

#set the labelled data as the time step + 1
Y_test = X_test[:,1:]

#strip n numbers off the rows for reshape
X_test = X_test[:,:-4]
Y_test = Y_test[:,:-3]

for iteration in range(len(X_test)):
    plt.plot(X_test[iteration,:], label="page"+str(iteration+1))

plt.xlabel("Time")
plt.ylabel("Value")
plt.legend()
plt.show()    
    
Y_test = Y_test.reshape((-1, n_steps, n_inputs))
X_test = X_test.reshape((-1, n_steps, n_inputs))
{% endhighlight %}

Now onto the training phase. We'll train the model using batch gradient descent. We'll also permute the batches (not the time instances) to avoid overfitting the model.

{% highlight python %}
import numpy
n_epochs = 5000
batch_size = 10
instances = X_test.shape[0]
saver = tf.train.Saver()

print("RNN construction", "n_layers", n_layers, "n_neurons", n_neurons, "n_inputs", n_inputs, "n_outputs", n_outputs)
print("training dataset", "shape", X_test.shape, "instances", instances, "batch_size", batch_size)
print("training with", "n_epochs", n_epochs, "iterations (instances // batch_size)", (instances//batch_size))

#open a TensorFlow Session
with tf.Session() as sess:
    init.run()
    
    #Epoch is a single pass through the entire training set, followed by testing of the verification set.
    for epoch in range(n_epochs):
    
        #print("X_test",X_test)
        idxs = numpy.random.permutation(instances) #shuffled ordering
        #print(idxs)
        #print(idxs.shape)
        X_random = X_test[idxs]
        #print('X_random', X_random)
        Y_random = Y_test[idxs]
        #print('Y_random', Y_random)
    
        #Number of batches, here we exhaust the training set
        for iteration in range(instances // batch_size):   

            #get the next batch - we permute the examples in each batch
            #X_batch, y_batch = next_batch(batch_size, n_steps)
            X_batch = X_random[iteration * batch_size:(iteration+1) * batch_size]
            y_batch = Y_random[(iteration * batch_size):((iteration+1) * batch_size)]
            
            #print("iteration", iteration, "X_batch", X_batch)
            #print("iteration", iteration, "y_batch", y_batch)
            
            X_batch = X_batch.reshape((-1, n_steps, rows))
            y_batch = y_batch.reshape((-1, n_steps, rows))
        
            #feed in the batch
            sess.run(training_op, feed_dict={X: X_batch, y: y_batch})
        
            if (epoch % 100 == 0 and iteration == 0):
                #check contents of first example in each batch
                #print("iteration", iteration, "X_batch", X_batch[0])
                #print("iteration", iteration, "y_batch", y_batch[0])
                mse = loss.eval(feed_dict={X:X_batch, y:y_batch})
                print("epoch", epoch, "iteration", iteration, "\tMSE:", mse)
            
            #print(epoch)
    
    saver.save(sess, "./my_model") 
{% endhighlight %}

Note the Mean Square Error is not converging towards zero. The model is failing to make accurate predictions.

I've shown below how you might go on to use the model to make predictions, looking 90 time steps forwards and appending to a array as we go, however we should be very sceptical about these results given in the high MSE.

{% highlight python %}
#Sequence generation using the model

max_iterations = 90 #number of time steps to look forward

with tf.Session() as sess:                        
    saver.restore(sess, "./my_model") 

    sequence = [[0] * n_steps] * rows
    for iteration in range(max_iterations):
        #print("iteration", iteration)
        
        X_batch = np.array(sequence[-n_steps*rows:]).reshape(-1,n_steps,rows)
        #print(X_batch)
        
        y_pred = sess.run(outputs, feed_dict={X: X_batch})
        #print("Y", y_pred)
        #print("numbers to be added", np.array(y_pred[0,n_steps-1]))
            
        sequence = np.append(sequence, y_pred[0,n_steps-1])      
        #print("sequence so far...", np.array(sequence))
        
    #print("end sequence", np.array(sequence))
    sequence[numpy.where(sequence<0)] = 0
    #print(sequence)
    sequence = np.array(sequence).reshape(-1,rows)
    #print(sequence)
{% endhighlight %}

{% highlight python %}
plt.figure(figsize=(8,4))

plt.figure(1)
plt.grid(True)
plt.title("Full iteration of prediction")

for iteration in range(rows):
    plt.plot(sequence[n_steps:,iteration], label = "page"+str(iteration+1))

plt.xlabel("Time")
plt.ylabel("Value")
plt.legend()

plt.figure(2)
plt.grid(True)
plt.title("First 10 time steps in predicted sequence")

for iteration in range(rows):
    plt.plot(sequence[n_steps:n_steps+10,iteration])

plt.xlabel("Time")
plt.ylabel("Value")
plt.legend()
plt.show()
{% endhighlight %}

<img src="\assets\wiki\forecast_1.jpg">
<img src="\assets\wiki\forecast_2.jpg">

<b>Next steps</b>. Deep learning is not guarenteed to effectively model these time series. Identifying any cycles in these time series,  dependencies between them may help to determine hyperparameters that train a better fitting model and indeed the best suited underlying neural architecture. We might also consider feeding the network a sequence of inputs, i.e. a sequence-to-vector and vector-to-sequence network (also known as an encoder-decoder) with and without peep-holes and compare the performance of these architectures.

The full notebook is available to fork on Kaggle here: <a href="https://www.kaggle.com/jamesdhope/wiki-competition-time-series-predictor-grucell">https://www.kaggle.com/jamesdhope/wiki-competition-time-series-predictor-grucell</a>.
