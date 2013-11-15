# Miscellaneous Python Tools

## pycalc

Inspired by [Crunch](http://ryanpcarney.com/technologblog/crunch-gets-crunchier-with-v20) I wrote up a Python script to do something similar. It allows for inline, line-by-line (i.e. no function definitions and for loops) code execution with appended results. Really the only difference between passing the code to Python directly is that it will append the results of expressions to the end of the line as a comment.

It is written with the intention of execution in vim like Crunch, but will happily accept anything from standard input.

Advantages:

* Code execution in-session
* Results appended, leaving original code intact for quick feedback cycle
* Access to anything Python like [NumPy](http://www.numpy.org/), [SciPy](http://www.scipy.org/), [SymPy](http://sympy.org/en/index.html), etc.

Disadvantages:

* Does not execute multi-line code
* Does not remember variables between executions

### Example

Say we have the following block of code in vim:

```
paragraph one

x = 23.21
y = 53.32
norm = lambda x, y: sqrt(x*x+y*y)
norm(x, y)
atan2(y, x)

paragraph three
```

If you have your cursor in that second paragraph you can evaluate it with `{V}!pycalc` in vim (assuming pycalc is in your path and your Python is located at `/usr/bin/python`). The result is the following:

```
paragraph one

x = 23.21
y = 53.32
norm = lambda x, y: sqrt(x*x+y*y)
norm(x, y)# = 58.1526138707
atan2(y, x)# = 1.1602370238

paragraph three
```

## pymap

This script started as some ideas to into my head while writing pycalc. Basically it lets you write small Python functions (really just [lambdas](http://docs.python.org/2/reference/expressions.html#lambda)) and apply them (i.e. [map](http://docs.python.org/2/library/functions.html#map)) to each input line.

*This documentation is incomplete*

It works by reading each line as it comes in from standard input. If a line does not start with a function marker then it is simply kept as an input line. If a line starts with a function marker then it is treated as code to be evaluated. Functions are evaluated as they're encountered with the results remaining persistent across functions. So you can apply one map and then apply a map on that result.

There are five operations you can perform. Each operation is determined by the first character.

* `!`: applied per item
* `*`: applied to list of items and then assigned to items
* `#`: applied per [enumerated](http://docs.python.org/2/library/functions.html#enumerate) item
* `-`: used to [filter](http://docs.python.org/2/library/functions.html#filter) items
* `=`: applied to list of items and then appends result to end of function

After the function marker you simply define your lambda function. If you don't include the `lambda` then it'll prepend it for you.

At this point the best way to explain it is to look at examples.

### Example

#### Example 1

Input:

```
[1, 2, 3]
[4, 5, 6, 7]
[8, 9, 10, 11, 12]
!x: sum(eval(x))
```

After evaluating all lines (such as `ggVG!pymap` in vim):

```
6
22
50
!x: sum(eval(x))
```

If you prefer keeping the input, you can do the following:

```
[1, 2, 3] # 6
[4, 5, 6, 7] # 22
[8, 9, 10, 11, 12] # 50
!x: '%s # %d' % (x, sum(eval(x)))
```

#### Example 2

Input:

```
John, [1, 2, 3]
Sarah, [4, 5, 6, 7]
Bob, [8, 9, 10, 11, 12]
!x: x.split(',')
!x: (x[0], eval(','.join(x[1:])))
!*name, items: 'Sum of %s\'s items: %d' % (name, sum(items))
```

After evaluating all lines (such as `ggVG!pymap` in vim):

```
Sum of John's items: 6
Sum of Sarah's items: 22
Sum of Bob's items: 50
!x: x.split(',')
!x: (x[0], eval(','.join(x[1:])))
!*name, items: 'Sum of %s\'s items: %d' % (name, sum(items))
```

