# 069：IBM《机器学习（无监督学习、深度学习和强化学习、毕业项目）｜machine learning》中英字幕 p69 30_深度学习的正则化技术.zh_en -BV1eu4m1F7oz_p69-

Now， we've discussed the importance of regularization and the trade off between bias and variance when working with different algorithms in prior courses。

In this current set of videos， we'll discuss different regularization techniques for deep learning specifically and why it's of utmost importance when creating these more complex models。

 So in this section， we're going to cover regularization techniques for deep learning and why it's so important for deep learning specifically。

😊。


We'll touch on dropout as well as early stopping， which are different regularization techniques specific to neural nets。




And with that， we'll also discuss different approaches to optimization when working through our neural nets and why those are important。





So to start off。Technically， a deep neural net is a neural net that has two or more hidden layers and often many more。




So a neural network becomes a deep neural network once we move beyond two layers。

 the importance of regularization when it comes to deep neural nets。





Is that with more and more layers， we can learn more and more complex models。

 and those complex models can often nearly perfectly fit to our training data。

 but due to its complexity will often overfit to that training data and not generalize well to that new data。







Now， with that in mind， let's quickly remind ourselves what regularization is。




Regularization is any modification we make to a learning algorithm that's intended to reduce its generalization error。

 but not necessarily its training error。


So again， we're not optimizing to training error。But rather tweaking our algorithm to optimize her generalization and usually at the price of additional training air。

So for neural nets， there are several means of regularization available to us。




Adding some regularization penalty in our actual cost function is an option。

 and this would be similar to what we saw with Lassso or Rige， for example。

 where more and higher weights will be penalized within our actual cost function。






We have something called drop out， where we'll randomly lose certain neurons in our network to ensure no model is over reliant on any particular neuron or any particular path。





Talk about early stopping， or will just be the idea of stopping gradient descent short so that's not perfectly fit to the training set。





And then with that， to some degree， those ideas of storcchastic and mini batch gradient descent that we discuss in prior videos may ensure that we don't perfectly fit to our training set。





And therefore may generalize a bit better than full batch grading descent。




Now， starting with that first option that we had listed there。

 we can choose a loss function that will penalize our model for having higher weights。 And again。

 this will be similar to Rir Russian if we're trying to predict a numerical value in that we can take something like our mean squared air。

😊。




And add on a regularization term that penalizes the weight squared。




And if we are trying to predict a categorical variable。




Rather than a numericalmer value variable， this will be very similar in that we will be simply adjusting our categorical loss function that we have to penalize again those larger weights。





Now let's move to a method that will be more specific to working with neural networks。

 namely dropout。


Now， with dropout， we'll be randomly removing a subset of the neurons for each batch。




So a common problem that neural nets faced in earlier implementations was that although there are all these different pathways available when optimizing these complex neural nets。




Often they would become over reliant on particular pathways。

While not much learning was done on other pathways through other nodes。




So creating a deeper or denser network， adding on more layers or adding on more nodes would tend to not add that much value to our neural net。





By adding in dropout。Our neural networks will not be able to over relyly on any individual pathway。




And thus， it'll become more robust to overfitting to that particular pathway。




And finally， since we're randomly dropping out neurons and learning on just a subset。

 we then will have to rescale those weights of the neurons at the end to reflect the percentage of the time that that neuron was active during training versus not active。






So we can think of this image that we have here as our standard fully connected feed forward neural network that we are familiar with at this point。





With drop out at each iteration， we end up with something like we see here to the right。

 where we drop a predetermined percentage of the nodes at each one of our layers so that our model is forced to learn iterations with different possible pathways through to our solution。






So what exactly do we mean also when we say that we are rescaling our neurons after training our model？

If you think about it when we are training our model。

 a node is going to have a probability P as we see here to the left of not being present。




At any particular iteration， and thus， the remaining weights are going to be scaled up to make up for that。




Thus when we actually get to testing time and all the weights are present。

We need to ensure that we appropriately rescale our weights。

 and that's why we have P times W at the end。


When all these weights are always present。


Now， another heuristic that was available to us that we discussed earlier is idea of early stopping。




And this just refers to choosing some rules at which to stop training our model to ensure that we don't overfit。




Now an example of this would be to start by checking the loss on a validation set。

 so not the training set， but rather some holdout set that validation set at every 10 epochs。





And if the validation loss at that next step，10 epochs later is higher than it was at the previous step。

 then we would stop training。


So that's the idea of early stopping。


Now that closes out this section on common regularization techniques that are available to us。




In the next videos， we'll discuss another important part of tuning your neural net。




Namely what type of optimizer you're going to use All right， I'll see you there。




![](img/88ca3d3a46cbc424b87612fdc6c73ad9_100.png)