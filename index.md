![image](https://user-images.githubusercontent.com/91272449/144743574-12760d55-d99e-4016-8858-439c8db092b2.png)


# Data transformation: Why and How it's Done!
Created by Lubin Grosbuis - last updated by Lubin Grosbuis on 10/12/2021
## Tutorial Aims
### 1. To develop an appreciation for the uses of data transformations
### 2. To develop the ability to identify situations in which data transformations are necessary using preliminary plots, statistical tests, and a bit of common sense.
### 3. To get familiar with statistical vocabulary used to describe data and distributions.
### 4. To learn to make appropriate data transformation decisions to address various issues.

## Tutorial Structure

#### <a href="#section1"> Part 1: Building a Model and Checking ANOVA Assumptions</a>

#### <a href="#section2"> Part 2: Data transformations to Meet ANOVA Assumptions</a>

#### <a href="#section3"> Part 3: Back to our Model: Using Transformed Data</a>

Introduction
---------------------------
Welcome to this tutorial on pre-analysis data transformations! In data science, we often want to build models of our data and carry out statistical tests to evaluate the effect of one or more explanatory variables on our dependent variable. However, models and statistical tests often make assumptions about our data, particularly regarding their distributions, and it is our job to ensure that these assumption are met. When these assumptions are violated, it is often possible to apply transformations to our data so that the values contain the same information, but satisfy model and statistical test assumptions. We also sometimes have to work with data communicating the same information in different units, which makes comparing data unfeasible, like trying to compare apples and pears. Nevertheless, solutions exist, so that these can become comparable.

In this tutorial, we will be learning how to identify issues surrounding violation of ANOVA assumptions and data reported in inconsistent units. We will then be looking at the different options available to us to transform our data, resolving the issues we identified. By the end of this tutorial, you will be equipped with tools useful to transform your data to suit your model and analysis, hopefully saving you many hours of frustration, staring blankly at your screen and sleepless nights trying to figure out how to transform your data.

This tutorial is NOT for beginners; We assume that you already know your way around the basic features of R, the coding language used in this tutorial, and that you are able to manipulate and visualize your data efficiently. If you are not at this stage yet, or need a refresher, you can revise these skills by following the following Coding Club tutorials;

 . Introduction to R - https://ourcodingclub.github.io/tutorials/intro-to-r
 
 . Basic data Manipulation - https://ourcodingclub.github.io/tutorials/data-manip-intro
 
 . Data Visualization - https://ourcodingclub.github.io/tutorials/datavis

This tutorial is aimed at intermediate coders with basic statistical knowledge, who are being introduced to general linear model development and analysis. This tutorial should in fact be complementary to the material covered in the Coding Club's tutorials "from distributions to linear models" (https://ourcodingclub.github.io/tutorials/modelling) and "Linear Mixed models" (https://ourcodingclub.github.io/tutorials/mixed-models).

You can get all of the resources for this tutorial from <a href="https://github.com/EdDataScienceEES/tutorial-lubin-g" target="_blank">this GitHub repository</a>. Clone and download the repo as a zip file, then unzip it.

<a name="section1"></a>

## Part 1: Building a Model and Checking ANOVA Assumptions

Right, let's get started. Start by opening `RStudio` and open a new script by clicking on `File/New File/R Script` on the top left of your screen. It's also good practice to put a title, date, your name and contact details at the top of your script. Also make sure you've set you working directory to the folder where all the files for this tutorial are stored and that you've installed and loaded the packages required for this tutorial.

```
# Pre-Analysis Data transformation: Why and How it's Done!
# Date: 10/12/2021
# Name: Lubin Grosbuis
# Contact: s1810379@ed.ac.uk

# Set the working directory
setwd("your_filepath")

# install.packages("ggplot2") # remove the hashtag and run this line if you haven't yet installed the ggplot2 package.

library("ggplot2") # load ggplot2 to make tutorial graphs
```
### Pro tip 1: Personalize your Rstudio theme
Boost your confidence by switching your background to black and text color to flashy green; This'll make you feel like Neo from _The Matrix_ and give people the impression you are some sort of coding wizard. What's more, it's much easier on the eyes when you're coding in the dark at 3am (don't tell me it's never happened-we've all been there). To do this, go to `Tools` in the Menu bar, and select `Global options/appearance` and set the editor theme to `Gob`.

Load and read the file `ant_data1.csv`. Have a look at what the data consists of using the `head`, `view` and `summary` functions. The data set is from a study investigating foraging efficiency of Red Wood ant colonies under different tree species. As you can see, this data set has two variables; "Food and "Tree".  The "tree" column  is of the  character type, and has three possible category entries; Rowan, Oak and Sycamore. The "food"" column, contains continuous numeric values, which represent food collection rates in units of dry biomass (mg)/per ant leaving a given tree, measured over a 30 minute period. 


```
ant_data <- read.csv("ant_data1.csv") # import and read the ant data set 
head(ant_data) # returns the first entries of all columns in the data set
summary(ant_data) # summary values for the ant data set
View(ant_data) # displays the whole data set as a table
```

Suppose we want to know if ants are more efficient at collecting food on certain trees. Perhaps some tree species support more prey, or might have a rougher bark or a more complex canopy structure which complicates an ants foraging task. We can plot the data as a boxplot to see if we notice any pattern. 

```
(Food_tree_plot <- ggplot(ant_data, aes(x = Tree, y = Food)) +
  geom_boxplot()) 

# adding a more informative y axis label and a cleaner background
Food_tree_plot + ylab("Foraging efficiency (mg of biomass/ant)") + theme_classic() 

ggsave(Food_tree_plot.png)  #saving the box plot as a png file
```
As you can see from the box plot, it seems the food collection rate of ants is highest in oak trees, lowest in Rowan, and falls somewhere in between these two extremes in Sycamore trees. However, there is a lot of overlap in these results, and it is good to investigate this relationship in an analysis of variance (ANOVA).

We can do this by creating a sinmple general linear model (an ANOVA assuming a linear relationship between independent and dependent variable)

```tree_food_mod1 <- lm(Food ~ Tree, data = ant_data) # modelling ant food collection as a response to tree species
summary(tree_food_mod1) # obtaining the ANOVA results for our linear model
```

Woah! Hold your horses! Before we have a look at our model results and draw any conclusions, we should make sure that our model is valid...That is, does it satisfy the assumptions of anova? Let's go over these assumptions and find out.

## ANOVA assumptions
ANOVA makes 3 key assumptions on the data.

1. __Independence of observations__: Each sample/data point is independent to every other, There is no correlation between them. Since we don't know how the data was collected, we're just going to have to trust the samplers on this one.

2. __Normality of Residuals__: We assume that the residual variance of our response data around the model's predicted values is normally distributed around a mean error value. In case you are unsure- "residuals"" refers to the difference between our raw data values and the values predicted by our model for a given value of our explanatory variable (x).

3. __Homoscedasticity of Residuals__: We assume equal variance of our response data around the model predictions for all values or categories of our explanatory variable. In other words, the variance of the error, or residuals is constant. The opposite condition, whereby residual variance is not homogenous throughout all model values would be termed "heteroscedasticity".


## Verifying Assumptions

Now that we've gone through the assumptions of ANOVA, it's time to check whether our model meets these assumptions.

### Step 1: Histogram of the response variable

A first step we can do to evaluate the satisfaction of ANOVA assumptions is having a look at the distribution of our response variable in a histogram. What we are looking for is a normal (also known as a Gaussian) distribution. In otherwords, we are looking for a bell-shaped distribution, where values increase in frequency around a central, mean value. You can copy and paste the code below to generate a frequency distribution histogram for the data.

```
hist(ant_data$Food) # making a basic histogram of the distribution of the variable "Food"
# Adding a more informative title
hist(ant_data$Food,breaks=10,col="grey",xlab="Foraging efficiency (mg of biomass/ant) ",main="")
ggsave("ant_food_histogram.png") # saving the histogram in the working directory as a png image
```
As you can see, our data doesn't look very normal; Low values are very fequent, and high values are much less frequent; We can describe our data as __right skewed__ or __right tailed__. We are definitely not seeing anything that would fit well under a "bell-shaped" curve. Not good. We need to investigate this further.

You might be wondering why we are looking at the distribution of the response variable. Afterall, I never said that ANOVA assumes normality of the raw data - and in fact, it does not. What is important is that the residuals of the raw data are normally distributed. This is a VERY common pitfall which many researchers fall into! Nevertheless, non-normality of residuals often coincides with non-normal raw data, which is why many people look no further than this in testing ANOVA assumptions. However, it is absolutely possible to have non-normal raw data, but normally distributed residuals with equal variance, and vice-versa. This is why as rigorous data scientists, we will go further in our investigation of assumptions.

### Step 2: Diagnostic Plots

A more insightful way of determining whether our model meets ANOVA assumptions is by using diagnostic plots. You can call the diagnostic plots simply by using the `plot()` function and entering your model name in the brackets. Run the code below, and hit `return` in the console to visualize each of the 4 plots in turn.

```
plot(tree_food_mod1)
```
Let's consider each one of these in turn, see what information they communicate and how they can help us determine if our model meets ANOVA assumptions.

__a) QQ Plot__

The QQ (quantile-quantile) plot is used to evaluate the distribution of the residuals of our response varaible with respect to the models' predictions. It plots the quantiles of our residuals against the theoretical quantiles of a normal distribution. In case you don't know or need a reminder, a quantile is simply an interval within a distribution in which a given proportion of the values will occur. If our residuals' distribution is close to normal, then its quantiles (y axis) should coincide well with the theoretical quantiles of a normal distribution (x axis), and should follow a near linear relationship.

Therefore we can assess how normal our residuals' distribution is by visually checking how well the points in the plot align with the straight dotted line y = x. If the points follow a linear trend and are well aligned with the dotted line, then we can conclude that the residuals are normally distributed.

![QQplot_mod1](https://user-images.githubusercontent.com/91272449/145222002-402208bd-8ae8-4b00-afdd-c6705868a2cc.png)

As we can see from the above plot, our residuals show some departure from a normal distribution; Around the center of the plot (0), points are slightly below the dotted line, whereas on the upper (right) side of the plot, points deviate above the straight line, which is characteristic of a right-skewed distribution. Left-skewed residuals would show a similar pattern on the lower, left hand side of the plot, with points deviating below the straight line. We can also describe a tailed distribution like the one we have here as displaying "kurtosis".

We can conclude that our data's residuals likely violate ANOVA's assumption of normally distributed residuals, so we will need to correct for this later!


__b) Residuals vs Fitted Plot__

The residuals vs fitted plot, as it name indicates, plots our dependent variable's residuals against fitted model values. This plot gives us 2 important pieces of information. 

Firstly, it can allow us to determine if the relationship we are modelling is linear. If we are dealing with a linear relationship, the residuals should be dispersed randomly around a mean residual value of approximately 0 for all fitted values, and so the red trend line should align well with the dotted line y = 0. Note that ANOVA does not assume linearity, but the linear model we are using to model the ant foraging efficiency does!

Secondly, the plot shows the spread of residual values for a given fitted value, allowing us to evaluate the assumption oh homoscedasticity. If residuals display homoscedasticity, then they should show an equal level of variance for all model values.

![Residvsfitted_mod1](https://user-images.githubusercontent.com/91272449/145222056-b76bea2c-c82c-47fd-88c7-ed0a83b58e04.png)

As you can see in the above plot, the red line shows a decreasing trend suggesting that we are dealing with a non linear relationship. Moreover, it seems that variance is not constant across model values...in other words, it looks like our residuals are heteroscedastic. We can see an increase in residual variance with increasing model values. Rowan (left cluster of dots) shows much less residual variance than oak (right side of plot). R also gives us row numbers (31, 38 and 40) for data points which show strangely large residual variance, which could potentially be outliers which we should remove.

Therefore, we have now found that the assumption of homoscedasticity is ALSO violated.

__c) Scale-Location Plot__

The Scale-Location Plot is very similar to the residuals vs fitted plot, with the exception that instead of plotting residual values as such, it plots the square root of the absolute residual values against fitted values. This can make it easier to verify the assumption of homoscedasticity, as the y axis scale now starts at 0, and all residuals are standardized with a standard deviation of 1. 

![scaleloc_mod1](https://user-images.githubusercontent.com/91272449/145222043-f4ca14f6-5c62-4556-ad2d-7ca6f92e9657.png)

The red line shows an increasing trend which reflects an increase in the magnitude of residual values with increasing fitted values. We can also once again see that for lower fitted values, residuals are clustered closer to 0 than for larger fitted values, where the residuals are more spread out.

__d) Residuals vs Leverage Plot__

The residuals vs leverage plots shows standardized residual values against leverage. Leverage quantifies the extent to which model predictions would change if a particular entry were removed from the model. This allows us to identify potential outliers in our data, for which we should investigate their validity (e.g was the sampling performed adequately to obtain this value). If reoving any values would significantly change model predictions, these would appear beyond a a dotted red line, representing a critical leverage value known as "Cook's distance"

![Residvslev_mod1](https://user-images.githubusercontent.com/91272449/145222076-17eda7c9-e774-432f-8405-df9e84f896ba.png)

As we can see from our plot, it seems we have no significant outliers in our data.

### Pro tip 2: If in Doubt, Have a Coffee
Coding and Coffee: now that's a good combo! According to this healthline article https://www.healthline.com/nutrition/what-is-caffeine , the caffeine molecule in coffee blocks receptors of adenosine in your brain. Adenosine is a neurotransmitter which causes you to relax and feel tired. Blocking adenosine receptors has the effect of keeping you focused and alert (and aroused according to the healthline article-but this won't help you code any better!) Therefore, if you want to code better, for longer, coffee is the way to go! Go have yourself an espresso and return in 5 minutes.


### Step 3: Statistical Tests

Lastly, we can test whether our model satisfies ANOVA assumption using statistical tests, which has the advantage of returning a probability, allowing us to more confidently determine if the deviations we see from ANOVA assumptions are actually large enough to significantly affect our conclusions.

a) Testing Normality of Residuals' distribution: The Shapiro-Wilk Test

