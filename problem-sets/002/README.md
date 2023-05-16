# Problem set 002

CIA, DAGs, Matching, and IV

## Goal

**README:** In this problem set, we will attempt to estimate the effect of school construction on reducing the Black-White income gap in the United States. Specifically, we will focus on the causal effect of the [Rosenwald schools](https://en.wikipedia.org/wiki/Rosenwald_School)—a large set of schools constructed in the early 1900s in rural counties in the South to give Black families better educational opportunities (relative to prevailing norms of discrimination and underfunding). The schools were funded by Julius Rosenwald and Booker T. Washington. 

[Aaronson and Mazumder have a really cool paper](https://www.jstor.org/stable/pdf/10.1086/662962.pdf) on how the Rosenwald schools helped close the Black-White education gap in the South. Whereas Aaronson and Mazumder use microdata and focus on education gaps (and find large and significant increases in education for rural Black children who had access to Rosenwald schools), we are going to look a the *income gap* using *aggregated data*.

We will walk through several regression specifications and empirical strategies—asking whether we any of these approaches plausibly identifies the causal effect of Rosenwald schools on the racial income gap.

More broadly, we want to 

- refresh the topics you learned in previous metrics course
- refresh/build/strengthen your `R` abilities
- build intuition about causality related to the regression, CIA, and matching lectures.

## Data

The data in this problem set come from four sources; three of which I downloaded from [NHGIS](https://nhgis.org) (see below). The four source—the *Mazumder* data—come from the replication files of a published paper by Daniel Aarsonson and Bhash Mazumder.

1. the 1860 US Census
2. the 2010 US Census
3. the American Community Survey (ACS), 2006-2010
4. *Paper:* [The Impact of Rosenwald Schools on Black Achievement](https://www.journals.uchicago.edu/doi/full/10.1086/662962)

The table below describes each variable in the dataset.

| Name | Description (Source) |
|:---|:---|
| `state` | State name |
| `county` | County name |
| `gisjoin` | Code for GIS joining |
| `pop_enslaved_1860` | County population of enslaved persons in 1860 (Census) |
| `pop_total_1860` | County population in 1860 (Census) |
| `pop_total_2010` | County population in 2010 (Census) |
| `pop_black_2010` | County black population in 2010 (Census) |
| `pop_white_2010` | County white population in 2010 (Census) |
| `income_black_2010` | County median income for Black households 2006-2010 (ACS) |
| `income_white_2010` | County median income for White households 2006-2010 (ACS) |
| `pct_pop_enslaved_1860` | Percent of county population enslaved in 1860 (Census) |
| `had_rosenwald_school` | Indicator variable: Did the county have a Rosenwald school? (Mazumder) |

The dataset focus on ten states that (unsuccessfully) attempted to secede (South Carolina, Mississippi, Florida, Alabama, Georgia, Louisiana, Texas, Virginia, North Carolina, and Tennessee). Each of these states was a "slave state" in 1860, *i.e.*, they allowed Black enslavement.

### Section 1: General regression analysis and the CIA

**01** Load whichever packages you think you'll want—and the dataset [`data-002.csv`](https://raw.githack.com/edrubin/EC607S23/master/problem-sets/002/data-002.csv). At a minimum you'll want something that makes pretty figures (e.g., `ggplot2`) and `fixest`.

**02** Create a few figures that illustrate the main variables and their relationships. Specifically, I'd like to see:

- a histogram of the county-level income gap (`income_white_2010 - income_black_2010`) split by whether the county had a Rosenwald school (`had_rosenwald_school`)
- a histogram of the percent of the population in 1860 that was enslaved (`pct_pop_enslaved_1860`) also split by whether the county had a Rosenwald school (`had_rosenwald_school`)
- a scatter plot with `pct_pop_enslaved_1860` on the *x* axis and the county income gap on the *y* axis with the points colored by whether or not the county had a Rosenwald school.

*Note:* Make sure you label your axes and that the figure is generally aesthetically pleasing.

**03** Time for some regressions. In each regression in this problem set, the outcome with be the county's difference between the median White income and the median Black income (i.e., `income_white_2010 - income_black_2010`). We will call this variable the (race) income gap.

For this first regression, just regress the income gap on the indicator for whether the county had a Rosenwald school. Report your results and briefly discuss the identifying assumptions. (What assumption would be necessary to give these estimates a causal interpretation?)

**04** Now add a state fixed effect. Report your findings and discuss the conditional independence assumption required by the regression (for a causal interpretation).

*Hint:* For a `state` fixed effect in `feols()`: `feols(y ~ x | state, data = fake_df)`

**05** Now add two controls from 1860: the percent of the population that was enslaved in 1860 (`pct_pop_enslaved_1860`) and the total county population in 1860 (`pop_total_1860`). Report your results. (You should still have a state fixed effect.)

**06** Does the change in the estimated coefficients on `had_rosenwald_school` from **04** and **06** match what you would have guessed if these new variables (`pct_pop_enslaved_1860` and `pop_total_1860`) were causing bias? Explain.

**07** What is the required conditional independence assumption required by **06**? Do you think it is valid? Explain your answer using a DAG (and discuss which variables should and should not be used as controls).

**08** What are your thoughts on adding the county's 2010 population (Black, White, and/or total) as a control in the regression? Will these controls help or hurt our CIA? Explain (maybe use a DAG?).

### Section 2: Matching

In this section, we are going to use two variables to match observations: `pct_pop_enslaved_1860` and `pop_total_1860`.

**09**  Use the Mahalanobis distance metric to calculate the "distance" between each treated (`had_rosenwald_school == 1`) county and its nearest untreated neighbor (distance in terms of the two variables we're using to match).

Once you find these distances, create a nice histogram of the treated units' distances to their nearest control unit.

*Hints:* The function `mahalanobis_dist()` in the `MatchIt` package does exactly what you think it would do: calculates Mahalanobis distance between all points in a dataset. However, this distance matrix will have the distance between *all* units. Your job is to get the distance from each *treated* unit to its nearest *control* unit. The `filter()` (in `tidyverse`) and `apply()` functions might help here.

**10** Now take the difference between each treated county and its closed control county. Once you've done that, take the average of all those pairwise differences. In other words: Estimate the treatment effect using nearest-neighbor matching (on Mahalanobis distance).

*Note:* Don't use `MatchIt`'s matching function. You should program this step yourself (of course with the help of ChatGPT and your other friends).

**11** How does the estimate in **10** differ from your prior estimates? Which do you "trust"?

**12** Is the estimator in **10** estimating an ATE or some other treatment effect? Explain (and name it if it is not the ATE).

### Section 3: Propensity-score methods

**13** Let's try some propensity-score matching. Step 1: Estimate some propensity scores!

To estimate propensity scores, we'll use a logistic regression. Don't worry if you don't know what it is. For now, all you need to know is that it is a nonlinear regression used for binary outcomes. We can use `feglm` from the `fixest` package to estimate a logistic regression, for example

`femlm(binary_y ~ I(x1^2) + I(x2^2) + x1 + x2 + x1:x2, data = fake_df, family = 'logit')`

will estimate a logistic regression (notice the argument `family = 'logit`). The example also shows you how to square variables and take interactions.

You need to estimate a logistic regression as a function of `pct_pop_enslaved_1860` and `pop_total_1860`, their squares, and their interaction. 

Finally, you can grab those beautiful (estimated) propensity scores from your saved regression using `saved_reg$fitted.values` (where `saved_reg` is my saved regression object output from `feglm`). You'll want to add those (estimated) propensity scores to your dataset.

**14** Estimate a regression where you control for the original variables **and** for the estimated propensity scores. Does anything change?? Report your results.

**15** Wait! How does overlap look? Make a figure to document the overlap.

**16** What percent of your observations aren't supported by overlap?

**17** Why do we care about enforcing overlap?

**18** Repeat the regression with overlap enforced. Report your results. Did anything change?

**19** Now weight your regression using the (inverse) propensity scores (as discussed in lecture). Report your results. Did anything change yet??

**20** Maybe we need a doubly robust approach. Estimate the "controlling for stuff" regression from **06** *for each block*, where blocks are based upon observations' propensity scores.

*Note:* If you're using state fixed effects with small-ish blocks, then some of the blocks may not have variation in `had_rosenwald_school`. For the sake of making your life easier, you can drop the state fixed effects or make the blocks bigger.

Report each block's estimated treatment effect and the ATE.

