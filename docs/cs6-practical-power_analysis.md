

# Power analysis

## Objectives
:::objectives
**Questions**

- What is power analysis?
- How can I use power analysis to design better experiments?

**Objectives**

- Be able to perform power analysis in R
- Understand the importance of effect size
- Use power, significance level and effect size to optimise your experimental design
:::

## Background
All hypothesis tests can be wrong in two ways:

* we can appear to have found a significant result when there really isn’t anything there: a false positive (or Type I error), or
* we can fail to spot a significant result when there really is something interesting going on: a false negative (or Type II error).

The probability of getting a false positive in our analysis is precisely the significance level we use in our analysis. So, in order to reduce the likelihood of getting a false positive we simply reduce the significance level of our test (from 0.05 down to 0.01 say). Easy as that.

Unfortunately, this has unintended consequences (doesn’t everything?). It turns out that reducing the significance level means that we increase the chance of getting false negatives. This should make sense; if we’re increasing the barrier to entry in terms of acceptance then we’ll also accidentally miss out on some of the good stuff.

**Power** is the capacity of a test to detect significant different results. It is affected by three things:

1. the effect size: i.e. how big of a difference do you want to be able to detect, or alternatively what do you consider a meaningful effect/difference to be?
2. sample size
3. the significance level

In an ideal world we would want to be carrying out highly powerful tests using low significance levels, to both reduce our chance of getting a false positive and maximise our chances of finding a true effect.

**Power analysis** allows us to design experiments to do just that. Given:

* a desired power (0.8 or 80% is considered pretty good)
* a significance level (0.05 or 5% is our trusty yet arbitrary steed once again)
* an effect size that we would like to detect

We can calculate the amount of data that we need to collect in our experiments. (Woohoo! it looks as if statistics will actually give us an answer at last rather than these perpetual shades-of-grey "maybes").

The reality is that most of the easily usable power analysis functions all operate under the assumption that the data that you will collect will meet all of the assumptions of your chosen statistical test perfectly. So, for example, if you want to design an experiment investigating the effectiveness of a single drug compared to a placebo (so a simple t-test) and you want to know how many patients to have in each group in order for the test to work, then the standard power analysis techniques will still assume that all of the data that you end up collecting will meet the assumptions of the t-test that you have to carry out (sorry to have raised your hopes ever so slightly 😉).

### Effect size
As we shall see the commands for carrying out power analyses are very simple to implement apart from the concept of effect size. This is a tricky issue for most people to get to grips with for two reasons:

1. Effect size is related to biological significance rather than statistical significance
2. The way in which we specify effect sizes

:::highlight
With respect to the first point a common conversation goes a bit like this:

me: "So you’ve been told to carry out a power analysis, eh? Lucky you. What sort of effect size are you looking for?"

you: "I have no idea what you’re talking about. I want to know if my drug is any better than a placebo. How many patients do I need?"

me:	"It depends on how big a difference you think your drug will have compared to the placebo."

you: "I haven’t carried out my experiment yet, so I have absolutely no idea how big the effect will be!"

me:	<sigh>

