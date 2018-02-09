# Basic Time-Series Analysis: A Drunk and Her Dog Explain Cointegration and the VECM Model




*This post is the third in a series explaining [Basic Time Series Analysis](http://blog.mindymallory.com/2018/01/basic-time-series-analysis-the-game/). Click the link to check out the first post which focused on stationarity versus non-stationarity, and to find a list of other topics covered. As a reminder, this post is intended to be a very applied example of how use certain tests and models in a time-sereis analysis, either to get someone started learning about time-series techniques or to provide a big-picture perspective to someone taking a formal time-series class where the stats are coming fast and furious. As in the first post, the code producing these examples is provided for those who want to follow along in R. If you aren't into R, just ignore the code blocks and the intuition will follow.* 

My colleague [\@TKuethe](https://twitter.com/TKuethe) showed me this awesome old paper by Michael Murray, [A Drunk and Her Dog: An Illusration of Cointegration and Error Correction](http://www.tandfonline.com/doi/abs/10.1080/00031305.1994.10476017).[^link] Then I found this extension by Aaron Smith and Robin Harrison, [A Drunk, Her Dog, and  Boyfiend: An Illusration of Multiple Cointegration and Error Correction](https://pdfs.semanticscholar.org/2ab1/0ac4252f50f9f0b7b6f22a1b5c7655c6eb40.pdf). The premise of both these papers is that if you follow a drunk out of the bar after a night of drinking, you will observe that their path looks much like a *random walk* - non-stationary. Also, if you have ever watched a dog freely exlporing, their path also looks much like a random walk. The dog will go this way and that, whever it's nose leads it. If however, the dog belongs to the drunk, and the drunk calls periodically for the dog, the dog will stay pretty close to the drunk. So the drunk and her dog go forth wandering aimlessly, together. 

This is exactly concept of cointegration, two (or more) series wandering aimlessly, together (at least never very far apart.)

In this blog post we cover cointeration and most common model for cointegrated time-series, the Vector Error Correction Model (VECM). The most intuitive cases (besides the drunk and her dog) are markets that are related by a production process, like the soybean complex. Soybean crushers buy soybeans and sell meal and oil. Economic theory would suggest the prices of those three commodities to maintain a relationship so that the profitability of soybean crushers trends around some modest number greater than zero. If there were persistant really high profits in the soybean crushing business, more soybean crushing capacity would be built, driving soybean prices higher and meal and oil prices lower. 

This can also come up in the stock market, especially if two companies are destined for similar fates. If you've ever read a trading book, trading this situation is sometimes called pairs trading. In commodities, it would be called spread trading. The idea is you look for two stocks (or commodity contracts) that are cointegrated (usually move together). Then you wait for a day when one goes up and one goes down. You bet that because they are cointegrated this deviation won't last long, so you sell the one whose price went up and buy the one whose price went down. Then, no matter if the prices go up or down generally, as long as the two prices come back in line, you can make money.  

[^link]: I linked the actual journal article, which is gated, but if you google the title you can find the pdf openly available. 

We will go through specific examples of statistical tests for cointegration and many of the things you might do to analyze a set of cointegrated series including: 

+ Johansen Cointegration Tests
+ Fitting and Interpreting a VECM Model
+ Granger Causality of Cointegrated Series
+ 

We will continue our examples using SPY (the S&P 500 exchange traded fund) and SHY (Goldman Sachs) prices. 


```r
# If you are following along, uncomment the next lines and run once to install the required packages 
# install.packages('ggplot2')
# install.packages('xts')
# install.packages('quantmod')
# install.packages('broom')
# install.packages('tseries')
# install.packages("kableExtra")
# install.packages("knitr")
# install.packages("vars")
# install.packages("urca")

library(quantmod)
getSymbols(c('SPY', 'SHY'))
```

```
## [1] "SPY" "SHY"
```

```r
SPY                <- SPY$SPY.Adjusted
SHY                 <- SHY$SHY.Adjusted
time_series           <- cbind(SPY, SHY)
colnames(time_series) <- c('SPY', 'SHY') 
```

# Vector Error Correction Model 

A VECM model adds a single term to the VAR in returns model that we learned in the last post. It adds an $\alpha (\beta_0 + \beta_1 SPY_{t-1} + \beta_2 SHY_{t-1})$ term to each equation. Using SPY and SHY prices, it looks like this.  

\begin{align}
\Delta SPY_t &= \gamma^{spy}_0 + \alpha_1 (\beta_0 + \beta_1 SPY_{t-1} + \beta_2 SHY_{t-1}) + \gamma^{spy}_1 \Delta SPY_{t-1} + \gamma^{spy}_2 \Delta SPY_{t-2} + \gamma^{spy}_3 \Delta SHY_{t-1} + \gamma^{spy}_4 \Delta SHY_{t-2} + \nu^{spy}_t \\
\Delta SHY_t  &= \gamma^{SHY}_0  + \alpha_2 (\beta_0 + \beta_1 SPY_{t-1} + \beta_2 SHY_{t-1}) + \gamma^{SHY}_1 \Delta SPY_{t-1}  + \gamma^{SHY}_2 \Delta SPY_{t-2}  + \gamma^{SHY}_3 \Delta SHY_{t-1}  + \gamma^{SHY}_4 \Delta SHY_{t-2}  + \nu^{SHY}_t 
\end{align}

The $\gamma$'s are regression coeficients on the lagged SPY and SHY returns, just like the $\beta$'s from our VAR blog post. The $\alpha_1 (\beta_0 + \beta_1 SPY_{t-1} + \beta_2 SHY_{t-1})$, and $\alpha_2 (\beta_0 + \beta_1 SPY_{t-1} + \beta_2 SHY_{t-1})$ terms are called error correction terms. 

It's at about this point where you may start appreciating why people say you should take linear algebra if you want to do econometrics. This is really messy looking and it is hard to really understand what is going on, what the heck those error correction terms are, and how on earth they might be useful. 

Rewriting this in matrix form makes it much neater and (I think) aids understanding. If you've had linear algebra, make sure you can reproduce this next step; it will help your understanding of the working parts. If you haven't yet had linear algebra, consider signing up next semester! and try to understand the explanation of what the $\alpha$'s and $\beta$'s are.  

As an intermediate step, write the error correction terms as a 2X1 matrix with the $\alpha$'s in it times a 1X3 matrix with the $\beta$'s and a linar combination of the price levels it in. The rest of it, is just the terms from a VAR written in matrix form too.[^more]

[^more]: What's crazy is that if we do a VECM model on more than two series, say three, there can actually be more than one equlibrium relationship, and thus, more than one error correction term. If that were true, we would need the $\alpha$ matrix to be 3X2 and the equilibrium relationship matrix (with the $\beta$'s in it) would be 2X4, with the second equlibrium relationship in the second row. Don't worry about the details of this, it would be covered at length in a real time series class. 



\begin{align}
\left[
\begin{array}
  SSPY_t \\
 SHY_t 
\end{array} 
\right]

&= 

\left[
\begin{array}
  s\gamma^{spy}_0 \\
 \gamma^{SHY}_0 
\end{array} 
\right]

+ 

\left[
\begin{array}
  s\alpha_1 \\ 
  \alpha_2
\end{array} 
\right]

\left[
\begin{array}
  s\beta_0 & \beta_1 SPY_{t-1} & \beta_2 SHY_{t-1}
\end{array} 
\right]



+ 

\left[
\begin{array}
  s\gamma^{spy}_1  & \gamma^{spy}_2  \\
   \gamma^{SHY}_1  & \gamma^{SHY}_2  \\
  
\end{array} 
\right]

\left[
\begin{array}
  s\Delta SPY_{t-1} \\
   \Delta SHY_{t-1}
\end{array} 
\right]

+ 

\left[
\begin{array}
  s \gamma^{spy}_3 & \gamma^{spy}_4 \\
    \gamma^{SHY}_3 & \gamma^{SHY}_4\\
  
\end{array} 
\right]

\left[
\begin{array}
  s\Delta SPY_{t-2} \\
   \Delta SHY_{t-2}
\end{array} 
\right]

+ 

\left[
\begin{array}
  s\nu^{spy}_t  \\
  \nu^{SHY}_t 
\end{array} 
\right]


\end{align}

Now, the $\beta_0 + \beta_1 SPY_{t-1} + \beta_2 SHY_{t-1}$ is the long run, or equilibrium relationship of the variables. Suppose we fit this VECM model and get estimates of the $\beta$s of $\beta_0 = 0.50$, $\beta_1 = 1$, $\beta_2 = -1$. That means we estimated the long run relationship between the two price series to be $0 = 0.50 + SPY_{t-1} - SHY_{t-1}$, or in other words $SHY_{t-1} = 0.50 + SPY_{t-1}$. 

So if we fit a VECM, the $\beta$'s tell us the equilibrium relationship that the two series like to maintain. This is a random process, though, so there will be periods of fairly large deviation from this relationship. How, exactly, will the two series respond so that the relationship starts getting pushed back in line with its long run equilibrium?

This is governed by the $\alpha$s. Suppose that $SPY$ gets too big relative to $SHY$ so that the long run equlibrium is violated and $0.50 + SPY_{t-1} - SHY_{t-1}>0$. Then to make the series come back into long run equlibrim, $SPY$ needs to go down relative to $SHY$ and $SHY$ needs to go up relative to $SPY$. This means $\alpha_1$ should be negative and $\alpha_2$ should be positive, to push the two prices back together when they drift apart. The magnitude of the $\alpha$'s govern which one repsonds more strongly to disequilibrium and are sometimes called speed of adjustment parameters.

\begin{align}
\left[
\begin{array}
  SSPY_t \\
 SHY_t 
\end{array} 
\right]

&= 

\left[
\begin{array}
  s\gamma^{spy}_0 \\
 \gamma^{SHY}_0 
\end{array} 
\right]

+ 

\left[
\begin{array}
  s\alpha_1 \\ 
  \alpha_2
\end{array} 
\right]

\left[
\begin{array}
  s\beta_0 & \beta_1 & \beta_2
\end{array} 
\right]

\left[
\begin{array}
  sSPY_{t-1} \\
  SHY_{t-1}
\end{array} 
\right]


+ 

\left[
\begin{array}
  s\gamma^{spy}_1  & \gamma^{spy}_2  \\
   \gamma^{SHY}_1  & \gamma^{SHY}_2  \\
  
\end{array} 
\right]

\left[
\begin{array}
  s\Delta SPY_{t-1} \\
   \Delta SHY_{t-1}
\end{array} 
\right]

+ 

\left[
\begin{array}
  s \gamma^{spy}_3 & \gamma^{spy}_4 \\
    \gamma^{SHY}_3 & \gamma^{SHY}_4\\
  
\end{array} 
\right]

\left[
\begin{array}
  s\Delta SPY_{t-2} \\
   \Delta SHY_{t-2}
\end{array} 
\right]

+ 

\left[
\begin{array}
  s\nu^{spy}_t  \\
  \nu^{SHY}_t 
\end{array} 
\right]
\end{align}


Now, let's do one more step of matrix algebra to prepare to talk about how we test whether or not this type of relationship exists in the next section. If I multiply the $\alpha$ matrix with the $\beta$ matrix I should get a 2X3 matrix.



\begin{align}
\left[
\begin{array}
  SSPY_t \\
 SHY_t 
\end{array} 
\right]

&= 

\left[
\begin{array}
  s\gamma^{spy}_0 \\
 \gamma^{SHY}_0 
\end{array} 
\right]

+ 
\left[
\begin{array}
 s\alpha_1 \beta_0 & \alpha_1 \beta_1 & \alpha_1 \beta _2 \\
 \alpha_2 \beta_0 & \alpha_2 \beta_1 & \alpha_2 \beta _2
\end{array}
\right]

\left[
\begin{array}
  sSPY_{t-1} \\
   SHY_{t-1}
\end{array} 
\right]

+ 

\left[
\begin{array}
  s\gamma^{spy}_1  & \gamma^{spy}_2  \\
   \gamma^{SHY}_1  & \gamma^{SHY}_2  \\
  
\end{array} 
\right]

\left[
\begin{array}
  s\Delta SPY_{t-1} \\
   \Delta SHY_{t-1}
\end{array} 
\right]

+ 

\left[
\begin{array}
  s \gamma^{spy}_3 & \gamma^{spy}_4 \\
    \gamma^{SHY}_3 & \gamma^{SHY}_4\\
  
\end{array} 
\right]

\left[
\begin{array}
  s\Delta SPY_{t-2} \\
   \Delta SHY_{t-2}
\end{array} 
\right]

+ 

\left[
\begin{array}
  s\nu^{spy}_t  \\
  \nu^{SHY}_t 
\end{array} 
\right]


\end{align}

## The Johansen Test for Cointegration

Alright, so that is a lot of nitty gritty matrix algebra, that I've been purposefully trying to avoid in this series of blog posts. But, the payoff is that we can talk about how the statistical test for cointegration works in a way that is analagous to t-tests that are more familiar. 

Recall that cointegration is when two or more variables ane known to be non-stationary, yet a linear combination of these exist so that the linear combination is stationary. Notice that the error correction term uses SPY and SHY prices in levels. We know that we cannot put non-stationary variables in a linear regression, yet if SPY and SHY are cointegrated these two regression equations will be ok. Why? Because when you multiply the matrix of $\beta$s times $[SPY_{t-1} SHY_{t-1}]'$ vector it will be stationary because the regression will find the $beta$\s (as long as such a matrix exists) that makes $\beta_0 + \beta_1 SPY_{t-1} + \beta_2 SHY_{t-1}$ stationary. 

Now if we don't know whether or not our variables are cointegration, we will need a test to determine this. The most common is called the Johansen cointegration test. What the Johansen test does is estimates the VECM and determines if the $\alpha \beta$ matrix is 'zero'. I put zero in quotes because the analagous concept of zero in numbers is the determinant in matrices. In a typical linear regression, if you are unsure whether you should include a variable on the right hand side you might just run the model and do a t-test to see if the coeficient on that variable is statistically different from zero. 

The Johansen cointegration test does something similar. It estimates the VECM model and tests whether or not the determinant of the $\alpha \beta$ matrix is zero. If it is not zero, then you have cointegration (probably - I'll explain more in a bit).

## Apply the Johansen Test

I think its really easiest to understand what the previous discussion means by looking at some actual output from a Johansen test. 


```r
library(urca)
johansentest <- ca.jo(time_series, type = "trace", ecdet = "const", K = 3)
summary(johansentest)
```

```
## 
## ###################### 
## # Johansen-Procedure # 
## ###################### 
## 
## Test type: trace statistic , without linear trend and constant in cointegration 
## 
## Eigenvalues (lambda):
## [1]  2.239910e-02  2.926375e-03 -6.389290e-19
## 
## Values of teststatistic and critical values of test:
## 
##           test 10pct  5pct  1pct
## r <= 1 |  8.19  7.52  9.24 12.97
## r = 0  | 71.46 17.85 19.96 24.60
## 
## Eigenvectors, normalised to first column:
## (These are the cointegration relations)
## 
##              SPY.l3     SHY.l3   constant
## SPY.l3       1.0000    1.00000   1.000000
## SHY.l3    -193.8095  -87.47647  -6.896003
## constant 16200.3089 6597.44021 408.951923
## 
## Weights W:
## (This is the loading matrix)
## 
##             SPY.l3        SHY.l3     constant
## SPY.d 1.319447e-06 -1.882505e-04 7.426777e-17
## SHY.d 9.069726e-06  3.262663e-06 7.646965e-18
```

Above is the output from the Johansen trace test for cointegration. There is also a similar test called the eigen test, which I won't cover in this post, but you interpret them the same way. They usually give the same conclusion, but not always. 

Focus on the output that has $r <= 1$ and $r <= 0$. These are the rusults from the actual test. The $r = 0$ line is testing the hypothesis that the rank of the matrix equals zero (think testing if the matrix is '=' zero, i.e., the error correction term doesn't belong). Notice that the test statistic, 64.96, is quite a bit above the one percent critical value of 24.60. Hence we should reject the null hypothesis. 

Next we consider the hypothesis that $r <= 1$, (which now that we have rejected $r = 0$, the test is for $r = 1$ since rank of a matrix is an integer). In this case, the test statistic is 7.56, which is less than all but the 10% critical values. Hence we fail to reject the null hypothesis and conclude the rank = 1. 

This means that there is indeed one cointegrating relationship, and the VECM is appropriate. 

What if we had rejected the $r = 1$ hypothesis? Then we would have to conclude the rank = 2. This would indicate the series are stationary after all. 

## Fit the VECM 

The fitted VECM is actually delivered from the call to `ca.jo`, but it is put into a bit more intuitive form by passing the `johansentest` object to the `cajorls()` function. 


```r
t <- cajorls(johansentest, r = 1)
t
```

```
## $rlm
## 
## Call:
## lm(formula = substitute(form1), data = data.mat)
## 
## Coefficients:
##          SPY.d       SHY.d     
## ect1      1.319e-06   9.070e-06
## SPY.dl1  -4.343e-02   9.060e-04
## SHY.dl1   5.881e-01  -1.204e-01
## SPY.dl2  -5.033e-02   7.980e-04
## SHY.dl2   1.073e-01  -5.867e-02
## 
## 
## $beta
##                ect1
## SPY.l3       1.0000
## SHY.l3    -193.8095
## constant 16200.3089
```

In the 'Coefficients' output, the $\alpha$'s and $\gamma$'s are given. Specifically, $\alpha_1$, the coeficient on the error correction term in the SPY equation, is 1.319e-06, and $\alpha_2$ is 9.070e-06. The remaining are the $\gamma$'s, the coeficients on the lagged return variables. 

The '$beta' output gives the error correction term. Here we have that $\beta_0$ is  16200.3089, $\beta_1$ is 1, and $\beta_2$ is -193.8095.

Notice that information is also present in the previous output. The $\alpha$'s are the first column of the 'loading matrix', and the $\beta$'s are the first column of the eigenvectors matrix. 

## What does this tell you?

In my opinion, this model provides more opportunity for 'economic analysis' than the previous models we covered in this series. The $\beta$'s tell you what the long run equilibrium among the series looks like, which should be influenced by various economic factors. For example, in [this]() paper my coauthors and I showed that a VECM model on one year deferred corn, ethanol, and natural gas prices produces an error correction term that looks an aweful lot like what you would get if you calculate a no-profit condition in the ethanol industry. 

Finally, as we mentioned before, the $\alpha$'s tell you how fast the series tend to move back together when they get out of whack, which speaks to the nature of frictions in the markets that might allow departures from long-run equilibrium. 

## Granger Causality on Cointegrated Series

You can also do Granger causality analysis on cointegrated series, but be warned you should take care in how you execute it. See Dave Giles' excellent [post](http://davegiles.blogspot.com/2011/10/var-or-vecm-when-testing-for-granger.html) about it, and [this](https://datazen.info/toda-yamamoto-implementation-in-r/) post of how to implement the test in R. 


# That's It! 

That covers the basics of what a VECM model is and how to analyze how a group of markets move around an equilibrium relationship. 

The final post in this series will summarize the series and provide a 'cook-book' like decision tree on how to decide which model is appropriate for your case. 

# References

Murray, M. P. (1994). A drunk and her dog: an illustration of cointegration and error correction. *The American Statistician*, 48(1), 37-39.   

Johansen, S. (1991) Estimation and Hypothesis Testing of Cointegration Vectors in Gaussian Vector Autoregressive Models, *Econometrica* 59 (6): 1551-1580.































