---
layout: post
title: Running R in Python with `Rpy2`  
category: Data Science
categories: ['R', 'R2py', 'eda', 'programming']
tags: ['R', 'R2py', 'eda', 'programming']
---

---
<h2 class="no_toc">Table of Contents</h2>

* TOC
{:toc}

<!-- Need this for table of contents above -->
---

## The power of R

Although a mature ML practitioner should be comfortable with both Python and R, it is always entertaining to tune your ears whenever there is 'friendly' banter between R and Python enthusiasts.

Python was my first data science language, and I must confess that I usually have the TidyVerse cheatsheet open (amongst others) whenever I use R to do exploratory analysis. 

Despite Python being so useful because of their high level frameworks that have abstracted away the details, one quickly realises that in certain areas R still remains king for data analysis.

An example of this would be time series analysis. The `statsmodels` library, whilst useful, has documentation that is vague at best. It assumes that you already know what is going on. 

To the best of my knowledge there is no function in Python that automatically fits an ARIMA model in R. As a result, we have enthusiasts writing [7 inner `for` loops](https://kaggle.com/kruthik93/utilizing-arima-to-forecast-uber-s-market-demand) to try and find an optimal SARIMAX model via grid search. 

Sometimes it's just better to use R a convenient function like `auto.arima()`. 

We begin with how to set up `rpy2` and a simple example. Despite being so simple, one must tread carefully because the behaviour of converting objects between 2 languages can be unexpected. 

## Setting up Rpy2

Let's try running some basic R by porting it into our Python environment. We will need 2 components:

```python
import rpy2.robjects as robjects
from rpy2.rojects.packages import importr
```

We need the `robjects` module in order to work with R objects and functions, whilst `importr` will allow us to import R packages into our environment to use.

Alright, let's test it out!

```python
pnorm = robjects.r['pnorm']
q = pnorm(1.96)
```

As you might know, `pnorm` will return a Gaussian CDF given the quantile, which is 0.975.

Now, let's say you wanted to manipulate this with some arithmetic.

```python
q_prime = q + 1
print(q_prime)
```

You might find it surprising for the output to be 

```
[1] 0.9750021 1.0000000
```

It turns out that `pnorm` is a `rpy2.robjects.vectors.FloatVector` object, and thus by adding something you are instead appending to the vector. 

To make this work you would need to index the object

```python
q_prime = q[0] + 1
```

which returns

```
1.9750021
```

Some R objects can be much more complicated to deal with. To make your life slightly easier, `rpy2` has a convenient function that allows (somewhat) easier translation of objects across languages. 

```python
from rpy2.robjects import pandas2ri
from rpy2.robjects import numpy2ri

pandas2ri.activate()
numpy2ri.activate()
```

`pandas2ri` and `numpy2ri` allow conversion from Python pandas and numpy objects to an appropriate R object and vice versa. 

If you try running the code above, the returned object will be a `np.ndarray`

```python
[0.9750021]
```

and you still will have to be careful because the '+ 1' will be element-wise addition! 

As you can imagine this can be 'fun' to debug if you make assumptions about the behaviour of returned R objects.

__You have been warned.__

Now let's try do something that is a little more complicated like porting R functions into Python. 

## Writing R functions

You're really not going to be bothered with using R if you just wanted to compute the CDF of a Gaussian, so let's look at how you can utilise `rpy2` to call `auto.arima()`.

If you are using this locally, make sure you have installed the `forecast` package using an R interface.

If not you may need to install it using Python. 

```python
utils = importr('utils')
utils.install_packages("forecast")
```

However, when I tried this I ran into problems so I would recommend to install packages using R. 

First, we write a R function, store it as a string and save it to a Python variable.

```python
 
auto_arima_R = """

auto_arima_R <- function(ts_R, steps, freq){

    library(forecast)

    ts_obj = ts(ts_R$sales, freq=freq)

    model <- auto.arima(ts_obj, seasonal=TRUE)
    predict <- forecast(model, h=steps)

    list(pred=predict[4], pred=model$fitted)
}
"""

auto_arima_R = robjects.r(auto_arima_R)
```

Alright, so now we have a python function that will call the `auto.arima()` function from the `forecast` package and return a n-steps ahead prediction. 

Let's generate some data and use it for forecasting. 

```python
import numpy as np
import pandas as pd
import random 

a = 2; b = 3
mu = 1; sd = 2
n = 200 

def rand_points(a, b, mu, sd, n):
  arr = []
  for i in range(n):
    arr.append(a*i + b + random.gauss(mu, sd))
  return arr
    
index = np.arange(0, n)
sales = rand_points(a, b, mu, sd, n)


df = pd.DataFrame({
    'date': index, 
    'sales': sales
})

print(df.head())

result = auto_arima_R(df, steps=8, freq=4)
```

You should be able to extract the result by indexing

```python
print(result[0][0])  # predictions 
print(result[1])     # fitted values 
```

So there you go! No more writing `for` loops to fit ARIMA models to your time series!

## Conclusion

Read the [docs](https://rpy2.readthedocs.io/en/version_2.8.x/index.html) and test the behaviour VERY carefully - you never know what's going to happen. 

The best way to utilise this wonderful library is to return the simplest object type you can e.g. a vector, and mold the vector back into a DataFrame or matrix using Python so you don't run into unexpected behaviours.

With great power comes great responsibility =)