We can test the assumption that residuals are normally distributed using a Shapiro-Wilk test. This test checks whether the null hypothesis that the data's distribution is not significantly different to a normal distribution is true. Paste and run the code below in your own script to run the test. Notice we first have to extract model residuals with the `resid()` function before we can run the test.

```
# Extracting model residuals
tree_food_mod1resids <- resid(tree_food_mod1) # creating an object containing model residual values

# Running the Shapiro-Wilk test
shapiro.test(tree_food_mod1resids) # running a Shapiro-Wilk test on model residuals
```
Have a look at the test results in the console. The p-value is much smaller than an arbitrary significance threshold value of α = 0.05, which means our residual distribution shows significant departure from the condition of normality, and it is unlikely to be due to chance!

From this, we can confirm that our model violates ANOVA's assumption that residuals are normally distributed!

b) Testing Residual Homoscedasticity: The Bartlett Test

Similarly to the Shapiro-Wilk test above, the Bartlett test checks whether the null hypothesis that residual variance is homoscedastic, i.e constant across fitted model values/categories, is true.

We can test our model by running the code below. 

```
bartlett.test(Food~Tree,data=ant_data) # running a Bartlett test on our model
```
Once again, have a look at the results in the console. The p-value is again much lower than a significance threshold of 0.05, so we can conclude that the null hypothesis of homoscedasticity of residuals does not hold, and that our model also violates this assumption of ANOVA.


