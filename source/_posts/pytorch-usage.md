---
title: python
date: 2020-09-28 14:39:06
categories:
- python
tags:
- python
- pytorch
- numpy
---

python常见库

<!--more-->

# python

## os

```python
chdir(path)
    Change the current working directory to the specified path.
    path may always be specified as a string.
```

## random

```plain
sample(population, k) method of random.Random instance
    Chooses k unique random elements from a population sequence or set.
    
    Returns a new list containing elements from the population while
    leaving the original population unchanged.  The resulting list is
    in selection order so that all sub-slices will also be valid random
    samples.  This allows raffle winners (the sample) to be partitioned
    into grand prize and second place winners (the subslices).
    
    Members of the population need not be hashable or unique.  If the
    population contains repeats, then each occurrence is a possible
    selection in the sample.
    
    To choose a sample in a range of integers, use range as an argument.
    This is especially fast and space efficient for sampling from a
    large population:   sample(range(10000000), 60)
```

## iter

```python
iter(...)
    iter(iterable) -> iterator
    iter(callable, sentinel) -> iterator
    
    Get an iterator from an object.  In the first form, the argument must
    supply its own iterator, or be a sequence.
    In the second form, the callable is called until it returns the sentinel.
```

## next

```python
next(...)
    next(iterator[, default])
    
    Return the next item from the iterator. If default is given and the iterator
    is exhausted, it is returned instead of raising StopIteration.
```

## zip

```python
In: list(zip(*[[1,2,3],['1',2,3],[1,2,3],[1,2,3]]))
Out: [(1, '1', 1, 1), (2, 2, 2, 2), (3, 3, 3, 3)]
```



# numpy

## random


```python
choice(...) method of mtrand.RandomState instance
    choice(a, size=None, replace=True, p=None)
    
    Generates a random sample from a given 1-D array
    
    Parameters
    -----------
    a : 1-D array-like or int
        If an ndarray, a random sample is generated from its elements.
        If an int, the random sample is generated as if a were np.arange(a)
    size : int or tuple of ints, optional
        Output shape.  If the given shape is, e.g., ``(m, n, k)``, then
        ``m * n * k`` samples are drawn.  Default is None, in which case a
        single value is returned.
    replace : boolean, optional
        Whether the sample is with or without replacement
    p : 1-D array-like, optional
        The probabilities associated with each entry in a.
        If not given the sample assumes a uniform distribution over all
        entries in a.
    
    Returns
    --------
    samples : single item or ndarray
        The generated random samples

    Examples
    ---------
    Generate a uniform random sample from np.arange(5) of size 3 without
    replacement:
    
    np.random.choice(5, 3, replace=False)
        array([3,1,0])
    #This is equivalent to np.random.permutation(np.arange(5))[:3]
```

