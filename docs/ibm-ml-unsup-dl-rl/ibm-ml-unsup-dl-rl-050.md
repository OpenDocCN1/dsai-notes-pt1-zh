# 050：IBM《机器学习（无监督学习、深度学习和强化学习、毕业项目）｜machine learning》中英字幕 p50 11_梯度下降基础.zh_en -BV1eu4m1F7oz_p50-

In this section， we will discuss gradient descent， one of the most crucial components that make learning the parameters of our neural nets actually possible。

So the learning goals for this section are going to include going over basic gradient descent。

As well as how to do stoacastic gradient descent or update our parameters by going through one observation at a time。

And then finally， mini batch grading descent， which will update our parameters using just a certain subset amount of our observations within our data。




So four gradientd descent。We're going to start with a cost function， J of beta。

 which looks like the following。And our goal is to find the beta。

The beta values at which this cost function is minimized。Now， in order to find that beta。

 which we see as our x axis， we initialize at some random point on our cost function。

And then we use the gradient to gradually descend towards that minimum value。

 and that minimum value means we minimize the cost function and that will be our optimal value for the beta。




Now let's discuss an example with linear Russian to help make this a bit clear。

So imagine that you're working with a simple regression or trying to learn two beta values。

 beta n and beta1。


Now in three dimensions， we have on one axis all the possible values for beta not。




On the other， we have all the possible values for beta 1 and on that final axis， that vertical axis。

 we have the output of the cost function we're trying to minimize for all values of beta n and beta 1。




Now that we have increased the number of dimensions。

 we now have a more complicated surface on which to find that minimum value。





So how do we find that minimum value if we can't tell exactly how this cost function will look for our given model？




The key again， is to start at a random point。



We then compute the gradients of this point in respect to beta n and beta 1。




And that gradients will always point in direction of the largest increase。




Now we take the negative of that gradient， and now we are pointing in the direction of the largest decrease。




Now， that gradients that we are discussing will be a vector in that same dimensional space as our parameters。

 consisting of the partial derivatives of each one of those parameters。




So we have the gradients， and that's going to tell us the direction of descent for each one of our individual parameters。





As we see here， we have for every single parameter each one of their partial derivatives。




We can then use the gradients and the given cost function to calculate a new point from that original initialized point。





So we started off with W， now w1 will equal W not minus a learning rate。

 which we'll discuss in just a second， multiplied by the gradient of our cost function。






And we can see that we have now moved closer to minimizing our cost function as we move down and we subtract that gradient。





Now that learning rate is going to be a tunable parameter that will tell us how large we want to make each one of our steps within our cost function。





And we want to be careful with this because too large of a step and we'll end up overshooting our minimum。




And too small of a step， and itll take too long to optimize our model。




Now， using the same concept of subtracting the gradient。

 we can iterate to move closer and closer to the minimum value from that last step。





So now W2 is going to be equal to that W1 we just calculated minus the gradient of the cost function。





And you see we move closer with W2， and then W3 will be the same thing subtracting from the weights we got from W2。

 the gradients。


And we say we move a bit closer。And eventually we end up with a global minimum。



![](img/5ae29fca941f85d1b8864cb9be594036_57.png)