<a name="section2"></a>

## Part 2: Data transformations to Meet ANOVA Assumptions

Now that we have identified which assumptions of ANOVA our model violates, it's time to solve these by transforming our data.

### Step 1: How Can We Transform our Data

"Transforming" our data means applying a mathematical function to our response variable (in our example in Part 1, the variable "Food" describing ant food collection rate) so that it fits the assumptions of our model, while transmitting the same information.

Let's go over the different types of data transformations we can put our data through...The end goal is to satisfy the assumptions of normality and homoscedasticity of residuals.

__1. Logarithmic Transformations__

Logarithmic, or log transformations are the most common and involve converting each value of the response variable to its logarithm, in the base of your choice. The most frequently used are log10 and the natural logarithm log(e), commonly written as ln.

Log transformations have the effect of stretching out the left side of a distribution (inflating the difference between lower values) and compressing the right hand side (higher values). This is therefore a useful transformation to squash, or reduce the right skew, or tail, of a distribution at higher values and get closer to a normal distribution. Moreover, this may also deal with heteroscedasticity, when categories with higher means also have greater residual variance (as is the case in our model of ant foraging efficiency under different trees)

Let's then log transform the "Food"" variable representing ant food collection rates in the ant_data data set that we loaded previously...

```
ant_data$log_food <- log(ant_data$Food) # creating a new variable, "log_food"
```
Note that the `log()` function transforms your data using the natural logarithm, ln. If you want to use base 10, specify this using the function `log10()` instead.

