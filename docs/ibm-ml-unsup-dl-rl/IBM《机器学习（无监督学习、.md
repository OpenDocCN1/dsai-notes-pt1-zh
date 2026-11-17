# IBM《机器学习（无监督学习、深度学习和强化学习、毕业项目）｜machine learning》中英字幕 -BV1eu4m1F7oz_p94-

Welcome back now in this exercise we're just going to play around with some of the parameters。

 show you some of the parameters that you can play around with。And then。On your own。

 as mentioned earlier， I would suggest you try playing around as well with these max features。

 the max length， as well as something that we won't do the hidden dimension for our R and N。

 I believe we're also going to work with the word embedding dimension and we'll see the performance for each as we move along。

So。


First thing that we have is we set the max features equal to 20，000。

 which is the same as what we had before。And then the other thing that we have is that we're setting the max length。

 so recall that we cap off our sequences or our sentences at a certain length and then pad them accordingly as well。

Here， rather than the 30 that we had before， we're going to pad or truncate at 80。




Everything else that we have here stays the same。The same holds for our H dimension。

 as well as our wordenbeddings， as well as the setup of our model， our R N N model。




So we're just going to run this， we're not going to walk through that again。




Again， we're going to use RMS prop and again the loss will be binary cross entropy。

 the metric that we will track is going to be accuracy along with that loss will automatically be tracked。

We run this and then again， finally we will fit on our training set using our X train and Y train and then having that hold out validation data of X test and Y test。




So we run this and again， this may take some time to run and the only difference again that we have with this sample versus before is that we're setting the max length where we will truncate our sequences up here at 80。

So I'm going to pause here and we'll come back once this training is done。




So that will actually take quite a bit of time to run we're not also going to run the accuracy results as we did before。

 but you can actually see those accuracy results here on the validation set and we see that it went up to 0。

842 compared to what we had before which was 0。7846 and that matches that evaluate output over here So we see we're able to increase our accuracy。







Now we want to see again， we're going to play around with just one more parameter here。




So we have， again， this time， instead of。


20，000 features， which is the amount of actual words we're going to use using the most common words。

 we bring that down to 5，000， keeping that max length up at 80。




And then for our word embedding dimension， recall that that's going to take those integer values that we're starting off with。

And convert them into x dimensional vectors here， 20 dimensional prior they were at 50 dimensional vectors。

 So of shrinking that down as well。 So we're changing two features actually here。




So you run this to get our new Xtrain and X test。


We get our new R&N model， everything else yet the same。




And then after that， again， we will use the compile with the same loss function， the same optimizer。

 tracking the same metrics。And we're going to call fit again here。




And then after that， this is going to run。 and this will take some time as well。

 So all these will take a bit of time and。It's part of the process when we're doing this deep learning。




But here we see it's running through that first epoch。

 and then we'll do that again for another 10 epochs here。

The goal being that if the accuracy on our holdoutet is continuing to improve。

 we should probably run it for more epochs。 And we did see that that was the case up here。

 If We look at the validation accuracy。 We see that it continued to go up。




After each epoch， so we probably could have continued to run that and get even higher accuracy。




So we're actually going to do that here。 we'll see after 10 epochs how well we were able to perform。

 and then after that， I'm just going to run this now。

It'll run for another 10 epoch and we'll see how much that accuracy can actually improve。

 so I'm going to pause it here and we will get back once we are done having both these items。

 which will be quite a bit of time。



Now， looking at our results here are going through now 20 epochs， 10 on the first run。

 another 10 on the next run， and that second 10， of course。

 as it was in our last notebook will pick up where we left off so here we see that we had a actually a 0。

8479。



On the training set， and we see that continues to go up as that loss continues to go down。




And we see towards the ends that we get that validation accuracy of about 0。84， that is。





Around equal to what we were able to accomplish in just 10 epochs。




Using the word embeddings with 50 dimensions， max features are 20000 and the max length of 80。




I'd say again， feel free to play around with these different parameters。

 see if you can improve the model， but we are also going to in the next lecture start to discuss a more powerful recurrent neural net structure with more long term memory。

 specifically LSTMs。


So I'll see you back in lecture where we will pick up with long short term memory models。 All right。

 I'll see you there。


![](img/f11f466a4aaf6b9495c5afc275de31d5_56.png)