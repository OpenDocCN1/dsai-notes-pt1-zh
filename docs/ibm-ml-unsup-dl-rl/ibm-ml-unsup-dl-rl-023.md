# 023：IBM《机器学习（无监督学习、深度学习和强化学习、毕业项目）｜machine learning》中英字幕 p23 22_平均漂移算法.zh_en -BV1eu4m1F7oz_p23-

Here we will be discussing our final clustering algorithm， the mean shift algorithm。Now。

 let's go over the learning goals for this section。




In this section， we're going to cover the mean shift clustering algorithm and how we use the concept of moving towards the highest density to help determine our different clusters。

And then we're also going to discuss the strengths and weaknesses of working with the mean shift algorithm。




Now the meanshift algorithm will work similarly to K means in that we will be partitioning our points according to their nearest cluster centroid。





For Ca means， though， the centroid represented the mean of all points within that cluster。




While with mean shift， that centroid is going to be the most dense point within the cluster。

 which in principle， can be anywhere in that cluster。





And the algorithm will assign points to a cluster by moving to the densest points within a certain window。





So how do we calculate this local density to say where the highest density point is？




In order to do so， we're going to calculate the weighted mean around each point。




So what do we mean here when we are asking for the weighted mean。




We can think of the weighted mean as assigning more weight to those points closer to the original point within our window。




So say we select this black point to start。


We calculate the weighted neme in the local neighborhood or within this window， this pink square。




And it would find that the densest point。Given the weighted mean would be here in pink。

And note on the side that the new mean does not have to be at a data point。




And can be somewhere else within this window。So how do we go about using this to create our different clusters？




So the steps are going to be that you choose a point and a window。

 So we saw that window size start at a random point。

We calculate that weighted mean within that window。




And then we shift the centroid of the window to the new mean。 So we shift that square。

 So it's now perfectly around that new weighted mean that we just found that new denser point。





We then continuously repeat steps 2 and 3 until convergence until there's no shift。

 meaning that we have reached the local density maximum and we'll call this the mode。

 So when the mode is reached。




And then we steps 1 through 4 for all data points。 until finally。

 data points that lead to the same mode will all be grouped together in that same cluster。





So let's visualize how this is done in practice。


So let's visualize how this actually works in practice。So we start with a centroid at a given point。




And then given that window， we sample that local density。

 and then we follow the gradient towards the denser direction。

 So we keep moving towards the highest density。 So we keep reclaiming where that denses point is。

 and we create our new window around it。 and we see we move along each one of our data points。





Until ultimately， we find that local density be maximal and we stop there。




We can do this again， starting at another point。We can sample the local density and again。

 follow that gradient towards the denser direction。

And we see that we move along towards that densest direction。 And again。

 we end up finding that same local maximum。 So we would assign those both to the same cluster。




We can do this again， starting at another point， this time， starting further away。




At a point that will probably lie outside this cluster。

 We sample that local density and follow the gradient towards the denser direction。




And you see that it moves along as we move towards that denser direction。

 and then it finds that local density maximum and stops there。





And to keep going， we can start at each one of the different points， sample that local density。




Follow the gradient towards that denser direction。 And here again。

 we see that that point finds the same local maximum。

 So we would end up labeling it as the same cluster。





And we keep going like this。And eventually， it's going to find for us4 unique local maxima。

 So we see them laid out here。 each one of our four local maxima。




And is going to assign the points to the centroids that they fall into。

So we see here that all of the pink fall under that pink centroid。

We see all the teal values falling next to that teal centroid and all the blue values falling under that blue centroid。

 And now we have， as well， the purple with its purple centroid。

 And we have our four different clusters。


And no cluster numbers needed or any distance parameters need to be defined。

 It's just going to move towards that densest direction and figure out those clusters for us。





Now， let's hone in a bit into what we mean here by this weighted mean。

 That mean that we keep moving towards as we get higher and higher density。





So that new mean is going to be calculated using the sum over points within the window。

 And we see this in both the numerator and the denominator。




We're also going to have this weighting or this kernel function that's going to allow us to give a certain weight。

 according to how far each one of these different points are from the previous mean。






And we see that in the numerator， we weight this according to each point。

 So we're going to weight that and then take the distance of that point and those that have a higher distance or a lower distance will have a higher weight。





And the common kernel that's used is going to be the RBF kernel。

 which is going to be similar to your Gaussian kernel。

 giving more weight again to those values that are closer and less weight。

 according to the normal distribution for those values that are further away。





Now let's talk about some strengths and weaknesses of working with a mean shift。




The mean shift is model free。 It does not assume the number or the shape of each one of our clusters。

 So that's going to be a pro that we didn't see when we worked with something like K means。





We can use just one parameter。 we don't have to tune over more than one parameter like we did with D scan。

 that parameter being the window size or the bandwidth。




And it will be robust to outliers。 We have that window size， and it won't be affected。

 and it can have those outliers outside of each one of our different clusters。





Some weaknesses。


The results will heavily depend on our window size。

 So it's going to depend on the bandwidths that we choose and selection of that window of that bandwidth is not going to be an easy thing to decipher in general。

And also， finally， can be slow to implement。The the complexity is going to be proportional to m N squared。

 where n is going to be the number of iterations that it has to do and N the number of data points。

 So the more data points that it goes is going to be more and more complex。

 You see that it's n squared complexity。 So if we have a large data set。

 this may take a while to converge。





Now let's walk through the syntax that you need in order to perform mean shift using Python。

 So first thing that we want to do is import the class containing that clustering method。

 So from SK learned dot cluster， we import mean shift。




We then create an instance of this class。Setting M S equal to mean shift。

 And we pass in our parameter bandwidth equals 2。


So again， our window here will be equal to two。


And then we fit the instance on the data， and we can use that to predict clusters for new data。

 So we call M that instance of our class dot fit on x1， so it finds our clusters using x1。




And then we can call MS dot predict on x2 to see which clusters they fall under given the new data。




So to recap。In this video， we talked about the meanshift clustering algorithm and how we use the concept of using a window。

 as well as the densesest point within our window to find our different centroids of our clusters。





And we discussed the algorithm's strengths and weaknesses。

 such as not needing to define the number of clusters。

 as well as understanding that this model will have a higher overall complexity。





So with that， we close out our different clustering methods， And in the next video。

 we will compare and contrast all the different methods that we discussed and which ones are best to use for which use cases。





![](img/247ed1d3351f828416859ad35779ab7f_124.png)