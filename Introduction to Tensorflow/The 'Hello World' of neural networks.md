```python
model = keras.Sequential([keras.layers.Dense(units=1, input_shape=[1])])
```

In keras, you use the word dense to define a layer of connected neurons. There's only one dense here. So there's only one layer and there's only one unit in it, so it's a single neuron. Successive layers are defined in sequence, hence the word sequential.

Two function roles to be aware of **loss** functions, **optimizers**
```python
model.compile(optimizer='sgd', loss='mean_squared_error')
```

The neural network has no idea of the relationship between X and Y, so it makes a guess. Say it guesses Y equals 10X minus 10. It will then use the data that it knows about, that's the set of Xs and Ys that we've already seen to measure how good or how bad its guess was. The loss function measures this and then gives the data to the optimizer which figures out the next guess. So the optimizer thinks about how good or
how badly the guess was done using the data from the loss function. Then the logic is that each guess should be better than the one before.