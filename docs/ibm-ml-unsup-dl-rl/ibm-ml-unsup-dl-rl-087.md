# 087：AlexNet.zh_en -BV1eu4m1F7oz_p87-

Now let's discuss Alexnett now Alex Net is named after Alex Sevsky， one of its main creators。

 and if you watched the intro course that we had here for all of the courses within this learning path。

 we would recall that this was when convolutional neural nets really hit the main stage。





And that was due to the fact that it won the competition here on ImageNe。




The goal of this competition was to predict the correct label from among a thousand different classes。




And this was amongst 1。2 million different images， so we're working with a very large data set or a large classification problem in general。





Now， again， Alexnet is considered the flashpoint for modern deep learning in general。




This is due to the fact that it demolished the competition at a top five air rate of 15。4%。




Whereas the next best was 26。2%。


So let's dive into Alex under the hood。Now here we have an actual diagram that Alex Matt。

 Now don't be too nervous about this breakdown of layers。

 The reason why we have two separate paths that this network is walking through is that in order to run a model on such a large data set。






What they actually did was split it up into two parallel paths。

 so rather than thinking about say this first layer that we have here。

 which we see is 55 by 55 by 48 twice， you can imagine using your normal convolutional layer that this would be something along the lines of 55 by 55 by 96。





Where that depth of 96 is split into two parallel paths。

 And then you see the same with the next layer and every layer moving forward。 So in the next layer。

 you can imagine rather again， than 27 by 27 by 1，28。 It would be 27 by 27 by2，56。

And when we look at this large network， along with those dense fully connected layers at the back end。




There was actually 60 million different parameters that had to be learned。




So that parallelization was very important and also this would take weeks to actually learn。

 but again， had this very high performance of knocking out the competition with that difference that we just saw of 15% to around 26%。






So now I want to go over just a few details， a few more details in regards to Alex Smith。




So first off。The AlexNet developers performed data augmentation before feeding through these images through the network。

And they did things such as cropping， horizontal flipping and other manipulations。

 And that augmentation helped with overfitting。So if you think about working with an image of a cat。

 for example， and you were to crop down that image but still had an image of a cat or did a horizontal flipping。

 but again， doing a horizontal flip of an image， you'd still have that image of a cat。

 you'd be able to avoid overfitting to those exact images and learn the extra features that actually go into what makes up a cat。




Now， the basic template， which we just saw in the last slide。

Is that we'll have convolutions with relo？And。Re lose as those activation functions at is。

 and relos were fairly new to use at the time。 And that was a major part of why they were able to create this huge breakthrough to train such a large network。

😊。




And with that， they would sometimes add on that max pooling layer after convolutional layers。

 and as we've seen before， at the end， there was a fully connected layer that led to that soft max classifier that allowed you to identify the class of that image。






Now that closes out Alex Ne。

![](img/77809e2417c3643fe38d34270f860690_44.png)