# How does Machine Learning fit into Agricultural Economics?




Machine learning and artificial intelligence are the biggest topics in tech right now, and the excitement is spilling over to economics. The 2018 ASSA meetings (I did not attend, just browsed the [program](https://assets.aeaweb.org/assets/production/files/6336.pdf)) had at least five sessions devoted to the topic of machine learning. [Susan Athey](https://www.gsb.stanford.edu/faculty-research/faculty/susan-athey), and [Guido Imbens](https://www.gsb.stanford.edu/faculty-research/faculty/guido-w-imbens) held a wildly popular continuing education workshop at the meetings that was later released as a [webcast](https://www.aeaweb.org/conference/cont-ed/2018-webcasts). At the 2017 ASSA meetings, which I did attend, there was only one or two sessions devoted to the topic of machine learning and AI in economics, but the sessions were standing room only. 

I have to confess I know next to nothing about machine learning and artificial intelligence. I know that it is vaguely connected to the idea of 'Big Data', because of the possibility fitting (or over fitting) flexible functional forms. What I don't know is how big does your data need to be? A lot of empirical work in agricultural economics falls decidedly in the realm of *medium data*. Our sample sizes are respectable, but not Amazon user clicks big. 

One area I am sure it can fit well is Precision Agriculture. If you are a big ag business company, like John Deere for example, and you have data streaming in from client equipment that gives you meter-by-meter measures of field characteristics on millions of acres, I'm sure machine learning tools will be appropriate. For the rest of us I'm not sure, but I want to find out. 

# The Point of this Post

I conviced [Todd Kuethe](https://twitter.com/TKuethe) and [Todd Hubbs](https://twitter.com/jt_hubbs) to try to write a paper with me predicting commercial bank delinquency rates uring neural networks. We're supposed to present this reasearch at the SCC-76 meetings in Kansas City on April 5, and none of us have any experience (at least I don't) with neural networds or machine learning, but this is good motivation to figure it out!

Ag econ is great in that most of our conferences require only a proposal. Some fields make you submit a whole paper just to propose to a conference. I am sure I would never learn anything new if I had to write a whole paper before I committed to stand in front of people and talk about it. I've been meaning to dig into machine learning and see what it is all about for a couple of years. Now, with the committment mechanism of having to produce a paper for the SCC-76 meetings, I will at least get through the basics. 

The reason I'm writing a series of blog posts about this is I think a lot of ag economists are in the same boat as me with regards to machine learning, so I will share the big picture of what I learn here. I'm going to start by going through [An Introduction to Statistical Learning, with Applications in R](http://www-bcf.usc.edu/~gareth/ISL/index.html) by James, Witten, Hastie, and Tibshirani. A PDF of the book is available on the website, as is all the code and data. It is also available for purchase on Amazon. 

![An Introduction to Statistical Learning](images\ISLcover.jpg) 

In this series of posts I'm not going to go through the code examples, those are available on the book's website. Instead, I'll be working through in my own mind how the approaches of machine learning and traditional econometrics are similar and different, and how 'big' does you data need to be to apply machine learning with some success? As I gain insight into these questions, I'll share them here. If you want a primer on machine learning versus traditional econometrics, this talk Susan Athey gave is pretty good.    

<iframe width="560" height="315" src="https://www.youtube.com/embed/Yx6qXM_rfKQ" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>



