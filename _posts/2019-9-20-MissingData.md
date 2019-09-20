---
layout: post
title: A beginner's guide to missing data
---

![_config.yml]({{ site.baseurl }}/images/sweden.jpg)

**What happens when researchers are missing information? Do they throw out the entire study?**
<br>

*Of course not!*
<br>

We can think of missing data like trying to build a piece of IKEA furniture. When you open the box ready to build your Swedish computer desk, you might find that some of the screws, nuts, and/or bolts are missing from the box. You can probably still build the desk, but it might not be as stable as it would have been had you had all the correct parts. 
<br>

![alt text](https://mnsearch.org/wp-content/uploads/iStock_000020875132Small.jpg "Source: mnsearch.org")

Missing data is quite common and occurs when a dataset lacks values for one or more variables. There are many ways to deal with a few missing observations in a dataset. However, the way  we deal with missing data actually depends on the *type* of missing data:
<br>
1. __Missing Completely At Random (MCAR)__: say while you are building your IKEA desk, you notice that a random number of screws, nuts, and bolts are each missing from the box. Similarly, MCAR data contains values that are randomly missing from all subsets of the data. 

2. **Missing At Random (MAR)**: this type of missing data would be similar to opening your IKEA box to find that a random number of only screws are missing, but you still have all your nuts and bolts. MAR data contains values that are randomly missing from only 1 subset of the data. 

3. **Missing Not At Random (MNAR)**: This would be equivalent to *all* of the screws missing from the IKEA box because there was an issue at the factory. MNAR data occurs when all the values are missing from only 1 subset of the data, and the reason why these values are missing has an impact on our data analysis. 

So: what do we do with these different types of missing data?
<br>

### Dealing with completely random missing data


When dealing with MCAR data, we can be pretty confident that the reason why we have missing data is not related to the data itself. In other words, we can assume that tossing out the observations with missing data will not throw a wrench in our data analysis. 
<br>

That's great! But how do we remove the observations with missing values?
<br>

Say we have a dataset that shows the number of bananas eaten each business day of the week for a number of monkeys:
<br>

|Monkey | Monday | Tuesday | Wednesday | Thursday | Friday |
|:----------|-----------|---------|---|---|---|
| Matilda   | 11 | 13 | 12 | 15 | 12 |
| Jerry | 10 | 13 | **0** | 12 | 11 |
| Boris | 14 | 10 | 12 | 13 | 14 |

We see that the zookeeper forgot to mark down how many bananas Jerry ate on Wednesday. Therefore, we want to get rid of Jerry's row. To start, we will need the `Pandas` and `NumPy` packages in python:
<br>
    
```python
import pandas as pd
import numpy as np
```
  
Then we will load our monkey dataset using the `pandas read_csv()` function:

```python
monkey_data = pd.read_csv('monkey-bananas.csv')
```

There are many ways to mark missing data in python (e.g. 0, NA, etc.), but we want to convert all of our missing data cells to the same value: NaN (or "Not a Number"). The purpose of this is to convert the missing data cells to a value that `NumPy` can recognize.

```python
monkey_data[:] = monkey_data[:].replace(0, np.NaN)
print(monkey_data)

    Monkey  Monday  Tuesday  Wednesday  Thursday  Friday
0  Matilda      11       13       12.0        15      12
1    Jerry      10       13        NaN        12      11
2    Boris      14       10       12.0        13      14
```

Now, we can target the rows where we see NaNs and delete those rows from our dataset.

```python
monkey_data.dropna(inplace=True)
print(monkey_data)

    Monkey  Monday  Tuesday  Wednesday  Thursday  Friday
0  Matilda      11       13       12.0        15      12
2    Boris      14       10       12.0        13      14
```

We can see if this worked by checking the number of rows in the modified dataset.

```python
print(monkey_data.shape)

(2, 6)
```

Now we have gotten rid of our randomly missing data and we are ready to carry out our analyses!

### Dealing with nonrandom and random missing data

Dealing with the other two types of missing data is a little bit more complicated. Because the data isn't missing completely at random, if we try to get rid of observations that contain missing data, we could be getting rid of important information. When we replace missing data cells with a new value calculated from other observations, this is called **imputing**. There are many complex ways to impute missing data, but I will just go over a basic way of replacing missing data with the mean of the column.

Say, instead of getting rid of Jerry's row, we just decided to fill the missing data cell with the mean of all the values in the Wednesday column.

We already know that the Wednesday column is the one with the missing data, but if we didn't have this prior information we would first need to locate all the columns containing missing data. This next line of code is just looking through `monkey_data` and returning the index `inds` of the columns with missing data.

```python
inds = monkey_data.columns[monkey_data.isnull().any()]
print(inds)

Index(['Wednesday'], dtype='object')
```

The `nanmean` function in `NumPy` conveniently takes the mean of the column that has the missing value. We can put the object `inds` into this function to return the mean of our column(s) of interest.

```python
monkey_data[inds] = monkey_data[inds].fillna(value = monkey_data[inds].mean())
print(monkey_data)   
                                             
    Monkey  Monday  Tuesday  Wednesday  Thursday  Friday
0  Matilda      11       13       12.0        15      12
1    Jerry      10       13       12.0        12      11
2    Boris      14       10       12.0        13      14                                           
```

And Voíla! We now have a crude estimate of the number of bananas Jerry ate on Wednesday!