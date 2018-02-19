# Machine Learning and Econometrics: Flexibilty versus Interpretability




*This is the second in a series of posts where I document my own process in figuring out how machine learning relates to the classic econometrics one learns in a graduate program in economics. Note that I am a B+ practitioner of applied econometrics, so the ideas I am working through are not new, or may not even be fully accurate. But, I think many of us applied economists are starting to learn and dabble in this area and I thought it might be fruitful to learn publicly in these blog posts.* 

I had at least three people reach out after my first machine learning post saying that they are starting to use these methods in their own research, and one former classmate in the corporate world who reached out to say these methods are already deeply embeded in their business. It feels like this is a really great time to sort these things out. 

# Flexibility versus Interpretability

I started going through [An Introduction to Statistical Learning](http://www-bcf.usc.edu/~gareth/ISL/index.html), and the first thing that struck me is that linear regression gets two whole chapters devoted to it (Chapters 3 and 6). I was a bit surprised because my niave view was that linear regression was something primitive and completely separate from the concepts of machine learning, wheras in this book, linear regression is presented as a baseline or starting place for the concepts of machine learning. This left me with a question in that why is econometrics as usually presented in an economics graduate program something separate methodologically from machine learning? 

In chapter 2, I think I got the answer to my question. The authors note that in prediction, there is always a tension between flexibility and interpretability of a model. Linear regression is quite inflexible, but it is readily interpretable. Non-linear models can be quite complex in how an independent variable is functionally related to the dependent variable. In economics, we care about interpretability. We go through great pains to find instruments, design experiments, get random samples, in order to argue causaility of one variable on another. Inference and interpretability matter a great deal to us, so inflexible models are often attractive for this purpose. 

Additionally, less flexible models often perform fairly well compared to more flexible models in terms of predictive accuracy, at least in out of sample testing, due to overfitting in flexible models. 

Our analysis is often naturally constrained in terms of the number of independent variables one might include. First, economic theory often suggests what things might be important independent variables, so there is less need to throw everything into a regression and see if it predicts. Or, you might be constrained by the scarcity of relevent data. If you conduct a survey, you can only ask so many questions and then you only have that many potential independent varaibles. To use a concept from my first machine learning post, we often have 'medium data' not big data. 

So I think there are good reasons why econometrics has gravatated to linear regression models.

+ We care very much about inference, less emphasis on prediction $\Rightarrow$ Linear regression is desireable
+ We often have medium data $\Rightarrow$ Linear regression is desireable

So does that mean we should scoff at this machine learning stuff? No I don't think so. Reading into chapter 5 (Resampling Methods) and chapter 6 (Linear Model Selection and Regularization), I think there are really nice opportunities to get a lot of benefit even in a linear model and even with medium data. Chapter 5 covers resampling methods including Cross-Validation and Bootstrap methods. Bootstrap is ubiquitous in econometrics, but cross-validation could be successfully utilized more, I think. Among those of us working with large 'medium data', like the Census or Ag Census, the methods are chapter 6 are fairly commonly employed, I think. 

I'll explore cross validation a little more in the remainder of this post. 

# Cross Validation