(To be honest this would be a relatively well-informed conversation: [this is much closer](https://www.youtube.com/watch?v=Hz1fyhVOjr4) to how things actually go)
:::

The key point about effect sizes and power analyses is that you need to specify an effect size that you would be interested in observing, or one that would be biologically relevant to see. There may well actually be a 0.1% difference in effectiveness of your drug over a placebo but designing an experiment to detect that would require markedly more individuals than an experiment that was trying to detect a 50% difference in effectiveness. In reality there are three places we can get a sense of effect sizes from:

1. A pilot study
2. Previous literature or theory
3. Jacob Cohen

[Jacob Cohen](https://en.wikipedia.org/wiki/Jacob_Cohen_(statistician)) was an American statistician who developed a large set of measures for effect sizes (which we will use today). He came up with a rough set of numerical measures for "small", "medium" and "large" effect sizes that are still in use today. These do come with some caveats though; Jacob was a psychologist and so his assessment of what was a large effect may be somewhat different from yours. They do form a useful starting point however.

There a lot of different ways of specifying effects sizes, but we can split them up into three distinct families of estimates:

1. **Correlation estimates**: these use R<sup>2</sup> as a measure of variance explained by a model (for linear models, anova etc. A large R<sup>2</sup> value would indicate that a lot of variance has been explained by our model and we would expect to see a lot of difference between groups, or a tight cluster of points around a line of best fit. The argument goes that we would need fewer data points to observe such a relationship with confidence. Trying to find a relationship with a low R<sup>2</sup> value would be trickier and would therefore require more data points for an equivalent power.
2. **Difference between means**: these look at how far apart the means of two groups are, measured in units of standard deviations (for t-tests). An effect size of 2 in this case would be interpreted as the two groups having means that were two standard deviations away from each other (quite a big difference), whereas an effect size of 0.2 would be harder to detect and would require more data to pick it up.
3. **Difference between count data**: these I freely admit I have no idea how to intuitively explain them (shock, horror). Mathematically they are based on the chi-squared statistic but that’s as good as I can tell you I’m afraid. They are, however, pretty easy to calculate.

For reference here are some of Cohen’s suggested values for effect sizes for different tests. You’ll probably be surprised by how small some of these are.

| Test| Small | Medium | Large |
|:- |:- |:- |:- |
|t-tests| 0.2 | 0.5 | 0.8 |
|anova | 0.1 | 0.25 | 0.4 |
|linear models | 0.02 | 0.15 | 0.35 |
|chi-squared | 0.1 | 0.3 | 0.5 |

We will look at how to carry out power analyses and estimate effect sizes in this section.

## Packages
We will be using the `pwr` package during this section, which contains functions that allow us to perform power calculations. Please install it now.

You can do this by running the following code in your console:


```r
install.packages("pwr")
```

Next, load it by running:


```r
library(pwr)
```

## Section commands
New commands used in this section:

| Function| Description|
|:- |:- |
|`pwr.t.test()`| Power analysis for 1-sample, 2-sample and paired t-tests |
|`pwr.f2.test()`| Power analysis for linear models |
|`cohen.ES()`| Conventional effect sizes for all tests |
|`cohens_d()`| Compute the effect size for t-test |

## t-tests
Let’s assume that we want to design an experiment to determine whether there is a difference in the mean price of what male and female students pay at a cafe. How many male and female students would we need to observe in order to detect a "medium" effect size with 80% power and a significance level of 0.05?

We first need to think about which test we would use to analyse the data. Here we would have two groups of continuous response. Clearly a t-test.

### Get the effect size
Now we need to work out what a "medium" effect size would be. In the absence of any other information we appeal to Cohen’s conventional values:


```r
cohen.ES(test = "t", size = "medium")
```

```
## 
##      Conventional effect size from Cohen (1982) 
## 
##            test = t
##            size = medium
##     effect.size = 0.5
```

This function just returns the default conventional values for effect sizes as determined by Jacob Cohen back in the day. It just saves us scrolling back up the page to look at the table I provided. It only takes two arguments:

*	test which is one of
    * "t", for t-tests,
    * "anova" for anova,
    * "f2" for linear models
    * "chisq" for chi-squared test
* size, which is just one of "small", "medium" or "large".

The bit we want is on the bottom line; we apparently want an effect size of 0.5.
For this sort of study effect size is measured in terms of Cohen’s d statistic. This is simply a measure of how different the means of the two groups are expressed in terms of the number of standard deviations they are apart from each other. So, in this case we’re looking to detect two means that are 0.5 standard deviations away from each other. In a minute we’ll look at what this means for real data.

### Carry out a power analysis
We do this as follows:


```r
pwr.t.test(d = 0.5, sig.level = 0.05, power = 0.8,
           type = "two.sample", alternative = "two.sided")
```

```
## 
##      Two-sample t test power calculation 
## 
##               n = 63.76561
##               d = 0.5
##       sig.level = 0.05
##           power = 0.8
##     alternative = two.sided
## 
## NOTE: n is number in *each* group
```

The first line is what we’re looking for `n = 63.76` tells that we need 64 (rounding up) students in each group (so 128 in total) in order to carry out this study with sufficient power. The other lines should be self-explanatory (well they should be by this stage; if you need me to tell you that the function is just returning the values that you’ve just typed in then you have bigger problems to worry about).

The `pwr.t.test()` function has six arguments. Two of them specify what sort of t-test you’ll be carrying out
* `type`; which describes the type of t-test you will eventually be carrying out (one of `two.sample`, `one.sample` or `paired`), and
* `alternative`; which describes the type of alternative hypothesis you want to test (one of `two.sided`, `less` or `greater`)

The other four arguments are what is used in the power analysis:

* `d`; this is the effect size, a single number calculated using Cohen’s d statistic.
* `sig.level`; this is the significance level
* `power`; is the power
* `n`; this is the number of observations per sample.

The function works by allowing you to specify any three of these four arguments and the function works out the fourth. In the example above we have used the test in the standard fashion by specifying power, significance and desired effect size and getting the function to tell us the necessary sample size.

We can use the function to answer a different question:

> If I know in advance that I can only observe 30 students per group, what is the effect size that I should be able to observe with 80% power at a 5% significance level?

Let's see how we do this:


```r
pwr.t.test(n = 30, sig.level = 0.05, power = 0.8,
           type = "two.sample", alternative = "two.sided")
```

```
## 
##      Two-sample t test power calculation 
## 
##               n = 30
##               d = 0.7356292
##       sig.level = 0.05
##           power = 0.8
##     alternative = two.sided
## 
## NOTE: n is number in *each* group
```

This time we want to see what the effect size is so we look at the second line and we can see that an experiment with this many people would only be expected to detect a difference in means of `d = 0.74` standard deviations. Is this good or bad? Well, it depends on the natural variation of your data; if your data is really noisy then it will have a large variation and a large standard deviation which will mean that 0.74 standard deviations might actually be quite a big difference between your groups. If on the other hand your data doesn’t vary very much, then 0.74 standard deviations might actually be a really small number and this test could pick up even quite small differences in mean.

In both of the previous two examples we were a little bit context-free in terms of effect size. Let’s look at how we can use a pilot study with real data to calculate effect sizes and perform a power analysis to inform a future study.

Let’s look again at the `fishlength` data we saw in the first practical relating to the lengths of fish from two separate rivers. This is saved as `data/tidy/CS6-fishlength.csv`.


```r
# read in the data
fishlength <- read_csv("data/tidy/CS6-fishlength.csv")

# look at the data
fishlength
```

```
## # A tibble: 68 × 3
##       id river   length
##    <dbl> <chr>    <dbl>
##  1     1 Guanapo   19.1
##  2     2 Guanapo   23.3
##  3     3 Guanapo   18.2
##  4     4 Guanapo   16.4
##  5     5 Guanapo   19.7
##  6     6 Guanapo   16.6
##  7     7 Guanapo   17.5
##  8     8 Guanapo   19.9
##  9     9 Guanapo   19.1
## 10    10 Guanapo   18.8
## # … with 58 more rows
```

```r
# summarise the data
fishlength %>% 
  select(-id) %>% 
  group_by(river) %>% 
  get_summary_stats(type = "common")
```

```
## # A tibble: 2 × 11
##   river   variable     n   min   max median   iqr  mean    sd    se    ci
##   <chr>   <chr>    <dbl> <dbl> <dbl>  <dbl> <dbl> <dbl> <dbl> <dbl> <dbl>
## 1 Aripo   length      39  17.5  26.4   20.1   2.2  20.3  1.78 0.285 0.577
## 2 Guanapo length      29  11.2  23.3   18.8   2.2  18.3  2.58 0.48  0.983
```

```r
# visualise the data
fishlength %>% 
  ggplot(aes(x = river, y = length)) +
  geom_boxplot()
```

<img src="cs6-practical-power_analysis_files/figure-html/unnamed-chunk-7-1.png" width="672" />

```r
# perform t-test
fishlength %>% 
  t_test(length ~ river)
```

```
## # A tibble: 1 × 8
##   .y.    group1 group2     n1    n2 statistic    df       p
## * <chr>  <chr>  <chr>   <int> <int>     <dbl> <dbl>   <dbl>
## 1 length Aripo  Guanapo    39    29      3.64  46.9 0.00067
```

From the summary statistics we can see that there are 39 observations for the Aripo river and only 29 observations for the Guanapo river. From the box plot we see that the groups appear to have different means and from the t-test analysis we can see that this difference is significant.

Can we use this information to design a more efficient experiment? One that we would be confident was powerful enough to pick up a difference in means as big as was observed in this study but with fewer observations?

Let’s first work out exactly what the effect size of this previous study really was by estimating Cohen’s d using this data.


```r
fishlength %>%
  cohens_d(length ~ river, var.equal = TRUE)
```

```
## # A tibble: 1 × 7
##   .y.    group1 group2  effsize    n1    n2 magnitude
## * <chr>  <chr>  <chr>     <dbl> <int> <int> <ord>    
## 1 length Aripo  Guanapo   0.942    39    29 large
```

The `cohens_d()` function calculates the effect size using the formula of the test. The `effsize` column contains the information that we want, in this case 0.94 .

We can know actually answer our question and see how many fish we really need to catch in the future:


```r
pwr.t.test(d = 0.94, power = 0.8, sig.level = 0.05,
           type = "two.sample", alternative = "two.sided")
```

```
## 
##      Two-sample t test power calculation 
## 
##               n = 18.77618
##               d = 0.94
##       sig.level = 0.05
##           power = 0.8
##     alternative = two.sided
## 
## NOTE: n is number in *each* group
```

From this we can see that any future experiments would really only need to use 19 fish for each group (18.77 from the first line rounded up, so no fish will be harmed during the experiment...) if we wanted to be confident of detecting the difference we observed in the previous study.

This approach can also be used when the pilot study showed a smaller effect size that wasn’t observed to be significant (indeed arguably, a pilot study shouldn’t really concern itself with significance but should only really be used as a way of assessing potential effect sizes which can then be used in a follow-up study).

## Exercise: one-sample
:::exercise
Performing a power analysis on a one-sample data set

Load in `data/tidy/CS6-onesample.csv` (this is the same data we looked at in the earlier practical containing information on fish lengths from the Guanapo river).

a. Assume this was a pilot study and analyse the data using a one-sample t-test to see if there is any evidence that the mean length of fish differs from 19 cm.
b. Use the results of this analysis to estimate the effect size.
c. Work out how big a sample size would be required to detect an effect this big with power 0.8 and significance 0.05.
d. How would the sample size change if we wanted 0.9 power and significance 0.01?

<details><summary>Answer</summary>

First, read in the data:


```r
fish_data <- read_csv("data/tidy/CS6-onesample.csv")
```

Let's run the one-sample t-test as we did before:


```r
fish_data %>% 
  t_test(length ~ 1,
         mu = 20,
         alternative = "two.sided")
```

```
## # A tibble: 1 × 7
##   .y.    group1 group2         n statistic    df       p
## * <chr>  <chr>  <chr>      <int>     <dbl> <dbl>   <dbl>
## 1 length 1      null model    29     -3.55    28 0.00139
```
There does appear to be a statistically significant result here; the mean length of the fish appears to be different from 20 cm.

Let's calculate the effect size using these data. This gives us the following output for the effect size in terms of the Cohen's d metric.


```r
fish_data %>% 
  cohens_d(length ~ 1, mu = 20)
```

```
## # A tibble: 1 × 6
##   .y.    group1 group2     effsize     n magnitude
## * <chr>  <chr>  <chr>        <dbl> <int> <ord>    
## 1 length 1      null model  -0.659    29 moderate
```

Our effect size is -0.66 which is a moderate effect size. This is pretty good and it means we might have been able to detect this effect with fewer samples.

:::Note
Although the effect size here is negative, it does not matter in terms of the power calculations whether it's negative or positive.
:::

So, let's do the power analysis to actually calculate the minimum sample size required:


```r
pwr.t.test(d = -0.6590669, sig.level = 0.05, power = 0.8,
           type = "one.sample")
```

```
## 
##      One-sample t test power calculation 
## 
##               n = 20.07483
##               d = 0.6590669
##       sig.level = 0.05
##           power = 0.8
##     alternative = two.sided
```

We would need 21 (you round up the n value) observations in our experimental protocol in order to be able to detect an effect size this big (small?) at a 5% significance level and 80% power. Let's see what would happen if we wanted to be even more stringent:


```r
pwr.t.test(d = -0.6590669, sig.level = 0.01, power = 0.9,
           type = "one.sample")
```

```
## 
##      One-sample t test power calculation 
## 
##               n = 37.62974
##               d = 0.6590669
##       sig.level = 0.01
##           power = 0.9
##     alternative = two.sided
```

Then we'd need 38 observations! We would need to do a bit more work if we wanted to work to this level of significance and power. Are such small differences in fish length biologically meaningful?

</details>
:::

## Exercise: two-sample paired
:::exercise
Power analysis on a paired two-sample data set

Load in `data/tidy/CS6-twopaired.csv` (again this is the same data that we used in an earlier practical and relates to cortisol levels measured on 20 participants in the morning and evening).

a. first carry out a power analysis to work out how big of an effect size this experiment should be able to detect at a power of 0.8 and significance level of 0.05. Don’t look at the data just yet!
b. Now calculate the actual observed effect size from the study.
c. If you were to repeat the study in the future, how many observations would be necessary to detect the observed effect with 80% power and significance level 0.01?

<details><summary>Answer</summary>

First, read in the data:


```r
cortisol <- read_csv("data/tidy/CS6-twopaired.csv")
```

We have a paired dataset with 20 pairs of observations, what sort of effect size could we detect at a significance level of 0.05 and power of 0.8?


```r
pwr.t.test(n = 20, sig.level = 0.05, power = 0.8,
           type = "paired")
```

```
## 
##      Paired t test power calculation 
## 
##               n = 20
##               d = 0.6604413
##       sig.level = 0.05
##           power = 0.8
##     alternative = two.sided
## 
## NOTE: n is number of *pairs*
```

Remember that we get effect size measured in Cohen's d metric. So here this experimental design would be able to detect a d value of 0.66 (2dp) which is a medium to large effect size.

Now let's look at the actual data and work out what the effect size actually is:


```r
cortisol %>% 
  cohens_d(cortisol ~ time, paired = TRUE)
```

```
## # A tibble: 1 × 7
##   .y.      group1  group2  effsize    n1    n2 magnitude
## * <chr>    <chr>   <chr>     <dbl> <int> <int> <ord>    
## 1 cortisol evening morning   -1.16    20    20 large
```


This value, -1.16, is a massive effect size. It's quite likely that we actually have more participants in this study than we actually need given such a large effect. Let calculate how many individuals we would actually need:


```r
pwr.t.test(d = -1.159019, sig.level = 0.01, power = 0.8,
           type = "paired")
```

```
## 
##      Paired t test power calculation 
## 
##               n = 12.10628
##               d = 1.159019
##       sig.level = 0.01
##           power = 0.8
##     alternative = two.sided
## 
## NOTE: n is number of *pairs*
```

So we would have only needed 13 pairs of participants in this study given the size of effect we were trying to detect.

</details>
:::

## Linear model power calculations
Thankfully the ideas we’re already covered in the t-test section should see us in good stead going forward and I can stop writing everything out in such excruciating detail (I do have other things to do you know).

For linear models we’ll just use `pwr.f2.test()` to do the power calculation and we won’t need a function for effect sizes (because it’s just based on R<sup>2</sup> and we should just be able to read that off the screen).

Read in `data/tidy/CS6-lobsters.csv`. This dataset was used in an earlier practical and describes the effect on lobster weight of three different food sources.

Type this in:


```r
# read in the data
lobsters <- read_csv("data/tidy/CS6-lobsters.csv")

# visualise the data
lobsters %>% 
  ggplot(aes(x = diet, y = weight)) +
  geom_boxplot()
```

<img src="cs6-practical-power_analysis_files/figure-html/unnamed-chunk-19-1.png" width="672" />

```r
# define the linear model
lm_lobster <- lm(weight ~ diet,
                 data = lobsters)

# perform ANOVA on model
anova(lm_lobster)
```

```
## Analysis of Variance Table
## 
## Response: weight
##           Df Sum Sq Mean Sq F value Pr(>F)
## diet       2 1567.2  783.61  1.6432 0.2263
## Residuals 15 7153.1  476.87
```
* the box plot shows us that there might well be some differences between groups
* the ANOVA analysis though shows that there isn’t sufficient evidence to support that claim given the insignificant p-value we observe.

So the question we can ask is:

> If there really is a difference between the different food sources as big as appears here, how big a sample would we need in order to be able to detect it statistically?

First let’s calculate the observed effect size from this study. For linear models the effect size is called Cohen’s f<sup>2</sup>. We can calculate it easily by using the R<sup>2</sup> value from the model fit and shoving it in the following formula:

\begin{equation}
f^2 = \frac{R^2}{1-R^2}
\end{equation}

We find R<sup>2</sup> from the anova on `lm_lobster` and then we can calculate Cohen's f^2:


```r
# get the effect size for ANOVA
R2 <- eta_squared(anova(lm_lobster))

R2
```

```
##      diet 
## 0.1797219
```

```r
# calculate Cohen's f2
R2 / (1 - R2)
```

```
##      diet 
## 0.2190987
```

So now we've got Cohen's f<sup>2</sup>. There’s one more thing that we need for the power calculation for a linear model (and this is where it gets a bit arbitrary); the degrees of freedom. We can either read those off the very bottom line of the `anova(lm_lobster)` output or we can extract them as follows:


```r
# get degrees of freedom
lm_lobster %>% 
  anova() %>% 
  anova_summary()
```

```
##   Effect DFn DFd     F     p p<.05  ges
## 1   diet   2  15 1.643 0.226       0.18
```

We have two different degrees of freedom: the **numerator degrees of freedom** (`DFn`) and the **denominator degrees of freedom** (`DFd`).Here the numerator degrees of freedom is 2. This is the number that we want. It is simply the number of parameters in the model minus 1. In this model there are three parameters for the three groups, so 3 - 1 = 2 (see the maths isn’t too bad). The other number is called the denominator degrees of freedom, which in this case is 15. This is actually the number we want the power analysis to calculate as it’s a proxy for the number of observations used in the model, and we’ll see how in a minute.

So, we now want to run a power analysis for this linear model, using the following information:

* power = 0.8
* significance = 0.05
* effect size = 0.219
* numerator DF = 2

We can feed this into the `pwr.f2.test()` function, where we use

* `u` to represent the numerator DF value
* `f2` to represent Cohen’s f<sup>2</sup> effect size value


```r
pwr.f2.test(u = 2, f2 = 0.219,
            sig.level = 0.05, power = 0.8)
```

```
## 
##      Multiple regression power calculation 
## 
##               u = 2
##               v = 44.12292
##              f2 = 0.219
##       sig.level = 0.05
##           power = 0.8
```

As before most of these numbers are just what you’ve put into the function yourself. The new number is `v`. This is the denominator degrees of freedom required for the analysis to have sufficient power. Thankfully this number is related to the number of observations that we should use in a straightforward manner:

$number\:of\:observations = u + v + 1$

So in our case we would ideally have 48 observations (45 + 2 + 1, remembering to round up) in our experiment.
There are two questions you might now ask (if you’re still following all of this that is – you’re quite possibly definitely in need of a coffee by now):

1. how many observations should go into each group?
    * ideally they should be equally distributed (so in this case 16 per group).
2. why is this so complicated, why isn’t there just a single function that just does this, and just tells me how many observations I need?
    * Very good question – I have no answer to that sorry – sometimes life is just hard.

The most challenging part for using power analyses for linear models is working out what the numerator degrees of freedom should be. The easiest way of thinking about it is to say that it’s the number of parameters in your model, excluding the intercept. If you look back at how we wrote out the linear model equations, then you should be able to see how many non-zero parameters would be expected. For some of the simple cases the table below will help you, but for complex linear models you will need to write out the linear model equation and count parameters (sorry!).

| Test| u|
|:- |:- |
|one-way ANOVA| no. of groups - 1 |
|simple linear regression| 1 |
|two-way ANOVA with interaction| no. of groups (`v1`) x no. of groups (`v2`) - 1 |
|two-way ANOVA without interaction| no. of groups (`v1`) + no. of groups (`v2`) - 2 |
|ANCOVA with interaction| 2 x no. of groups – 1 |
|ANCOVA without interaction| no. of groups |

## Exercise: Mussel muscles
:::exercise
Calculating effect size

The file `data/raw/CS6-shelllength.csv` contains information from a pilot study looking at whether the standardised length of the anterior adductor muscle scar in the mussel _Mytilus trossulus_ differs across five locations around the world (well it might be of interest to someone...).

Find the effect size from this study and perform a power calculation (at 0.8 and 0.05 significance level) to determine how many mussel muscles need to be recorded in order to be confident that an effect really exists.

<details><summary>Answer</summary>

Let's load in the data:


```r
musseldat <- read.csv("data/raw/CS6-shelllength.csv")
```

Let's just have a quick look at the data to see what we're dealing with:


```r
boxplot(length ~ location,
        data = musseldat)
```

<img src="cs6-practical-power_analysis_files/figure-html/unnamed-chunk-24-1.png" width="672" />

So we are effectively looking at a one-way ANOVA with five groups. This will be useful to know later.

Now we fit a linear model:


```r
# define the model
lm.mussel <- lm(length ~ location,
                data = musseldat)

# summarise the model
summary(lm.mussel)
```

```
## 
## Call:
## lm(formula = length ~ location, data = musseldat)
## 
## Residuals:
##       Min        1Q    Median        3Q       Max 
## -0.025400 -0.007956  0.000100  0.007000  0.031757 
## 
## Coefficients:
##                     Estimate Std. Error t value Pr(>|t|)    
## (Intercept)         0.078012   0.004454  17.517  < 2e-16 ***
## locationNewport    -0.003213   0.006298  -0.510  0.61331    
## locationPetersburg  0.025430   0.006519   3.901  0.00043 ***
## locationTillamook   0.002187   0.005975   0.366  0.71656    
## locationTvarminne   0.017687   0.006803   2.600  0.01370 *  
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.0126 on 34 degrees of freedom
## Multiple R-squared:  0.4559,	Adjusted R-squared:  0.3918 
## F-statistic: 7.121 on 4 and 34 DF,  p-value: 0.0002812
```
From this we get that the $R^2$ value is 0.4559 and we can use this to calculate Cohen's $f^2$ value using the formula in the notes:


```r
f2 <- 0.4559 / (1 - 0.4559)
f2
```

```
## [1] 0.8378974
```

Now, our model has 5 parameters (because we have 5 groups) and so the numerator degrees of freedom will be 4 $(5 - 1 = 4)$. This means that we can now carry our our power analysis:


```r
pwr.f2.test(u = 4, f2 = 0.8378974,
            sig.level = 0.05 , power = 0.8)
```

```
## 
##      Multiple regression power calculation 
## 
##               u = 4
##               v = 14.62182
##              f2 = 0.8378974
##       sig.level = 0.05
##           power = 0.8
```

This tells us that the denominator degrees of freedom should be 15 (14.62 rounded up), and this means that we would only need 20 observations in total across all five groups to detect this effect size (Remember: number of observations = numerator d.f. + denominator d.f. + 1)

</details>
:::

## Exercise: epilepsy
:::exercise
Power and effect

The file `/data/raw/CS6-epilepsy1.csv` contains information on the ages and rates of seizures of 236 patients undertaking a clinical trial.

a. Analyse the data using a linear model and calculate the effect size.
b. If there would be a relationship that large between age and seizure rate how big a study would be needed to observe the effect with a 90% power.


<details><summary>Answer</summary>

Let's load in the data:


```r
epilepsydat <- read.csv("data/raw/CS6-epilepsy1.csv")
```

Let's just have a quick look at the data to see what we're dealing with:


```r
plot(seizure ~ age,
     data = epilepsydat)
```

<img src="cs6-practical-power_analysis_files/figure-html/unnamed-chunk-29-1.png" width="672" />

So we are effectively looking at a simple linear regression here.

Now we fit a linear model:


```r
# define the model
lm.epilepsy <- lm(seizure ~ age,
                  data = epilepsydat)

# summarise the model
summary(lm.epilepsy)
```

```
## 
## Call:
## lm(formula = seizure ~ age, data = epilepsydat)
## 
## Residuals:
##      Min       1Q   Median       3Q      Max 
## -0.77513 -0.19585 -0.04333  0.22288  1.24168 
## 
## Coefficients:
##              Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  0.814935   0.124857   6.527 4.12e-10 ***
## age         -0.001990   0.004303  -0.463    0.644    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.413 on 234 degrees of freedom
## Multiple R-squared:  0.0009134,	Adjusted R-squared:  -0.003356 
## F-statistic: 0.2139 on 1 and 234 DF,  p-value: 0.6441
```

From this we get that the $R^2$ value is 0.0009134 (which is tiny!) and we can use this to calculate Cohen's $f^2$ value using the formula in the notes:


```r
f2 <- 0.0009134 / (1 - 0.0009134)
f2
```

```
## [1] 0.0009142351
```

This effect size is absolutely tiny. If we really wanted to design an experiment to pick up an effect size this small then we would expect that we'll need 1000s of participants.

Now, our model has 2 parameters (an intercept and a slope) and so the numerator degrees of freedom (`u`) will be 1 (2 - 1 = 1!). This means that we can now carry our our power analysis:


```r
pwr.f2.test(u = 1, f2 = 0.0009142351,
            sig.level = 0.05 , power = 0.9)
```

```
## 
##      Multiple regression power calculation 
## 
##               u = 1
##               v = 11493.05
##              f2 = 0.0009142351
##       sig.level = 0.05
##           power = 0.9
```

This tells us that the denominator degrees of freedom (`v`) should be 11494 (11493.05 rounded up), and this means that we would need 11496 participants to detect this effect size (Remember: number of observations = numerator d.f. (`u`) + denominator d.f. (`v`) + 1). 

</details>
:::

## Exercise: drug versus placebo
:::exercise
Study size

We wish to test the effectiveness of a new drug against a placebo. It is thought that the gender and the age of the patients may have an effect on their response.

a. Write down a linear model equation that might describe the relationship between these variables including all possible two-way interactions.
b. How big a study would we need to detect a medium effect size (according to Cohen) at a power of 90%, with significance level 0.05?


<details><summary>Answer</summary>

Here we have a system with a single response variable, and three categorical predictor variables. One of them is `gender`, with two possible levels (M, F). One of them is `treatment`, again with two possible levels (Drug, placebo) and one of them is continuous (`age`). A linear model with all possible two-way interactions would look something like this:

`response ~ treatment + gender + age + treatment:gender + treatment:age + age:gender`

In order to do a power calculation for this set up, we'll need four things:

1. the **effect size**. Here we're told it's a medium effect size according to Cohen so we can use his default values.
2. The **desired power**. Here we're told it's 90%
3. The **significance level** to work to. Again we're told this is going to be 0.05.
4. The **numerator degrees of freedom**. This is the tricky bit. We can do this by adding up the degrees of freedom for each term separately.

The effect size is easy to get:


```r
cohen.ES(test = "f2", size = "medium")
```

```
## 
##      Conventional effect size from Cohen (1982) 
## 
##            test = f2
##            size = medium
##     effect.size = 0.15
```

Alternatively, we could have looked this up online (which may give us different values, or values that are relevant to a specific discipline).

The numerator degrees of freedom is best calculated by working out the degrees of freedom of each of the six terms separately and then adding these up.

There are three simple ideas here that you need:

1. The degrees of freedom for a categorical variable is just the number of groups - 1
2. The degrees of freedom for a continuous variable is always 1
3. the degrees of freedom for any interaction is simple the product of the degrees of the main effects involved in the interaction.

So this means:

* The `df` for `gender` is 1 (2 groups - 1)
* The `df` for `treatment` is 1 (2 groups -1)
* The `df` for `age` is 1 (continuous predictor)
* The `df` for `gender:treatment` is 1 (1 x 1)
* The `df` for `gender:age` is 1 (1 x 1)
* The `df` for `age:treatment` is 1 (1 x 1)

Rather boring that all of them were 1 to be honest. Anyway, given that the denominator degrees of freedom is just the sum of all of these, we can see that $u = 6$.

We now have all of the information to carry out the power analysis:


```r
pwr.f2.test(u = 6, f2 = 0.15,
            sig.level = 0.05, power = 0.9)
```

```
## 
##      Multiple regression power calculation 
## 
##               u = 6
##               v = 115.5826
##              f2 = 0.15
##       sig.level = 0.05
##           power = 0.9
```

We get a denominator df of 116, which means that we would need at least 123 participants in our study (Remember: number of observations = numerator d.f. (u) + denominator d.f. (v) + 1). Given that we have four unique combinations of `gender` and `treatment`, it would be practically sensible to round this up to 124 participants so that we could have an equal number (31) in each combination of gender and treatment. It would also be sensible to aim for a similar distribution of age ranges in each group as well.

</details>
:::

## Key points

:::keypoints
- Power is the capacity of a test to detect significant results and is affected by
    1. the effect size
    2. sample size
    3. the significance level
- Power analysis optimises the trade-off between power, significance level and the desired effect size that we would like to detect
- Point 3
:::
