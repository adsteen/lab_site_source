---
title: 'Linear models with a sparse predictor'
author: admin
date: '2020-01-22'
slug: drew-learns-statistics-linear-models-with-a-sparse-predictor
categories: []
tags: []
subtitle: ''
summary: ''
authors: []
lastmod: '2020-01-22T14:02:09-05:00'
featured: no
image:
  caption: ''
  focal_point: ''
  preview_only: no
projects: []
header-includes:
   - \usepackage{amsmath}
   - \usepackage{amssymb}
   - \usepackage{amsfonts}
output: md_document
draft: true
---
  
{{ if and (not .Params.disable_mathjax) (or (in (string .Content) "\\") (in (string .Content) "$")) }}
<script src="{{ "/js/math-code.js" | relURL }}"></script>
<script async src="{{ .Site.Params.MathJaxCDN | default "//cdnjs.cloudflare.com/ajax/libs" }}/mathjax/{{ .Site.Params.MathJaxVersion | default "2.7.5" }}/MathJax.js?config=TeX-MML-AM_CHTML"></script>
{{ end }}

I want to buy a sweet ride<sup>[1]</sup>
=============================

I spent some time over the weekend thinking about statistical models of
car prices, because I’m a Cool Guy and I do this kind of thing
for fun, and also to procrastinate from real work.

Specifically, I’m interested in how to predict the sales price of
unusual/classic cars on a well-known car auction site, which I won’t
name because doing this sort of thing may not 100% fit within their
terms of service. Obviously, the mileage of a car is a strong
determinant of its sales price: lower mileage cars are more expensive
than high-mileage cars. We could describe this with a simple linear
model([2]):

    price = *β*<sub>0</sub> + *β*<sub>1</sub> ⋅ mileage + *ϵ*

But the cars I’m interested in have pretty weird histories: they often
have had engines swapped, or odometers rolled over, or some other funny
business such that their actual mileage is not known. Buyers hate this.
**I’d really like to know how big of a discount buyers demand for not
knowing a car’s mileage.** The problem is that formally speaking, we
can’t write a single model for this, because mileage doesn’t have a
value when mileage is unknown. So I’m looking for a way to encode the
model

