# 015：IBM《机器学习（无监督学习、深度学习和强化学习、毕业项目）｜machine learning》中英字幕 p15 14_维数灾难笔记本第2部分.zh_en -BV1eu4m1F7oz_p15-

Building off of what we just discussed in two dimensions。 So we had our square and our circle。

 And we saw that 21 per cent of our square was outside of the circle。




We are now going to push that out into three dimensions and work with a sphere rather than a circle and a cube rather than a square。




Now， I want to remind you how this ties back to a data science problem。

 The idea here thinking about the two dimensions is that we have for each dimension。

 that is a different feature。 So a different covariate。 So we have covariate A， Covariate B。

 Both have been normalized。 And we can think that the values lie between negative one and one for each one of those。

 and they've been standardized。 So they have a mean of 0 and a standard deviation of one。









If we think about this value and look at the square here。




The idea is that to be a single unit away from that center value。




That would indicate that you're one standard deviation away， whether you're pointing horizontally。

 vertically or diagonally。 And we can see all the different values that lie one standard deviation from the mean。

 That's your unit circle。 And then all values that are outside of that circle are going to relate to those that are far away from the mean above one standard deviation from the mean。

 But still within that negative one to one range。 So we see。

 given that we're working with values between and negative one and one for our covariates。








These are the values outside the circle that will be， in a sense， outliers。




Now， in that same sense， if we were to add on a third dimension Covariate C。

 And that's what we're planning to do here。 The idea is。

 we're still working from negative one to one。 Still。

 our sphere now will indicate one standard deviation away in any direction， now。

 not just diagonally but diagonally within space。





And then anything outside that sphere， we can then again think of in the same sense that we just did with the circle and the square that this is more than one standard deviation away still between negative one and one and see how many outliers we have。





So again， with the square we had 21% now we're moving to plotting in three dimensions。




I'm going to show you step by step how you can plot some of these values in three dimensions so that you can go home or leave this notebook and then be able to plot in three dimensions yourself as well。





So the first thing that we do is we are going to have to import this axes 3D library Now。

 if we don't do this。And we try to createate our figure。




And then from that figure， we get our current axes and make them 3D projections。




We'll see that we get in error。


We will have to first import that library。That axes 3D to give us that option。




So。We pull this in。


And I'm going to run through just as we did before。

 have to cell above so we can see this step by step。 But now that I've imported axes 3D。

 we see that now， rather than working in two dimensions。

 you can see how we can start to work within three dimensions。






So hopefully this is exciting to see that we have values for X， Y axes and then now a z axis as well。





We're then going to draw our cube。 Now we have here this idea of combinations and taking the products。

 I don't want to walk too much into it。 I'll show you quickly how the product works。

 and I would suggest you can look at the combinations and see how it works as well built off of this product that I'm about to create。






But right now， we're taking the product of three R's and the R is just defined as negative 1 and one。




In order to make this a little clear。


We're going to use three different lists of two， rather than negative one and one， though。

 we're going to use one and2。3，4， and。5，6。 And when I take the product。

 I'll take the list so that we can see this output。 otherwise it's just a generator object。




We also have to make sure that we import that library。




You see that it comes up with every possible combination， not accounting for ordering。 So1，3 and 5。

 taking the first value from each of the lists， then1，3 and 6。 So first， first and second。

 and then 1，4，5，1，4，6。



2，3，5 so you can see how it's going through each one of these different values ensuring that it covers all the different possible combinations。





So it does that with negative 1，1。 And then the combinations of value of 2 will give you values of two for each one of different combinations。

 and I wouldn't worry too much about it。 That point here is given again that。






We're pulling out an S and an E， it's going to output two different values when we get that combination。




We're going to take the sum of S minus E， and that has to be equal to。This is using our R1。

 R1 minus r 0 in order for it to be an edge on our Q。So that's all it's trying to do。

 is's trying to find where each of our edges lie。


Now I'm going to pull out this portion of code just to show you how one line is drawn in three dimensional space。





Let me。All this。


