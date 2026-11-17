# 109：IBM《机器学习（无监督学习、深度学习和强化学习、毕业项目）｜machine learning》中英字幕 p109 70_GAN的工作原理.zh_en -BV1eu4m1F7oz_p109-

In this video， we're actually going to take a look at the generative adversarial network training procedures。

So diving right in。The first step will be as usual with our neural networks to start with randomly initialized weights for our generator and discriminator networks。




Now to start our generator process， which is meant to find some distribution that does well in representing the actual values in our training set。

 be those images or whatever it may be。



Will be to pass through just a random noise vector。




So our input's going to be a random noise vector， and the goal is that our generator network will be able to take this random noise and come up with the appropriate weights to generate the complex distributions of our images of whatever our training set is。






And then that generated image should then be the same size as the actual training set images as it will later be fed through our discriminator network。





The next step， step three。Will be for the discriminator to predict the probability that the generated image is a real image or one that our network has just created。





So once again， that is going to pass through our discriminator network， our produce image。

 and the goal is to discriminate between actual training images and those our generator network is producing。






And the output will be the probability that we are working with an actual image and not a generated one。





Step 4。


We are then going to want to compute the losses， both assuming that the image we just generated is fake。




And a loss function assuming that the image generated was real。

 and we'll get into this in just a second。


We're going to call the loss function assuming it was fake， which it is， of course， just a reminder。

 we are working here in the realm of that generated image。





So we're going to call the loss function assuming it was fake L0。

 as in how far away was our discriminator from predicting there's a zero probability that this generated image is not real。





And then we're going to call the loss function assuming it was real， which again， it's not。

 we're going to call that loss function L1。


Representing how far discriminator was from predicting this was 100% a real image。




And then these two combined will produce our total loss function。

 which we can later leverage to train both our generator and discriminator networks。





Now， here in step five， we'll make a little bit clear why we have these two different loss functions。




So our discriminator network is going to only care about correctly predicting that this image is not a real image。




So I'll want to use that loss function L0。One， we're talking about the discriminator network。

So how far off was the output from predicting zero is what we're trying to get here。

 so we back propagate in relation to the loss。Of。


How far off we were to saying that this is not a real image。

And then we update the weights of only our discriminator network accordingly。




Now， where does our L1 penalty come into play？Our L1 penalty。Again。

 the L1 being assuming that our image produced was actually real。

That's going to be ignored by our discriminator network because our discriminator network doesn't want to optimize assuming that a fake image is real。





The goal of computing L1 is to ultimately tell us if we're doing a good job of producing an image that seems realistic。





And thus， our output from our discriminator is too far from one。




So we compute our gradients in regards to L1 and backpo through without training the discriminator and pass that out ultimately to our generator。





So rather than using that gradient to train our discriminator network。

We use it to actually update our weights for our generator network。




So we continue to back propagate using that L1 gradient to update the weights of our generator。




Now what's missing in this training procedure that we just walked through？




The goal of our discriminator network is to learn to classify fake images as fake。

 real images as real。



And for it to actually improve its ability to distinguish between fake and real images。




We're also going to have to give it images from the actual training set。




So our next step。


Is for the real images within our training set。To be passed through。

And we want to calculate the probability that。The image passed through from our actual training set is real。




So we passed that real image XR before we were working with XG that X generated。

 and want to know the probability that this is actually a real image。

 and that's going to be the goal of the discriminator now。





We then compute a new L1 loss for our real image。So again， when trying to predict real。

We're actually going to want our discriminator to output one。

 so we're actually going to use this also to update our discriminator function。




And the L1 in regards to Xr is going to be how far off our probability was。




What should be as close to one as possible from that value of one？




And we can use that L1 loss。To train and update our weights appropriately within that discriminator network。





And we repeat this procedure with new random noise from the generator each time。




And continue until images from the generator begin to look real。




Now， we must note that values of losses from discriminator and generator may still be fluctuating when the generator is producing realistic images。

 so we can't use these alone to determine when to stop training。






And there are ways of quantifying image quality within our generative process such as the inception score to help us find where we should actually stop training。

 that's a bit beyond the scope of the lesson， but I encourage you looking into it if you want to delve a bit deeper into working with GANS。






![](img/18038320176b6285b3d6e4006151f6de_103.png)