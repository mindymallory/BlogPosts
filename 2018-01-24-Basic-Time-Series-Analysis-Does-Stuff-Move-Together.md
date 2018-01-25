# Basic Time-Series Analysis: Does Stuff Move Together (VAR & VECM)?




*This post is the second in a series explaining [Basic Time Series Analysis](http://blog.mindymallory.com/2018/01/basic-time-series-analysis-the-game/). Click the link to check out the first post which focused on stationarity versus non-stationarity. As a reminder, this post is intended to be a very applied example of how use certain tests and models in a time-sereis analysis, either to get someone started learning about time-series techniques or to provide a big-picture perspective to someone taking a formal time-series class where the stats are coming fast and furious. As in the first post, the code producing these examples is provided for those who want to follow along in R. If you aren't into R, just ignore the code blocks and the intuition will follow.* 

In this post I hope to explain how one goes about estimating the relationship among several variables over time. This approach has natural applications in agricultural economics and finance. In ag econ there are commodities whose prices are inherently related because of substitution or complementary effects in production and/or consumption (e.g., corn and soybeans), or because of production processes (e.g., soybeans, soybean oil, and soybean meal). In finance, security prices for companies in a similar sector might be related because of common economic conditions driving profitability (e.g., Bank of America and J.P. Morgan Chase). 

In time-series analysis, there are two basic models typically used to estimate and evaluate the relationships between multiple variables over time.

+ Vector Auto-regression (VAR)
+ Vector Error Correction (VECM)

We will start with the Vector Auto-regression model, because it is the simpler one. Also for simplicity, we will continue as in the first post using SPY (the S&P 500 exchange traded fund) and GS (Goldman Sachs) prices.[^whycomm]

[^whycomm]: If you are wondering why an ag economist like me isn't using commodity prices for my examples, it's because I want to use prices that can be 'pulled' from a data vendor like Yahoo Finance quickly and for free (so spot prices are out), and I don't yet want to introduce the complication of how to create a series of 'nearby' futures contract prices. 


```r
# If you are following along, uncomment the next lines and run once to install the required packages 
# install.packages('ggplot2')
# install.packages('xts')
# install.packages("stargazer")
# install.packages('quantmod')
# install.packages('broom')
# install.packages('tseries')
# install.packages("kableExtra")
# install.packages("knitr")

library(quantmod)
getSymbols(c('SPY', 'GS', 'SHY'))
```

```
## [1] "SPY" "GS"  "SHY"
```


# Vector Autoregression Model 

A VAR model that estimates the relationship between SPY and GS looks like the following. 

\begin{align}
SPY_t &= \beta^{spy}_0 + \beta^{spy}_1SPY_{t-1} + \beta^{spy}_2SPY_{t-2} + \beta^{spy}_3GS_{t-1} + \beta^{spy}_4GS_{t-2} + \nu_{spy} \\
GS_t  &= \beta^{gs}_0  + \beta^{gs}_1SPY_{t-1}  + \beta^{gs}_2SPY_{t-2}  + \beta^{gs}_3GS_{t-1}  + \beta^{gs}_4GS_{t-2}  + \nu_{gs} 
\end{align}

It consists of two linear regression equations, the first explaining the SPY price and the second explaining the GS price. Notice both equations have the exact same explanatory variables. Namely, today's (time *t*) price is explained by yesterdays' SPY price $(SPY_{t-1})$, the day before yesterday's SPY price $(SPY_{t-2})$, yesterday's GS price ($GS_{t-1}$), and the day before yesterday's GS price ($GS_{t-2}$). This is why the VAR is sometimes called a 'reduced form' model. We haven't specified any economic theory about how these equations should be formed. We have simply written them down with exactly equal opportunity for past SPY prices to affect its own and GS prices and for past GS prices to affect its own and and SPY prices. Our estimates of the $\beta$'s might reveal more about the economics. This is in contrast to a 'structural model' where the correct explanatory variables to include on the right hand side would be informed by economic theory. With the VAR, we opt for a fairly general model and let the data do the talking. 

## Levels or Returns?

If you read my first post in the series you should be wondering why in the world it is OK to put the SPY and GS prices into the VAR in levels. After all, we found strong evidence that both are non-stationary. This is true, the VAR model written above would have all the problems of a spurious regression we discussed in the first post. To remedy the situation, we can write the VAR using returns instead of price levels. 

Before we do that, let's note some conventions for writing returns. I see returns denoted most often according to this convention where time *t* SPY returns are expressed as

$$\Delta SPY_t = log(SPY_t) - log(SPY_{t-1}),$$ 

and lagged (time *t-1*) returns are expressed as

$$\Delta SPY_{t-1} = log(SPY_{t-1}) - log(SPY_{t-2}),$$ etc.


People are often loose with whether $\Delta SPY_t$ means logged differences (percent returns) or straight differences (without the logs). In price analysis I think it is almost always preferable to deal with percent changes, so you will mostly see logged differences. Also fairly common is notation like

$$r^{SPY}_t = log(SPY_t) - log(SPY_{t-1}),$$

to more explicitly state that they are using logged differences to create percent returns.

Then, the VAR in returns (using the first notation) is

\begin{align}
\Delta SPY_t &= \beta^{spy}_0 + \beta^{spy}_1 \Delta SPY_{t-1} + \beta^{spy}_2 \Delta SPY_{t-2} + \beta^{spy}_3 \Delta GS_{t-1} + \beta^{spy}_4 \Delta GS_{t-2} + \epsilon_{spy} \\
\Delta GS_t  &= \beta^{gs}_0  + \beta^{gs}_1 \Delta SPY_{t-1}  + \beta^{gs}_2 \Delta SPY_{t-2}  + \beta^{gs}_3 \Delta GS_{t-1}  + \beta^{gs}_4 \Delta GS_{t-2}  + \epsilon_{gs} 
\end{align}

Another thing that you sometimes see is the inclusion of 'exogenous' variables. That is exogenous as opposed to endogenous. In this context, both SPY and GS are endogenous because they get to be on the left hand side of an equation. They are being determined by lags of the variables. Sometimes you want to include an 'exogenous' variable that might be affecting the endogenous variables, but you don't want to give it it's own equation for whatever reason. Let's consider SHY, a 1-3 year treasury bond etf to be an exogenous variable. All that means in the context of a VAR model is that you put lags of the SHY on the right hand side, but SHY doesn't get it's own equation. 


