We copy this， we're going to move it above。And we're saying for S and E。

 we don't care too much about that。 But what we do care about is this zip of S and E and then plotting that。

 So in order to see what that， well， this the star is going to ensure that it unpacks it。

 So rather than just creating generator object， we'll see that actual output。

 And I'll actually print here so we can see what。




This output looks like。So。Zipping S and E。And then I'm going to break。

 so we're just going to plot one line。


So I'm going to run this。R is not defined yet， if you forgot to copy that in。

 say r equals negative 11。



And we see that we plotted this one line。Now the zip S。

This is going to be our x values of our two points， the y values of our two points。

 and the z values of our two points。


So we're plot from negative one， negative  one， negative  one， up to negative  one， negative  one，1。

 so that's the idea that we're seeing here。And it's hard to see in three dimensional space。

 but we are going from negative one， negative1 negative one up to 111。




Now， when we run through all the different lines， all we're doing is using this plot 3D。

 which will work exactly the same as just plot in two dimensional space。

 And that is just creating those lines connecting those two dots the same way you would do in two dimensional space。

 calling ax dot plot。





So if I don't run the brake here and do this for let the for loop run all the way through。

 you see here that we now have our cube connecting each one of these points that we have here。





The next step that we want to do in order to draw our sphere is we're first going to create this mesh grid。

So I'm going to copy this above into a different cell。And in order to make this a little clear。

 this is going to be the number of points。


If you were to do without the J， the J just in general， so you know。

 within Python means a complex number。


We are working here with the J， not because we're working with complex numbers。

 but the complex numbers just let us know that rather than counting by 20。

 we want 20 points in between 0 and two times pi。



That's all we're doing here by using the complex number。But we're going to reduce this。

 just for our example， to3 and 2， so F。


Three values and two values。 And the idea is that when we want to plot along many different points and we want to cover So here it's supposed to go from0 to two times pi。

 and we want to have three different values， so it'll go0， then pi then two times pi。




And then we're also going from 0 to pi with just two values。 So 0 to pi。

 And the idea is that we want to plot all the possible combinations of these points。

 And in order to do that， we have to create this mesh grid so that we have 0，0， as well as 0 and pi。

 and then。



Pi coming from our count from 0。Through22 pi， we then have pi and 0。

 and then pi and pi for our second axis and so on and so forth。

 So that's the idea of the mesh grid to allow you to plot on each one of these multiple points。 Now。

 it has two outputs。





For each one of the different grids， those are both equal in shape。So you have 0 and 1。

 That's supposed to be your X and Y here， we're plotting in three dimensions。

 And all we're doing is taking that two dimensional graph。





And we're expanding that to create our sphere by using each one of those different points and taking the cosine of each of these values ranging from。





From U， you go from0 to2 pi， and then from V， from 0 to pi。




Multiplying them together。 And then first Z， we just get cosine of V。

 and that will createate our sphere。So we're going to have our three points。

 all these multiple points。 And right now， they're just points out in space。

And in order to connect all those spaces into one final sphere。

 we're going to use this plot wire frame， which will connect all those dots together。😊。





So we call x dot plot wireframe on the X， Y， and z。




And we run this。And then we see all these different points that were created in three dimensional space。

 all being connected by this wire frame。


And ultimately， we saw here how to plot in 3D and may be difficult to visualize how much extra empty space there is。





But if we think about it in terms of the equations。

 the volume of this sphere is given by 4 over 3 pi R cubed。

 And since we're working with a cube with a radius of2 R。

 it's going to have two R cubed in terms of volume。And when we calculate the percent of that cube。

 again， thinking， thinking of this as three different covariates。




We can see that the volume outside the sphere is going to be one minus that volume of the sphere。





Four over three times pi r cubed， over two are cubed。You do some cross multiplication。

 You end up with 1 minus pi over 6， and approximately 48 per cent of your values being outside the cube。

 So working with that same range of negative one to one。






And that same radius being described as your standard deviation and being beyond that being a bit of an outlier。

 we see that 48% cent of our values are now outliers。

 Now that we've moved up to three dimensional space。





So that closes out this video in the next video we will continue and show you how you can actually generalize this to even higher dimensional space and see those percentages as we continuously increase the number of dimensions。





![](img/b80f59e4271847e53f2572a06664fff0_157.png)