$$
\\textrm{price} = \\beta\_0 + \\left\\{
  \\begin{array}{ll} 
    \\beta\_1 \\cdot \\textrm{mileage} & : \\textrm{mileage known}\\\\
    \\beta\_{1'} \\cdot \\textrm{mileage} & : \\textrm{mileage unknown}  
    \\end{array} \\right\\} + \\epsilon
$$

\[@Jeff\_Jetton\](<a href="https://twitter.com/Jeff_Jetton" class="uri">https://twitter.com/Jeff_Jetton</a>),
\[@pbulsink\](<a href="https://twitter.com/pbulsink" class="uri">https://twitter.com/pbulsink</a>),
and
\[@Hao\_and\_Y\](<a href="https://twitter.com/Hao_and_Y" class="uri">https://twitter.com/Hao_and_Y</a>)
kindly [led
me](https://twitter.com/drdrewsteen/status/1219267790563151884) via
Twitter to a solution that seems mathematically correct and is easy to
code, which I want to archive here.

They converged on the idea of setting mileage to 0 whenever it is
missing, and then creating a second variable for ‘mileage missing’, with
its own coefficient, so the linear model would look like this:

price = *β*<sub>0</sub> + *β*<sub>1</sub> ⋅ mileage + *β*<sub>1′</sub> ⋅ mileage unknown + *ϵ*
 Cool, I can write an R formula that encodes that model!

    mod <- lm(mileage + mileage.unknown, data = my_df)

In this model, *b**e**t**a*<sub>0</sub> represents the ‘base’ price of a
car (before mileage, and any other potential variables I might want to
include in my model, are considered). *β*<sub>1</sub> is the rate at
which the car loses value per mile, and *β*<sub>1′</sub> is the discount
buyers demand for a car of unknown mileage.

Does it work?
-------------

Just to be sure, I’ll create some simulated data and see whether I can
recover reasonable values. I’ll set *β*<sub>0</sub> to $10,000,
*β*<sub>1</sub> to -$0.005, and *β*<sub>1′</sub> to -$2,500.

    library(tidyverse)

    set.seed(944)
    mileage <- runif(n = 100, min = 0, max = 1e6) # Random distribution of mileages
    mileage.penalty <- -0.005 
    mileage.known <- sample(c(TRUE, FALSE), size = 100, replace = TRUE) # Random cars have "unknown" mileage
    mileage[!mileage.known] <- 0 # Those with unknown mileage are set to 0
    base.price <- 1e4
    TMU.penalty <- -5e3

    # Calculate the price according to the formula I defined above & add noice
    price <- base.price + mileage.penalty*mileage*mileage.known + TMU.penalty*(!mileage.known) + rnorm(length(mileage), mean = 0, sd = 200)

    # Put the relevant data in a data frame
    df <- data.frame(price, mileage, mileage.known)
    glimpse(df)

    ## Observations: 100
    ## Variables: 3
    ## $ price         <dbl> 5249.511, 8888.228, 9772.401, 8241.758, 4910.818, …
    ## $ mileage       <dbl> 0.00, 214497.43, 75515.27, 402734.43, 0.00, 0.00, …
    ## $ mileage.known <lgl> FALSE, TRUE, TRUE, TRUE, FALSE, FALSE, TRUE, FALSE…

    ggplot(df, aes(x = mileage, y = price, colour = mileage.known)) + 
      geom_point() + 
      expand_limits(ymin = 0) + 
      theme_minimal()

![](944_blog_post_files/figure-markdown_strict/unnamed-chunk-2-1.png)

    mod <- lm(price ~ mileage+mileage.known)
    summary(mod)

    ## 
    ## Call:
    ## lm(formula = price ~ mileage + mileage.known)
    ## 
    ## Residuals:
    ##     Min      1Q  Median      3Q     Max 
    ## -459.18 -121.87   -5.52  134.82  502.89 
    ## 
    ## Coefficients:
    ##                     Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)        4.975e+03  2.826e+01  176.08   <2e-16 ***
    ## mileage           -5.151e-03  9.815e-05  -52.49   <2e-16 ***
    ## mileage.knownTRUE  5.124e+03  5.640e+01   90.85   <2e-16 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 191.6 on 97 degrees of freedom
    ## Multiple R-squared:  0.9889, Adjusted R-squared:  0.9887 
    ## F-statistic:  4337 on 2 and 97 DF,  p-value: < 2.2e-16

Note that the results aren’t quite what I expected: `(Intercept)`,
implying the base case, is $5.000 instead of $10,000. That’s because
`lm()` is taking the ‘base case’ as unknown mileage, and then adding
extra cost for when mileage is known. Thus, the price of a pristine car
is really the estimates of `(Intercept)` + `mileage.knownTRUE`.

Maybe this isn’t totally dumb
=============================

So this was a, uh, “fun” way to spend a Sunday evening, but was it a
total waste of time from a professional standpoint? I think maybe not? I
can easily imagine environmental data sets in which you’d really like to
know a response to a continuous variable - say, phytoplankton blooms as
a function of nutrients - where sometimes you have the data you want
recorded, and other times you have some qualitative data, like “low” or
“eutrophied”. This approach is likely to be useful in such a situation.

In fact, I am reasonable certain that there is a whole literature about
this problem, but that is *well* beyond the scope of a fun weekend
problem.

[1] This is the first of some number of posts in which I attempt to
teach myself statistics in public.

[2] I would greatly appreciate it if someone could explain to me why
statisticians looked at the normal equation for a line,
*y* = *m**x* + *b*, and said, “Yeah, we’re going to go in another
direction on that.”