### An Important Consideration
A simple log transformation as described above will not work if the transformed value is 0, as we cannot get the logarithm of 0. This will return an "NA", which is not dramatic if you only have a few zeroes in your data and your sample size is large. This becomes an issue when we are working with population/count-abundance data, which is often full of zeroes.

This issue can be overcome by adding a constant to your variable before log transformation, such as 1. In this way, all zero values  will become 1, and the transformation becomes __log(x+1)__.

__2. Square Root transformation__

A Square Root transformation involves taking the square root of the response variable. It essentially has the same effect on a distribution as a log transformation. However, a log transformation compresses larger values more aggressively than a square root transformation. It is up to you to decide which transformation to use depending on how extreme the kurtosis of your residuals; the right skew, is. We can apply this transformation to the "Food" variable in ant_data using the `sqrt()` function.

Note: We cannot get the square root of a negative value, so if your variable contains negative values, you may find it easier to use log transformation. If you insist on performing a square root transformation, then add a constant value to your variable so that all values are positive before square rooting.

```
ant_data$sqrt_food <- sqrt(ant_data$Food) # creating a new variable, "sqrt_food"
```

We can look at how log and square root transformations affect the distribution of the "Food" variable by plotting a histogram of the two new variables. Paste and run the code below and look at the plots...which transformation seems more appropriate to you?

