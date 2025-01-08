+++
title = "Regular Expressions 101 (Part 2)"
date = '2025-01-08T22:24:00+07:00'
draft = false
lang = "en"
slug = "regular-expressions-101-part-02"
tags = ["regex", "python"]
+++

[မြန်မာဘာသာဖြင့် ဖတ်ရှုရန်]({{< relref path="2025010501/index.my.md" lang="my" >}})

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

## Greedy and Non-Greedy Quantifiers
### Greedy Quantifier
Standard quantifiers like `.`, `?`, `+`, `*`, and `{from, to}` are **greedy** by default. Greedy means that they will try to match as much as possible while still satisfying the pattern. In other words, they match the longest possible string that fits.

Though it may seem confusing, the following code will help clarify how greedy quantifier works:
```python
import re

print(re.findall(r'\w+', 'abcdefh123!@#'))
```
When you run the code, the result will be as expected: `['abcdefh123']`. When you change the pattern to `r'\d+'`, the result will be `['123']`.

The behavior of the `+` quantifier is not always straightforward and can be a bit tricky when it comes to **backtracking**. Let’s break down the example step-by-step to better understand how it works:
```python
import re

print(re.findall(r'.*hello', 'xhello123'))
```
It may seem like `.*` matches the entire sentence, resulting in `['xhello123']`. However, `.*` does match the whole sentence initially, but then it processes the remaining tokens one by one in backtracking. If this is unclear, I’ll explain it step by step below:

1. `.*`: `xhello123`
2. `.*h`: `xhello123` -> `xhello12`3 -> `xhello1`23 -> `xhello`123 -> `xhell`o123 -> `xhel`lo123 -> `xhe`llo123 -> `xh`ello123
3. `.*hello`: `xhello`

I borrowed this example from [DataCamp](https://www.datacamp.com/)'s [Regular Expression in Python course](https://campus.datacamp.com/courses/regular-expressions-in-python) without hesitation. If you’re a beginner looking to learn Python and data engineering, I highly recommend [DataCamp](https://www.datacamp.com/).

### Non-Greedy Quantifier
The non-greedy quantifier, also known as the lazy quantifier, differs from the greedy quantifier in that it matches the smallest possible portion of the string.
```python
import re

print(re.findall(r'\w+?', 'abcdefh123!@#'))
```
When you run the code, the result will be as expected: `<re.Match object; span=(0, 1), match='a'>`.

## Outroduction
As mentioned in Part 1, mastering regular expressions requires a lot of practice. It’s essential for both gaining a deep understanding and improving your skills. I’ll share more about capturing groups and backreferences in future posts.

## References:
- [Regular Expression in Python by Data Camp](https://campus.datacamp.com/courses/regular-expressions-in-python)
- [regex101.com](https://regex101.com/)
- [Set and Ranges by javascript.info](https://javascript.info/regexp-character-sets-and-ranges)
