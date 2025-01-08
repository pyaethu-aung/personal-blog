+++
title = "Regular Expressions 101 (Part 2)"
date = '2025-01-08T22:24:00+07:00'
draft = true
lang = "en"
slug = "regular-expressions-101-part-02"
tags = ["regex", "python"]
+++

[Part 1]({{< relref path="2024122001/index.en.md" lang="en" >}})

## Pipe Metacharacter
Aside from the metacharacters discussed in [Part 1]({{< relref path="2024122001/index.en.md" lang="en" >}}), `.`, `?`, `+`, `*`, `^`, and `$`, another useful one is the `|`.
```python
import re

print(re.findall(r'Go|Python', 'I\'m intrested in Go, JavaScript, Python, and SQL'))
```
Just like in other programming languages, the `|` can be seen as an **OR operator**. In the Python code above, since it matches either **Go** or **Python**, the result will be `['Go', 'Python']`.

## Character Classes
Square brackets `[]` are used in regular expressions to define **sets** and **ranges**.
- A **set** allows you to specify a collection of characters to match. For example, [abc] matches any one of `a`, `b`, or `c`.
- A **range** lets you specify a range of characters. For example, `[a-z]` matches any lowercase letter from **a** to **z**. `[0-9]` matches any digit from **1** to **9**.

### Set
```python
import re

print(re.findall(r'[nl]ot', 'Not not Hot hot Lot lot'))
```
In the above code, `r'[nl]ot'` specifies that the string must start with **n** or **l**, followed by **ot**. As a result, the matches will be `['not', 'lot']`.

If you want the `[nl]` expression to be case-insensitive, the pipe character, `|`, can be used to achieve this.
```python
import re

print(re.findall(r'[N|nL|l]ot', 'Not not Hot hot Lot lot'))
```
The result will be `['Not', 'not', 'Lot', 'lot']`. Instead of using the `|` operator, you can achieve the same result by using the **ignorecase flag** to make the pattern case-insensitive as below.
```python
import re

print(re.findall(r'[nl]ot', 'Not not Hot hot Lot lot', re.IGNORECASE))
```