```
hist(ant_data$log_food,breaks=10,col="grey") # frequency distribution histogram for "log_food"

hist(ant_data$sqrt_food,breaks=10,col="grey") # frequency distribution histogram for "sqrt_food"
```
The right skew in the distribution of our original "Food" variable was fairly strong, and it seems the less extreme square root transformation was insufficient to get rid of the right tail in the distribution. We should probably continue our analysis with the log transformed variable as it looks more normally distributed, although we do not see a clear peak in frequency around the mean, at the center of our distribution. Nevertheless, ANOVA is relatively robust to such small deviations from normality, so we shouldn't worry about this too much.

__3. Squaring__

So far, we have discussed transformations which can help us tackle the issue of right skewed distributions, as this is relevant to the data we have at hand and a very common occurence.- But what if we encounter the opposite issue? A left skewed distribution; with few low values and many high values. Transforming the data by squaring the response variable can resolve this. As you might expect, this has the exact opposite effect of square root transformation, and so can remove left-tails in a distribution.

This transformation is not relevant to the issues we have encountered in our analysis. Nevertheless, below is a line of code for this transformation on a hypothetical data set for future reference.

```
your_data$squared_variable <- (your_data$original_variable)^2
```
Note: squaring a negative and postive value of the same magnitude yields the same result. Therefore, make sure that when applying this transformation, your variable only contains values equal to or greater than 0 to avoid confusion!

__4. Arcsine square root and Logit Transformation__

We may sometimes encounter data which follows a binomial distribution; This means instead of having a single central frequency peak in a distribution, we have two different peaks. Such data very often violates the assumptions of normality and homoscedasticity of residual variance. This is often the case of percentage and proportion data. A binomial distributions' variance is a function of the mean, and reaches a maximum value at a proportion of 0.5, and declines towards zero at proportions of zero and one.

We can linearize a binomial relationship and stabilize residual variance to satisfy ANOVA assumptions using the Arcsine square root and logit transformations. These transformations have the effect of stretching out proportions that are close to 0 and 1 and compressing proportions close to 0.5. The difference between these two transformations is that logit transformation "stretches"" and "compresses" proportions more agressively than Arcsine square root. It is up to you to decide which one is most suitable depending on how extreme the two peaks in your binomial distribution appear. You can always try both transformations and evaluate which is most suitable by comparing the distributions of your transformed variable, using diagnostic plots and statistical tests, as we did before.

Below is some code showing how you would transform your original response data `original_variable` 
using Arcsine square root and logit transformation to create a transformed variable, `new_variable` within the hypothetical dataset, `your_data`.

__Arcsine transformation__

For the Arcsine transformation, apply arcsine by using the `asin()` function call, and nest your response data within its brackets, applying the squareroot function `sqrt` to it.

```
your_data$new_variable <- asin(sqrt(your_data$original_variable))
```
Note: Arcsine cannot deal with values lower than 0 or greater than 1, so make sure the values of your variable fit within this scale! If your variable is expressed as a percentage, simply divide values by 100 to obtain proportions instead, and work with this.


__Logit transformation__

