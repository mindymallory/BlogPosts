# Basic Time-Series Analysis: Does this Cause That, or Do They Move Together (VAR & VECM, and Granger Causality)?




*This post is the third in a series explaining [Basic Time Series Analysis](http://blog.mindymallory.com/2018/01/basic-time-series-analysis-the-game/). Click the link to check out the first post which focused on stationarity versus non-stationarity, and to find a list of other topics covered. As a reminder, this post is intended to be a very applied example of how use certain tests and models in a time-sereis analysis, either to get someone started learning about time-series techniques or to provide a big-picture perspective to someone taking a formal time-series class where the stats are coming fast and furious. As in the first post, the code producing these examples is provided for those who want to follow along in R. If you aren't into R, just ignore the code blocks and the intuition will follow.* 

In this post I hope to explain how one goes about estimating the relationship among several variables over time. This approach has natural applications in agricultural economics and finance. In ag econ there are commodities whose prices are inherently related because of substitution or complementary effects in production and/or consumption (e.g., corn and soybeans), or because of production processes (e.g., soybeans, soybean oil, and soybean meal). In finance, security prices for companies in a similar sector might be related because of common economic conditions driving profitability (e.g., Bank of America and J.P. Morgan Chase). 



In time-series analysis, there are two basic models typically used to estimate and evaluate the relationships between multiple variables over time.

+ Vector Auto-regression (VAR)
+ Vector Error Correction (VECM)

We will start with the Vector Auto-regression model, because it is the simpler one. Also for simplicity, we will continue as in the first post using SPY (the S&P 500 exchange traded fund) and GS (Goldman Sachs) prices. 


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

library(quantmod)
getSymbols(c('SPY', 'GS'))
```

```
## [1] "SPY" "GS"
```

```r
SPYRet                <- log(SPY$SPY.Adjusted) - log(lag(SPY$SPY.Adjusted))
GSRet                 <- log(GS$GS.Adjusted) - log(lag(GS$GS.Adjusted))
time_series           <- cbind(SPYRet, GSRet)
colnames(time_series) <- c('SPY', 'GS') 
```


# Vector Autoregression Model 

A VAR model that estimates the relationship between SPY and GS looks like the following. 

\begin{align}
SPY_t &= \beta^{spy}_0 + \beta^{spy}_1SPY_{t-1} + \beta^{spy}_2SPY_{t-2} + \beta^{spy}_3GS_{t-1} + \beta^{spy}_4GS_{t-2} + \nu_{spy} \\
GS_t  &= \beta^{gs}_0  + \beta^{gs}_1SPY_{t-1}  + \beta^{gs}_2SPY_{t-2}  + \beta^{gs}_3GS_{t-1}  + \beta^{gs}_4GS_{t-2}  + \nu_{gs} 
\end{align}

It consists of two linear regression equations, the first explaining the SPY price and the second explaining the GS price. Notice both equations have the exact same explanatory variables. Namely, today's (time *t*) price is explained by yesterdays' SPY price $(SPY_{t-1})$, the day before yesterday's SPY price $(SPY_{t-2})$, yesterday's GS price ($GS_{t-1}$), and the day before yesterday's GS price ($GS_{t-2}$). This is why the VAR is sometimes called a 'reduced form' model. We haven't specified any economic theory about how these equations should be formed. We have simply written them down with exactly equal opportunity for past SPY prices to affect its own and GS prices and for past GS prices to affect its own and and SPY prices. Our estimates of the $\beta$'s might reveal more about the economics. This is in contrast to a 'structural model' where the correct explanatory variables to include on the right hand side would be informed by economic theory. With the VAR, we opt for a fairly general model and let the data do the talking. 

## Levels or Returns?

If you read my first post in the series you should be wondering why in the world it is OK to put the SPY and GS prices into the VAR in levels. After all, we found strong evidence that both are non-stationary. This is true, the VAR model written above would have all the problems of a spurious regression we discussed in the first post. Writing the VAR using returns instead of price levels will usually remedy the situation, as noted in the first [post](http://blog.mindymallory.com/2018/01/basic-time-series-analysis-the-game/). 


Then, a VAR(2) using price returns (using the $\Delta$ notation to indicate $\Delta SPY_t = log(SPY_t) - log(SPY_{t-1})$) is

\begin{align}
\Delta SPY_t &= \beta^{spy}_0 + \beta^{spy}_1 \Delta SPY_{t-1} + \beta^{spy}_2 \Delta SPY_{t-2} + \beta^{spy}_3 \Delta GS_{t-1} + \beta^{spy}_4 \Delta GS_{t-2} + \epsilon_{spy} \\
\Delta GS_t  &= \beta^{gs}_0  + \beta^{gs}_1 \Delta SPY_{t-1}  + \beta^{gs}_2 \Delta SPY_{t-2}  + \beta^{gs}_3 \Delta GS_{t-1}  + \beta^{gs}_4 \Delta GS_{t-2}  + \epsilon_{gs} 
\end{align}

## What is it Used for?

The VAR model is used to determine the relationship among several variables. Most commonly, one would test for what is called [Granger Causality]()


```r
library(vars)
library(stargazer)

var <- VAR(time_series[2:dim(time_series)[1],], p = 2, type = "const", )
stargazer(var$varresult$SPY, var$varresult$GS, type = 'html', dep.var.labels = c("Equation 1-SPY  Equation 2-GS"))
```


<table style="text-align:center"><tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left"></td><td colspan="2"><em>Dependent variable:</em></td></tr>
<tr><td></td><td colspan="2" style="border-bottom: 1px solid black"></td></tr>
<tr><td style="text-align:left"></td><td colspan="2">Equation 1-SPY Equation 2-GS</td></tr>
<tr><td style="text-align:left"></td><td>(1)</td><td>(2)</td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">SPY.l1</td><td>0.003</td><td>0.216<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td>(0.028)</td><td>(0.055)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">GS.l1</td><td>-0.063<sup>***</sup></td><td>-0.127<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td>(0.014)</td><td>(0.028)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">SPY.l2</td><td>-0.098<sup>***</sup></td><td>-0.135<sup>**</sup></td></tr>
<tr><td style="text-align:left"></td><td>(0.028)</td><td>(0.054)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">GS.l2</td><td>0.018</td><td>0.036</td></tr>
<tr><td style="text-align:left"></td><td>(0.014)</td><td>(0.028)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td style="text-align:left">const</td><td>0.0004</td><td>0.0001</td></tr>
<tr><td style="text-align:left"></td><td>(0.0002)</td><td>(0.0005)</td></tr>
<tr><td style="text-align:left"></td><td></td><td></td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">Observations</td><td>2,787</td><td>2,787</td></tr>
<tr><td style="text-align:left">R<sup>2</sup></td><td>0.021</td><td>0.011</td></tr>
<tr><td style="text-align:left">Adjusted R<sup>2</sup></td><td>0.020</td><td>0.010</td></tr>
<tr><td style="text-align:left">Residual Std. Error (df = 2782)</td><td>0.012</td><td>0.024</td></tr>
<tr><td style="text-align:left">F Statistic (df = 4; 2782)</td><td>14.895<sup>***</sup></td><td>7.955<sup>***</sup></td></tr>
<tr><td colspan="3" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left"><em>Note:</em></td><td colspan="2" style="text-align:right"><sup>*</sup>p<0.1; <sup>**</sup>p<0.05; <sup>***</sup>p<0.01</td></tr>
</table>






























