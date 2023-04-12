# Problem set 1

**1.** Write a function that uses OLS to estimate the coefficients and standard errors of a linear regression model.

The function should

- accept two arguments: `y` and `X` (both matrices)
- output estimated coefficients
- output estimated standard errors

Your function should only use matrix operations (e.g., `%*%`) and basic summary statistics (e.g., `sum`). Do not use more complex functions.

*Hint:* The function named `function()` allows you to write a function. For example, 

```{r}
function(a,b) {
  a + b
}
```

**2.** Show that your function works using the `mtcars` dataset in `R`. Specifically: Use your function to regress `mpg` on an intercept, the number of cylinders (`cyl`), horsepower (`hp`), and weight `wt`.

Compare your results to those of `lm`.

**3.** What assumptions do your standard errors rely upon (to be approximately correct)?

**4.** What assumptions do your coefficient estimates need to be causal?

