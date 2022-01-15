---
layout: post
title: "Integration"
excerpt: "Numerical Integration functions for ploting with python"
categories: [Numerical_Method]
tags: [Python]
---
## Gauss Quadrature
this is numpy-friendly code   
you need to have **_Closed-Form expression_**

### How to use
example for integrating f(x) = sin(x^2)
```python
import numpy as np
import Gauss_Quadrature as gq


def f(x):
    return np.sin(x**2)

x1 = np.arange(0, 10, 0.01)

#integrated value list y
y1 = gq.Quadrature(f, x1)
```
## Simpson's Rule
this is numpy-friendly code   
you need to have values of function and x values with regular interval(no matter odd numbered or even numbered)  
A number of samples will be decreased by more than half  

### How to use
example for integrating y1(note Gauss Quadrature code part)
```python
import Simpson_Rule as sim

#integrate again!
y2, x2 = sim.Integration(y1, x1)
```   
the last value of y2 is same as it's **second value from behind**  

- Gauss Quadrature
  [Code Link](https://gist.github.com/MadPlayer/e14de3127ccfd1bd2485d3d6ad1842bb)

- Simpson's Rule
  [Code Link](https://gist.github.com/MadPlayer/4fa32c859fc912c3ddce3aa72159598d) 
