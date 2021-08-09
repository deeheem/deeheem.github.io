---
title: 5 Surprises to expect when learning Python
date: 2021-04-09 20:26:42
category:
- Languages
- Python
tags:
- "#python"
- "#java"
---

## Introduction

When Java is your everyday language, and you encounter Python occasionally as part of your day-to-day routine, Python can seem to be a bliss: you don’t need to go through docs or learn the language first in order to understand simple scripts written in Python. That’s when you realise the beauty of the language, but never seem to bother much about it. 

Then there comes a point in time where you now want to write some Python code and the ability to simply read Python code does not suffice. You go to your favourite place to learn about new technology - i.e. the official docs. And while you are learning about it, in your mind you are comparing the features with your favourite (till now?) language: Java. You come across some “AHA!” moments and some “WTF?” moments.

Here are 5 such moments I faced when going through the same process of learning Python as a Java developer (leaving the categorisation between “AHA!” And “WTF?” to the reader’s imagination):

## The 5 surprises

### (1) Docstrings inside functions

I really like the customisations we can do in the Javadoc comments over classes, methods etc. I heavily use a lot of [tags](http://www.drjava.org/docs/user/ch10.html#javadoc-writing) while documenting stuff. 

In Python, when I saw the docstrings are meant to be written inside the function definitions, it looked like an unclean approach! I mean, just compare the code documentation in the below examples:

**Example 1: JavaDoc in Java**

```java
/**
* Returns an Image object that can then be painted on the screen. 
* The url argument must specify an absolute <a href="#{@link}">{@link URL}</a>. The name
* argument is a specifier that is relative to the url argument. 
* <p>
* This method always returns immediately, whether or not the 
* image exists. When this applet attempts to draw the image on
* the screen, the data will be loaded. The graphics primitives 
* that draw the image will incrementally paint on the screen. 
*
* @param  url  an absolute URL giving the base location of the image
* @param  name the location of the image, relative to the url argument
* @return      the image at the specified URL
* @see         Image
*/
public Image getImage(URL url, String name) {
    try {
        return getImage(new URL(url, name));
    } catch (MalformedURLException e) {
        return null;
    }
}
```

**Example 2: DocString in Python**

```python
def example_generator(n):
    """Generators have a ``Yields`` section instead of a ``Returns`` section.

    Args:
        n (int): The upper limit of the range to generate, from 0 to `n` - 1.

    Yields:
        int: The next number in the range of 0 to `n` - 1.

    Examples:
        Examples should be written in doctest format, and should illustrate how
        to use the function.

        >>> print([i for i in example_generator(4)])
        [0, 1, 2, 3]

    """
    for i in range(n):
        yield i
```

I tried to find a reasoning behind why this would be the case in Python, but looks like there doesn't exist an official explanation. One possible reason and the only benefit I could think of is that in Python, unlike Java, we can access the doc through code using the `__doc__` utility, hence making it easy for the compiler to extract the doc from within the function than messing it up when trying to find it outside the function. 

### (2) Default value evaluates only once

In Python, a function like this will accumulate the arguments passed to it on subsequent calls:

```python
def f(a, L=[]):
    L.append(a)
    return L

print(f(1))
print(f(2))
print(f(3))
```

This will print:

```text
[1]
[1, 2]
[1, 2, 3]
```

To avoid this and make this work in the Java-expected way (of not sharing the arguments between function calls), we need to put in extra efforts:

```python
def f(a, L=None):
    if L in None:
        L=[]
    L.append(a)
    return L
```

### (3) Mixed casing

The naming of classes and functions is inconsistent. For classes, `UpperCamelCase` is the convention, while for functions and methods, `lowercase_with_underscores` is the convention.

_(unclean_approach++?)_

### (4) No private attributes inside classes

Like whattt. But there exist a lot of hacks around the idea of private attributes in Python that require developer discipline. The language itself isn’t restricting the developer to misuse the phenomena!

### (5) The “Batteries included philosophy”

The Python source distribution has a rich and versatile standard library that is immediately available, without making the user download separate packages.

Functionality like formatting strings, json conversion, sending emails - are a few utilities among many which are available out of the box.

Think of doing the same in Java? Get ready to deal with JARs, or introduce dependency management tools like Maven!

## Final words

There are multiple other nuances in Python and Java but these 5 top the list for me which trigger my OCD associated with clean code (some in good ways and some in bad).

Let me know in the comments section what were some of your “AHA!” and “WTF?” moments!