For the logit transformation, we can use both proportions or percents, simply by adapting the function slightly. The transformation involves taking the log of the proportion (or percentage) divided by one minus the proportion (or 100 minus the percentage). The code below shows how the transformation is done for data expressed as a proportion or percentage.

```
your_data$new_variable = log (p / (1 - p)) # where p is the original variable (your_data$original_variable) expressed as a proportion


your_data$new_variable = log (p / (100 - p)) # where p is the original variable (your_data$original_variable) expressed as a percentage

```
### Pro tip 3: MUSIC
Having trouble staying focused? Ambulance sirens outside, flatmate on a call with a friend, neighbour hoovering the room right above your head...are all these sounds distracting you? Consider drowning out all these noises by playing some music nice and loud. Avoid playing very lyric heavy/melodic songs as these may distract you even more. The author's favorites are groovy 80s love songs and portuguese rap (if you don't speak the language, you won't understand it and won't be able to sing it so it probably won't distract you). 


### Back transforming data

We have now seen various ways in which our data can be transformed according to the issue at hand.
Transformations can allow us to meet our model assumptions and run our analysis and obtain model predictions. However, we have to remember that these model predictions will have little relevance to the units and scale of our raw data. 

For instance, if we investigated the effect of soil Nitrate availability on a plant's growth rate in cm/day, but decided to log transform the growth rate to satisfy ANOVA assumptions, then our model's estimate of the increase in growth rate for a unit increase in Nitrate availability would be expressed as log(growth rate) which cannot directly be related to cm. 

Therefore, once we obtain model predictions, we must reverse the transformation to obtain meaningful results in the original response unit. Back transforming model predictions is fairly intuitive. Nevertheless, below are the operations which must be applied to back transform model results, for the different transformations described.

__1. Logarithmic transformations__
exponentiate the value using the same base used in your log transformation, for instance;

log10(x) is reversed by 10^x

log(x) , or ln(x) is reversed by e^x

Remember that for log(x+1), we must also subtract 1 from the antilog, 10^x, for a complete back transformation.

__2. Square Root transformation__
Take the square of your model prediction to back transform it to original units ans scale.

__3. Squaring__
Take the square root of your model prediction to back transform it to original units ans scale.

__4. Arcsine square root and logit transformation__

To reverse logit --> (exp^x)/(1 + exp^x)

To reverse arcsine --> sin^2(x)


Note that the transformations mentionned above are by no means an exhaustive list, but just the most common approaches to resolving issues related to violation of model assumptions. Feel free to explore other options and make other transformations as you evolve in your data science journey. As you learn about more about modelling, you may also find that you can run statistical analysis on more sophisticated models which do not have as many constraining assumptions.
 

### Pro tip 4: Take a Break
If you are like me, you have the attention span of a goldfish. After 40 minutes of coding, you start getting more and more distracted -checking social media, opening youtube, wandering through random wikipedia pages...At this point don't stay in front of your computer. Take a 15 minute break - call someone, go outside and come back with a fresh mind, ready for another 40 minutes of productive work. 


<a name="section3"></a>

## 3. Part 3: Back to our Model: Using Transformed Data

Alright, we've seen all the possibilities we have to transform data depending on the issues we face, and the pitfalls we should avoid falling into. Congratulations on getting up to here! We're nearly finished, so please bear with me!

### Step 1: making a new model using transformed data

Let's get back to modelling the effect of tree species on ant foraging efficiency, which we were trying to investigate in part 1). Remember that in part 2), we transformed the "Food" column representing ant food collection rates using `log(x)` and `sqrt(x)`, to create two new variables. Having looked at the frequency distribution histograms for these new variables, we concluded that the "log_food" variable arising from the `log(x)` transformation was more likely to satisfy ANOVA assumptions.

Let's then create a new model, of the response of log transformed food collection rates to different tree species. Paste and run the code below to create this new model, which we will call `tree_logfood_transmod`.

```
tree_logfood_transmod <- lm(log_food ~ Tree, data = ant_data) # modelling log(ant food collection) as a response to tree species
```
### Step 2: Checking if ANOVA assumptions are met

We can check whether our new model now satisfies the assumptions of ANOVA. As we learnt to do in Part 1), we can generate diagnostic plots using the `plot()` function to check for residual normality and homoscedasticity.

```
plot(tree_logfood_transmod)
```
Great! It looks like our new model satisfies the assumption of homoscedasticity of residuals. We can see this as the red line of the Scale-Location plot is flat and straight, and the spread of the residuals is the same across increasing fitted values.

