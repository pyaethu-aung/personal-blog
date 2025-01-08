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

### Range
```python
import re

if re.fullmatch(r'[A-Za-z0-9]+', 'NoSpaceAndSpecialCharacter0123456789'):
    print('Match')
else:
    print('Not match')
```
Most of the programmers commonly use the above regular expressions which allow only alphabets and numbers, while excluding space and special characters. In this case, the code will execute `print('Match')`. However, if there are any **space** or **special character** in the string, it will execute `print('Not match')`.

### Not In `^`
The `^` character, which is discussed in [Part 1]({{< relref path="2024122001/index.en.md" lang="en" >}}) as indicating the start of the string, is also used within sets and ranges to mean not in.
```python
import re

print(re.findall(r'[^A-Za-z0-9]+', 'NoSpaceAndSpecialCharacter#!0-0 123456789'))
print(re.findall(r'[^nl]ot', 'not hot lot'))k
```
For the first print statement, the pattern matches anything that is not an alphabet or a digit. Hence, the result will be `['#!', '-', ' ']`.

For the second print statement, the pattern matches anything that does not start with **n** or **l**. The output will be `['hot']`.
