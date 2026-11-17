# 013：IBM《机器学习（无监督学习、深度学习和强化学习、毕业项目）｜machine learning》中英字幕 p13 12_余弦距离和Jaccard距离.zh_en -BV1eu4m1F7oz_p13-


So we start here with a bit of a less intuitive distance metric， namely the cosine distance。




So we're going to start off again with two points in two dimensional space just to highlight our example。




And hopefully from the lines that we just drew， it should be clear that this is already shaping out to be much different than the L1 and L2 metrics that we just discussed。




What we really care about with the cosine distance is the angle between these two points。

This metric gives us the cosine of the angle between these two vectors defined by each of these two points。





Which in order to move up to higher dimensions， this formula will still hold of taking that dot product。

 as you see in the numerator over the norm of each point in the denominator。




And the key to the cosine distance is that it will remain insensitive to the scaling with respect to the origin。




That is we can move one of those points as we have here。




Along that same line and that distance would remain the same。




So any two points on that same ray， passing through the origin will have a distance of zero from one another。




And the idea is that we want to see the relationships here between rec scene visits。

 between one point and the other， much more so than we care about the actual physical distance between the two。




So recency being one and visits being one is equal to in regards to the cosine distance and how far away it is。

 recency being 10 and visits being 10。




Maybe it would be along that same ray。So for two vectors that are pointing in the same direction。

 our cosine distance will spit out zero。


It'll think of them as very close or essentially exactly the same。




But for Euclidean distance。It may think of them as very far apart。

 depending on where those values actually lie， even if they are on the same line。

So how is this useful being able to classify them is exactly the same if they are pointing in the same direction？





Let's say we have text data and our features are going to be different counts of different words within the documents。




Now just because one document is longer than the other， so it has more counts of each of these words。





Does not mean that they need to be far away from one another and thus cluster differently。




Maybe they're about the exact same thing。Maybe one of those articles is a summary of the other。




In that case， you want to mark them as close to one another。

 and cosine distance will come in handy in that situation。

 So if you have three counts of the word data science and 10 counts of the word。





Application， and then you had 30 of data science and 100 of application。

 then you probably want to assume that those are along the same category and cluster those together。

 even though their Euclidean distance may be far apart。

 their cosine distance there would have been in the exact same direction and thus zero。








Another advantage of the cosine distance is that it's more robust against this cursive dimensionality。




Euclidean distance can get affected and lose meaning if we have a lot of features。

 as we saw in our initial discussion of that curse of dimensionality。





So our takeaway here is that the best choice of distance is going to heavily depend on what our application is。




Another distance metric to keep in mind is going to be the jackard distance。

 which will be useful for text as well。



And it applies to sets， and an example of this is used pretty often will be something that we walk through here。

 which is the word occurrence， the unique word occurrence。





So say we have a sentence。 A， I like chocolate ice cream。




That set of A is just going to be the unique words in that sentence，I like chocolate ice and cream。





Say sentence B is going to be， do I want chocolate cream or vanilla cream？




So set B is going to be do I want chocolate cream or and vanilla。

 again not counting that second cream， only those unique values。





And then the jackard distance is going to be one minus the amount of value shared。

 So the intersection over that union。 So the shared values of the two sentences over the length of the total unique values between those two sentences。

 And we'll see this example in just a second and the calculation as well。







And it can be used as a different option when we have these text documents to group similar topics together。





So using this example。We can cacate the score between our two sentences and running through it。

 we see that our intersection is going to end up having three words。




And there are nine unique words total。So the distance is going to be 1 to minus 1 third equals2 third or 0。

67， and that will be our distance。



So that closes out our different distance metrics and overall in this discussion。




Just to recap， we discussed the importance of having different measures of distance between our two points。




As well as the applications of distance measures to clustering and how the measures of distance or similarity will ultimately have a large effect on the groupings that we end up creating。




And with that， we discussed the Euclidean as our most common metric where we used our old me that we learned from back in the day of a squared plus B squared equals C squared。

 we discussed the Manhattan distance， which was the absolute value of each distance's individual features all added together。






We discussed the cosine similarity， which highlighted the angle between our points。

 and then finally we discussed the Jaarard distance。

 which was useful to showing the difference in similarities for different sets of values。






All right， I'll see you in the next video。

![](img/67e1f32a2d4e35c107df49b3cfec12d8_102.png)