# 075：IBM《机器学习（无监督学习、深度学习和强化学习、毕业项目）｜machine learning》中英字幕 p75 36_卷积神经网络（CNN）简介.zh_en -BV1eu4m1F7oz_p75-

In this set of videos， we're going to introduce a new neural net architecture called Convolutional neural Nes。

And this has been incredibly powerful for image recognition and is now being used to solve many other tasks as well。

So in order to cover this topic， we're going to discuss convolutional neural networks in general and a bit about that architecture with that original motivation of working with image data in mind。






And then we're going to go over many common terms that you're going to need to know and we'll understand much deeper as we go through these videos such as grid size。

 padding， pooling， and depth。



Now let's start off with the motivation behind these convolutional neural networks。

So if we imagine an image and the way that an image works is that each one of the different pixels will have a different numerical value to give you the density within the red green blue spectrum。

 we can think about it on the gray scale to start， but the idea is that there's going to be some type of relationship between each one of our different pixels。

 which are going to be each one of our different features。






And the structure of our neural network so far treats all of our inputs interchangeably。

 and that the relationship or that spatial arrangement of those features have no impact on our model。





So there's no relationship between these individual features。

 you just have this ordered set of variables， feature one， feature  two， and so on。

 and what we want is to be able to incorporate our domain knowledge of how images are actually built。





In building out our neural net architecture。


Now again， these convolutional networks that we discuss here were developed to deal with image data and the motivation behind them will become clear as we work through examples involving image data。

 but increasingly as I mentioned before， these approaches are also being applied in other common analytical problems of regression and classification。

 such as working with time series。





Now， some thoughts to keep in mind when diving into the motivation behind working with a new architecture。

The variables。And in this case， the variables are our pixels。Are going to have a natural topology。

 they're going to have this spatial component that's actually meaningful。

And this makes images different from， say， loan default prediction where the variables do not have this natural topology or relationship in space from one to another。





We'll also want translation and variance， so when we're trying to identify whether there's a certain object in the image。

We want to ensure that it doesn't matter the size of that object or the orientation of that object。

 it'll be translation andvari。We also want our model to be able to appropriately handle issues of pixel densities changing due to lighting and contrast。




Convolutional neural nets are going to be based on what we know about the structure of images and also what we know about the human visual system。




The human visual system has receptive fields， which respond to horizontal bars， vertical bars， etc。

 and pieces them together。


Within data， many of the pixels， which again are going to be our features。

 will actually tend to have fairly similar values that perhaps won't add much information on their own。

 and we want to keep that in mind as well。



Within these images， we're also going to want to be able to identify edges and shapes that exist within that data。

And then finally， you will also want to ensure that it's scale invariant， again。

 meaning that it will classify an object within that picture as a cat。

 no matter the size of that object， so again， this idea of invariance。

 whether it's the size or the orientation。

![](img/7664e344d33207681664119f4442a7a4_37.png)