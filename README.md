# Cirrus SR22 price analysis
Eitan Frachtenberg  
October 07, 2016  




### Data

Let's start by looking at the data collection, copied primarily from controller.com listings (and can be found at https://docs.google.com/spreadsheets/d/1yVOkI8Kzx-MZKUAS6V1GZim6vDXtvC_MnLlQR1pgMj4/edit?usp=sharing). We have a total of 99 airplanes from 2008 to 2015. All of them use Garmin Perspective avionics so are either G3 (53 airplanes) or G5 (46 airplanes). The engine is either Turbo (49 airplanes), Turbo Normalized (17 airplanes) or normally aspirated (33 airplanes). In addition, we look at these features: TKS: 88 airplanes (mostly FIKI); Air conditioner: 88 airplanes; SVT: 99 airplanes; ESP: 83 airplanes; EVS: 74 airplanes; and Yaw damper: 93.

To simplify, let's start by looking at airplane cost simply as a function of total flight time, ignoring top overhauls and airplane features. We'll combine the effect of multiple variables later:


```r
ggplot(data = data, aes(x=TTSN, y=Price, color=Gen)) + geom_point() + geom_smooth(method=glm)
```

![](sr22-cost_files/figure-html/eda-by-gen-1.png)<!-- -->

Note: Prices are in aviation units ($1K) and are *asking prices*. This is important because not all airplanes are realistically priced, and some prices go down significantly at sale (especially for the G5s, it appears).

the plotted lines are just smoothing using linear regression. They don't represent real planes (which are in the points), but rather a statistical attempt to capture the trend, with 95% confidence interval (in gray bands).

Clearly, G5s command a significantly higher asking price premium ($100K+) over G3s for equivalent hours.

### Effect of the engine type

We can look at the G3s and G5s separately for an evluation of the 'Turbo' premium:


```r
ggplot(data = subset(data, Gen=='G5'), aes(x=TTSN, y=Price, color=Turbo)) + geom_point() + geom_smooth(method=glm)
```

![](sr22-cost_files/figure-html/eda-by-turbo-1.png)<!-- -->

```r
ggplot(data = subset(data, Gen=='G3'), aes(x=TTSN, y=Price, color=Turbo)) + geom_point() + geom_smooth(method=glm)
```

![](sr22-cost_files/figure-html/eda-by-turbo-2.png)<!-- -->

There are just too few normally aspirated airplanes in this data set (15 points) to make any sweeping statements, but it appears that Turbo engines do appreciate faster than NAs, likely because of the increased overhhaul cost. It's also interesting to note that NA G3 airplanes appear to cost more than the TN for equivalent hours. I find this counter-intuitive, but it may be explainable by a third variable with more analysis (for example, many TN airplanes have no AC, whereas most N airplanes do).

Also remarkable is that Turbo models (SR22T) command a much higher price premium than TN. Ignoring the debate on the merits of factory turbo vs. after-market, the data suggests a strong market inefficiency on the G3s, offering similar technical capabilities at a much lower price for the TNs. And although the data is too sparse to be conclusive, it doesn't appear like the two turbo models depreciate at significantly different rates over flight hours, so there doesn't appear to be a higher reserve/mx cost associated with either turbo.

### Cost of different features

For a more complete picture of three more variables in isolation, let's look at the mean price of all airplanes when grouped by different variables:


```r
data$Age <- max(data$Year) - data$Year
ggplot(data) +
  geom_bar(aes(Age, Price, fill=Gen), position = "dodge", stat = "summary", fun.y = "mean")
```

![](sr22-cost_files/figure-html/price-by-variable-1.png)<!-- -->

```r
ggplot(data) +
  geom_bar(aes(TKS, Price, fill=Gen), position = "dodge", stat = "summary", fun.y = "mean")
```

![](sr22-cost_files/figure-html/price-by-variable-2.png)<!-- -->

```r
ggplot(data) +
  geom_bar(aes(AC, Price, fill=Gen), position = "dodge", stat = "summary", fun.y = "mean")
```

![](sr22-cost_files/figure-html/price-by-variable-3.png)<!-- -->

G5s show very little depreciation with age so far (at least in asking price; this will likely correct as the used market for G5 grows and becomes more efficient), whereas the G3s show a clearer trend. Depreciation appears to accelerate after five years (perhaps the extended warranty limit).
The TKS and AC options add about $200K on average to the cost of a used plane --- significantly more than the cost of the option for a new airplane. This is not the actual cost premium, as we'll see next. It ignores the combined effect of other variables like age, premium appeareance, etc. In fact, it is likely that airplanes with this options are more "fully loaded" than those without. Cirrus sells the popular GTS package, which offers a discount on planes that bundle many of these features together.


### Linear regression

Finally, let's try to combine all the factors into a single predictive cost model (looking separately at G3 and G5):


```r
fitG5 <- glm(Price ~ Turbo + Age + TTSN + TKS + AC + Warranty.mos + EVS + Useful.load, data = subset(data, Gen=='G5'))
summary(fitG5)
```

