---
title: 'Python basics'
date: 2016-01-14
permalink: /posts/2016/01/post-python/
tags:
  - SQL
  - python
  - database
---
Below are some notes on Python features accumulated from daily practice.


## String

`\` can be used to escape quotes; `"\"Yes,\" they said."`. By adding an `r` before the first quote, characters prefaced by `\` will not be be interpreted as special characters.

Python strings are immutable. Therefore, assigning to an indexed position in the string results in an error.

To remove spaces remove any whitespace from both the beginning and the end of a string: `srtip()`
Formatted string literals let you include the value of Python expressions inside a string by prefixing the string with `f` or `F` and writing expressions as `{expression}`.

We can combine strings and numbers by using the format() method: `myorder = "I want {0} pieces of item {1} for {2} dollars."; print(myorder.format(1, 2, 3))`
<pre>
>>> year = 2016
>>> event = 'Referendum'
>>> f'Results of the {year} {event}'
'Results of the 2016 Referendum'
# assing an integer after the ':' will cause that field to be a minimum number of characters wide. 
>> table = {'Sjoerd': 4127, 'Jack': 4098, 'Dcab': 7678}
>>> for name, phone in table.items():
...     print(f'{name:10} ==> {phone:10d}')
...
Sjoerd     ==>       4127
Jack       ==>       4098
Dcab       ==>       7678
</pre>


## Data types
Lists supports concatenation: `squares = [1, 4, 9, 16, 25]; squares+[3,6,7]`

The list methods make it very easy to use a list as a stack, where the last element added is the first element retrieved. To add an item to the top of the stack, use `append()`. To retrieve an item from the top of the stack, use `pop()` without an explicit index. 

List Comprehensions provides a concise way to create lists. E.g.,`[(x, y) for x in [1,2,3] for y in [3,1,4] if x != y]`

A tuple consists of a number of values separated by commas and parenthesised by round bracket: `t = (12345, 54321, 'hello!)'`. Tuples are immutable, and usually contain a heterogeneous sequence of elements that are accessed via unpacking: `x,y,z=t`

A set is a collection which is unordered and unindexed. In Python sets are written with curly brackets. Once a set is created, you cannot change its items, but you can add new items using `update()`

## Class

Python is an object oriented programming language. Almost everything in Python is an object, with its properties and methods.

All classes have a function called `__init__()`, which is always executed when the class is being initiated.

The self parameter is a reference to the current instance of the class, and is used to access variables that belongs to the class. It does not have to be named self , you can call it whatever you like, but it has to be the first parameter of any function in the class:
<pre>
>>> class Person:
...  def __init__ (self, name, age):
...   self.name = name
...    self.age = age
...  def myfunc(self):
...  print("Hello my name is " + self.name)
>>> p1 = Person("John", 36)
>>> p1.myfunc()
>>> To Modify Object Properties
>>> p1.age = 40
>>> Delete Object Properties
>>> del p1.age 
</pre>

## Functions 

By default, arguments may be passed to a Python function either by position or explicitly by keyword. A function definition may look like:
	1. Positional-only parameters are placed before a `/`. If so, the parameters' order matters, and the parameters cannot be passed by keyword.
	2. Keyword-only parameters are placed after a `*`, and means that the parameters must be passed by keyword argument.

Arbitrary Argument Lists: A function can be called with an arbitrary number of arguments. These arguments will be wrapped up in a [tuple](https://docs.python.org/3/tutorial/datastructures.html#tut-tuples). Before the variable number of arguments, zero or more normal arguments may occur.

<pre>
>>>def concat(*args, sep="/"):
...		return sep.join(args)
>>>concat("earth", "mars", "venus")
	'earth/mars/venus'
>>>concat("earth", "mars", "venus", sep=".")
	'earth.mars.venus'
</pre>

Dictionaries can deliver keyword arguments with the `**` operator.
<pre>
>>>d = {"voltage": "four million", "state": "bleedin' demised", "action": "VOOM"}
...parrot(**d)
</pre>

Small anonymous functions can be created with the `lambda` keyword. They are syntactically restricted to a single expression. Like nested function definitions, lambda functions can reference variables from the containing scope
<pre>
>>> def make_incrementor(n):
...     return lambda x: x + n
...
>>> f = make_incrementor(42)
>>> f(0)
42
>>> f(1)
43
</pre>

## Looping

When looping through dics, the key and corresponding value can be retrieved at the same time using the `items()` method
<pre>
>>> knights = {'gallahad': 'the pure', 'robin': 'the brave'}
>>> for k, v in knights.items():
...     print(k, v)
...
gallahad the pure
robin the brave
</pre>`

When looping through a sequence, the position index and corresponding value can be retrieved at the same time using the enumerate() function
<pre>
>>> for i, v in enumerate(['tic', 'tac', 'toe']):
...     print(i, v)
...
0 tic
1 tac
2 toe
</pre>

To loop over two or more sequences at the same time, the entries can be paired with the `zip()` function.
<pre>
>>> questions = ['name', 'quest', 'favorite color']
>>> answers = ['lancelot', 'the holy grail', 'blue']
>>> for q, a in zip(questions, answers):
...     print('What is your {0}?  It is {1}.'.format(q, a))
...
What is your name?  It is lancelot.
What is your quest?  It is the holy grail.
What is your favorite color?  It is blue.
</pre>

To loop over a sequence in reverse, first specify the sequence in a forward direction and then call the `reversed()` function.

To loop over a sequence in sorted order, use the `sorted()` function.

## Modules

By adding the following block at the end of the module, you can make the file usable as a script as well as an importable module.
<pre>
>>>if __name__ == "__main__":
	# do something here, usually call a function in the module
</pre>

The `dir()` function can be used to find out wh9ch names a module defies: `import fibo, sys; dir(fibo)`. 

If a package’s `__init__.py` code defines a list named `__all__`, it is taken to be the list of module names that should be imported when from package import * is encountered.

You can also write relative imports, with the from module import name form of import statement. These imports use leading dots to indicate the current and parent packages involved in the relative import:
<pre>
>>>from . import echo
>>>from .. import formats
>>>from ..filters import equalizer
</pre>

## Erroes and exceptions

The `try` block lets you test a block of code for errors.

The `except` block lets you handle the error.

The `finally` block lets you execute code, regardless of the result of the try- and except blocks.

The `else` keyword to define a block of code to be executed if no errors were raised

A `try` statement may have more than one except clause, to specify handlers for different exceptions. At most one handler will be executed.  

An `except` clause may name multiple exceptions as a parenthesized tuple,
<pre>
>>> while True:
...	try:
...		x = int(input("Please enter a number: "))
...       break
...	except (RuntimeError, TypeError, NameError):
...		pass
</pre>

The `raise` statement allows the programmer to force a specified exception to occur.
<pre>
>>> try:
...     raise NameError('HiThere')
... except NameError:
...     print('An exception flew by!')
...     raise
</pre>

The `finally` block, if specified, will be executed regardless if the try block raises an error or not.
<pre>
>>> def divide(x, y):
...     try:
...         result = x / y
...     except ZeroDivisionError:
...         print("division by zero!")
...     else:
...         print("result is", result)
...     finally:
...         print("executing finally clause")
...
>>> divide(2, 1)
result is 2.0
executing finally clause
>>> divide(2, 0)
division by zero!
executing finally clause
>>> divide("2", "1")
executing finally clause
Traceback (most recent call last):
TypeError: unsupported operand type(s) for /: 'str' and 'str'
</pre>


## Standard Library

Operating System Interface. `os.getcwd()` to return current working directory; `os.chdir('/server/accesslogs')` to change to directory; `os.system('pwd')` run command line operations. `os.remove()` to remove file; `osrmdir()` to remove folder;

The `glob` module provides a function for making file lists from directory wildcard searches: `glob.glob(*.py)`

Command line arguments. 
<pre>
>>> python demo.py one two three
>>> import sys
>>> print(sys.argv)
['demo.py', 'one', 'two', 'three']
</pre>

Use `pprint` to print prettier

## File handlling 

The key function for working with files in Python is the `open()` function. `"r" - Read, "a" - Append, "w" - Write, "x" - Create, "t" - Text, "b" - Binary`. 
<pre>
# read
f = open("demofile.txt", "r")
print(f.readline())
f.close() 
# append
f = open("demofile2.txt", "a")
f.write("Now the file has more content!")
f.close()
# Open the file "demofile3.txt" and overwrite the content
f = open("demofile3.txt", "w")
f.write("Woops! I have deleted the content!")
f.close()
</pre>


















