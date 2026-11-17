# 074：IBM《机器学习（无监督学习、深度学习和强化学习、毕业项目）｜machine learning》中英字幕 p74 35_分类交叉熵.zh_en -BV1eu4m1F7oz_p74-

So we just went through that notebook introducing Carros。

 and in that notebook we saw that we needed to actually implement some transformations in order to actually ensure that our neural nets performed optimally。

Here we're going to talk about some other important transformations to keep in mind when training our neural net models。

So let's go over the learning goals for this section。In this section。

 we're going to cover prepro and preparing your data for analysis。

 so all the steps that are going to have to come into play when you're thinking about creating a neural network。





Part of that will be if you're doing multi class classification。




How to set it up so that you can predict across multiple classes rather than what we've seen so far where there's just one class or the other。





And then finally， we're going to discuss the importance of scaling your neural net models。

 and we saw this in our notebook as we went ahead and we used the standard Scalar in order to scale our data。

 and you can also use something like the Minmac Scalar， which we've seen in earlier courses。






So for binary classification problems。


One we just trying to decide between two different classes。

We have a final layer with just a single node and a sigmoid activation and we saw that in just our last notebook where we had a full dense network they all connected to that final node。

 there was only one node in that final layer and we had a sigmoid activation function in order to allow for that output。







Now the sigmoid activation function has many desirable properties。




One is that it gives an output strictly between0 and1。

 and that value can be interpreted as a probability so we can say which one is more likely and by how much。






It's going to have a nice derivative， meaning that it's going to be easy to find the gradient as well as to use that to do back propagation。




And it's going to be analogous to logistic regression， or you'll have a bunch of input。

 go into that linearly go into that node， and then you'll have that one nonlinear transformation as you do with logistic regression to output that value again between zero and1。






Now the question is， is there a way to extend this to a multiclass setting if we're trying to predict across multiple classes？




If we want to do this multiclass classification， we can use what we learned in regards to one hot encoding and we use this most frequently when working with different feature variables and we're just going to use that concept for our outcome variable。





So100 coding again is four categories。


And you can take， for example， a vector with length equal to the number of categories。

 so say that your vector just has one value for each category and those different categories are going to be in this case。

 checking， saving and mortgage the type of account that you have there。







You can then represent each category with one at a particular position and zero everywhere else。




So， for example， with our bank account example， rather than just having 1，2，3， we can have。





Three new columns where one of those columns for checking， perhaps that top value was checking。

 we put a one there on top。


And then that second value was savings， so we put a one in the middle and zeros everywhere else。

 and again， that top zero reference to whether that value is going to be checking that bottom one will be whether or not it's mortgage and we put a one at that bottom value because that bottom value was mortgage and zeros everywhere else。








So for multiclass classification problems， we're going to let that final layer be a vector with length equal to the number of possible classes as we just saw on the last slide。





And then we can extend the idea of the sigmoid to multi class classification using this soft mass function。

 and that soft mass function is just going to be the E2。

 whatever that Z output was for a particular class。😊。






Over the sum of E to the Z for all of the classes combined。




And what that does is it's going to yield a vector with entries that are going to be between0 and1。

 normalizing them all to between0 and1， and that will ultimately sum to one。

 so that we can get the probabilities for each one of the individual classes。






Now for the loss function。


When we even input it in， it's going to be categorical cross entropy that we're trying to calculate。




And this is just going to be the log loss function in disguise。

 so we take that cross entropy and that's equal to negative Yi。

 y being the actual values times log of Yi， whatever that prediction is。






And the derivative of this will have a nice property when used within the soft max so that the derivative of that last Z I in regards to that soft max is going to be Y I。

 the prediction minus Y I。


![](img/4159a310f16f4a9e66e5897c5ab9e70d_79.png)