```
## 
## Call:
## glm(formula = Price ~ Turbo + Age + TTSN + TKS + AC + Warranty.mos + 
##     EVS + Useful.load, data = subset(data, Gen == "G5"))
## 
## Deviance Residuals: 
##     Min       1Q   Median       3Q      Max  
## -47.367  -13.770   -1.637    7.789   63.565  
## 
## Coefficients:
##                Estimate Std. Error t value Pr(>|t|)  
## (Intercept)  570.967968 315.564905   1.809   0.0829 .
## TurboT        58.099475  24.834423   2.339   0.0280 *
## Age          -22.350652   8.192120  -2.728   0.0117 *
## TTSN          -0.061222   0.026256  -2.332   0.0284 *
## TKSTRUE       52.071122  23.543364   2.212   0.0368 *
## ACTRUE        32.653912  26.474258   1.233   0.2294  
## Warranty.mos   0.269612   0.468686   0.575   0.5705  
## EVSTRUE       39.987428  37.693270   1.061   0.2993  
## Useful.load   -0.007166   0.242136  -0.030   0.9766  
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## (Dispersion parameter for gaussian family taken to be 870.0856)
## 
##     Null deviance: 84915  on 32  degrees of freedom
## Residual deviance: 20882  on 24  degrees of freedom
##   (13 observations deleted due to missingness)
## AIC: 326.5
## 
## Number of Fisher Scoring iterations: 2
```

```r
fitG3 <- glm(Price ~ Turbo + Age + TTSN + TKS + AC + Warranty.mos + EVS + Useful.load, data = subset(data, Gen=='G3'))
summary(fitG3)
```

```
## 
## Call:
## glm(formula = Price ~ Turbo + Age + TTSN + TKS + AC + Warranty.mos + 
##     EVS + Useful.load, data = subset(data, Gen == "G3"))
## 
## Deviance Residuals: 
##    Min      1Q  Median      3Q     Max  
## -36.66  -15.30   -8.29   16.20   85.96  
## 
## Coefficients:
##               Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  696.47663  168.37597   4.136 0.000349 ***
## TurboT        -2.98983   26.79083  -0.112 0.912032    
## TurboTN      -43.44571   22.97161  -1.891 0.070229 .  
## Age          -19.57390    8.85142  -2.211 0.036378 *  
## TTSN          -0.06383    0.01628  -3.922 0.000606 ***
## TKSTRUE       28.12814   17.91492   1.570 0.128964    
## ACTRUE        21.27592   21.05604   1.010 0.321964    
## Warranty.mos  -0.22382    1.50487  -0.149 0.882958    
## EVSTRUE       16.17466   15.02539   1.076 0.291988    
## Useful.load   -0.13509    0.13228  -1.021 0.316943    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## (Dispersion parameter for gaussian family taken to be 882.4427)
## 
##     Null deviance: 177569  on 34  degrees of freedom
## Residual deviance:  22061  on 25  degrees of freedom
##   (18 observations deleted due to missingness)
## AIC: 346.94
## 
## Number of Fisher Scoring iterations: 2
```

One of the most statistically significant factors on asking price is hours TTSN, with an average depreciation of about $61 per hour for G5 ($64 for G3s; recall that G5s don't show a strong asking price depreciation yet). This is perhaps lower than expected (certainly lower than hourly lease cost), but some high-time airplanes already had at least partial overhaul; and it's probably sufficient to cover the overhauls and chute repack reserves. The other hourly costs are consumables and ongoing, so don't affect the buyer much.

The addition of a Turbo engine to a G5 adds about $58K to asking price, although this likely bundles some of the GTS packaging as well, "borrowing" some of the value of features like AC and FIKI. The Turbo engine only adds about $-3K to the G3s, and the Turbo-normalized (TN) in fact reduces the asking price by some $43K!

The other predictive factors with statistical significance and enough samples were TKS/FIKI (adding about $52K for G5, $28K for G3s); AC only adds about $33K to the asking price (G5), again, probably because it's bundled with Turbo most of the times (contrast with the G3s).

The age in years depreciation is treated as a linear variable (averaging about $-20K per year for the G3s), even though it shouldn't be linear (airplanes typically lose a percentage of value every year). Treating this variable on a logarithmic axis yields a more statistically significant effect of age.

Although the number of months still available on the warranty did not turn out to be very statistically significant for G5s, it's interesting to note that the model priced each remaining month at about $270.

ESP and useful load don't have a predictable effect on cost. In fact, EVS cost is probably included in the GTS bundle, and useful load is negatively correlated with cost, because higher useful load implies fewer options.

There were only a few planes with no yaw damper, and all of them had SVT, so these factors were excluded.

Here's an example prediction using this data:

```r
check = data.frame(Gen=as.factor("G5"), Turbo=as.factor("T"), Age=2, TTSN=700, TKS=TRUE, AC=TRUE, Warranty.mos=28, EVS=TRUE, Useful.load=1085)
predict(fitG5, newdata=check)
```

```
##        1 
## 665.9979
```
 Which can also be manually extrapolated as:
 
570.9679685 + (58.0994745 * 1) + (-22.3506519 * 2) + (-0.0612216 * 700) + (52.071122 * 1) + (32.6539124 * 1) + (0.2696123 * 28) + (39.9874278 * 1) + (-0.0071656 * 1085)
