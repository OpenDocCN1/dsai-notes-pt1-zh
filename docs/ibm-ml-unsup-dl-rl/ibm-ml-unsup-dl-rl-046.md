# 046：IBM《机器学习（无监督学习、深度学习和强化学习、毕业项目）｜machine learning》中英字幕 p46 7_前向传播的矩阵表示.zh_en -BV1eu4m1F7oz_p46-

So imagining working with just a single row of data again， so if we have a single data point。

With a certain amount， so that's a single row with a certain amount of features。




Our W1， or our first weight would be a 3 by4 matrix， taking the input values of x 1， x 2 and x 3。

 which would be a 1 by3 matrix and multiplying that one by three matrix by the3 by4 matrix， the W1。






And that will result in our1 by4 matrix， which would be our z1。




We can then pass all the values of Z1。Through our activation function。And that would result in a1。

 another four vector。



In order to make this as clear as possible。And as we saw on the prior side。

 we can think of our input values as a0。


And every a， a0， A1a2 will be the value that's passed as input into the next layer。





And again， our z1 is equal to the dot product of x and W1。




And our A1 is just going to be the activation function of Z1。

 and that will be passed on to the next layer。



Now to take this a step further to see how this computes through the entire process of our neural network for a single row。




We are planning to start with a vector representing that row。

 in our case that was a row vector of length  three。

And plan to end with an output of a row vector of length three as well， which means in this example。

 we're probably performing classification with output of three classes。






Now we showed how we got Z1 as a dot product of x and W1。




That allows us to calculate A1 by taking the activation function of Z1。

Z2 or the second layer of Z is calculated as a linear combination of each of those a ones that we just calculated。





And in order to get a linear combination of the correct dimensions， we have W2。

 which matches up the shape of A1 and our eventual shape of the values we want for a2。

A2 is then again just the activation function of Z2。




And then Z3 is then again the linear combination of that prior output of A2。




So we need a new weight matrix， W3。


And once we get the linear combination of A2， and since this is the final layer。

 we just take the soft max of Z3 in order to give us the predicted probabilities for each one of our different classes and that will be our predicted why。






Now， in practice， we're not just working with a single row at a time。




But rather we'd be working with an entire data set worth a rosese。

But in order to calculate this generalized version of our multi layerer perceptionceptron。

 our equations should look very similar or exactly the same。





So this time we're inputting an n by3 matrix where n is the number of rows still working with three columns though。





And our output should also be an n by3 matrix with a predicted probability for each one of our different rows。





Now， the math should be the same， though， as we saw before， but this time。

 the dot product of X and W is now just going to be an n by4 matrix or whatever the size of our next layer is rather than it just being one by 4。

 we're now n by 4。 So if you imagine again， that that X is an n by 3。





We can have our WBA3 by4 so that we'd end up with an n by4 matrix for z1。




We can then take the activation function for all of the outputs that we get for Z1 and end up with our output from that first layer。




And again， we have the appropriate matrix to get the linear combination of each one of those outputs for each one of the different rows。





And end up with Z2。We pass each of those Z2s through the activation function to get A2。

That A2 will be the output。


For the second layer and input into the third layer。




And that will give us Z3 when we take the linear combination of A2 and W3。




And taking that Z3 now for multiple rows， we can take the soft max。





And end up with predicted probabilities for each one of the three classes for all of our N rows。




So that expands it out to the amount of rows within our entire data set。



![](img/aec49ae5e347cf8ee9ef312d483928ef_75.png)