It also seems from the Q-Q plot (shown below) that our new model satisfies the assumption of normally distributed residuals, as most points fall on the straight line, and the right skew is gone. However, we do see that there is departure from normality at the tails of the distribution, but at least this departure is symmentrical on both sides. This departure at the tails tells us that the tails are shorter than we would expect in a normal distribution.

Finally, we can confirm that our model satisfies the assumptions of ANOVA by running the Shapiro-Wilk and Bartlett tests on our new model.

```
# Extracting our new model's residuals
tree_logfood_transmod_resids <- resid(tree_logfood_transmod) # creating an object containing our new model's residual 

shapiro.test(tree_logfood_transmod_resids) # running a Shapiro-Wilk test on the extracted residuals

bartlett.test(log_food~Tree,data=ant_data) # running a Bartlett test for our model
```
Fantastic! P-values for both tests are greater than than the significance threshold α = 0.05, which means there is no significant departure from normality and homoscedasticity in our model's residuals. This means all ANOVA assumptions are satisfied and our log(x) transformation has fulfilled its mission!

### Step 3: Extracting ANOVA results and back transformation

Finally, now that all model assumptions are met, we're ready to have a look at the ANOVA results for our model. Paste and run the code below in your script to get ANOVA results in the console.

```
summary(tree_logfood_transmod)
```
Let's look at the results together. The p-value at the bottom is 0.001255, which means that our model is significant at α = 0.05. In other words, the tree species that ants forage on __significantly__ affects their food collection rate. 

The adjusted R-squared value is approximately 0.14. This means that the different tree species explain 13% of the variance in food collection rates we observe between samples.

Now looking further up we have our coefficients, which are the three tree species investigated. Notice that we can't see Oak mentionned anywhere. This is because Oak is actually represented by the intercept row, and the two other tree species have their estimates expressed relative to oak. 

The Pr(>|t|) column shows individual p-values for each category (in our case, tree species). We can see that anr foraging efficiency in oak and Rowan differs significantly, but that the foraging efficiency of ant on Sycamore is not significantly different to the other species.

The last thing we have to do is to back-transform our coefficient estimates to relate these to our original units of foraging efficiency; mg of dry biomass/ant. To do so, we need to exponentiate our coefficient estimates. This means raising e to the power of the model estimate. This can be done with the `exp()` function.

For oak, which is the intercept, the models' estimate is `exp(3.0253)`, therefore 20.6 mg of biomass per ant leaving the tree.

The other estimates are expressed relative to the intercept, oak. Therefore, we must first individually subtract the Rowan estimates and Sycamore estimates from the intercept value of oak before we exponentiate these to get the model estimates in the relevant units.

For Rowan, our model predicts `exp(3.0253-0.7376)`, therefore 9.6 mg of dry biomass per ant.

For Sycamore, our model predicts `exp(3.0253-0.3670), therefore 14 mg of dry biomass per ant.

Well done on making it up to here! This is the end of this tutorial.

In this tutorial we learned:

##### - How to build a simple general linear model and what the assumptions of ANOVA are
##### - How to check if ANOVA assumptions are met using histograms, diagnostic plots and statistical tests
##### - How to use the right ststistical terminology to describe histograms, diagnostic plots, and distributions
##### - How data can be transformed in different situations to meet general linear model assumptions
##### - How we can back-transform data to interpret model estimates and ANOVA results obtained from transformed data.

### If you want to learn more about the topics covered in this tutorial and modelling, follow the links below;
__data transformations__: https://en.wikipedia.org/wiki/Data_transformation_(statistics)
__interpreting diagnostic plots__: https://rpubs.com/Amrabdelhamed611/669768 and https://drewtyre.rbind.io/post/scale-location/
__interpreting general linear model results__: https://vulstats.ucsd.edu/pdf/Gelman.ch-04.regression-transformations.pdf and http://www.sthda.com/english/articles/40-regression-analysis/163-regression-with-categorical-variables-dummy-coding-essentials-in-r/
__Articles on confusing normality of raw data and residuals__: https://www.introspective-mode.org/assumptions-residuals-variables/ and https://pubs.er.usgs.gov/publication/5224239

#### If you have any questions about completing this tutorial, please contact me at s1810379@ed.ac.uk

