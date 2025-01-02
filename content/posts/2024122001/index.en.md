+++
title = "Regular Expressions 101"
date = '2024-12-20T02:41:08+07:00'
draft = true
lang = "en"
slug = "regular-expressions-101"
tags = ["regex", "python"]
+++

## Introduction
`([A-Za-z])\w+\s+\W\D`  
When you come across a regular expression like that, chances are a lot of programmers (myself included) have no clue what it means at first glance. And when it’s time to write a regex for validation, most of us probably head straight to [RegExr](https://regexr.com/) or [regex101](https://regex101.com/). These days, though, we’ve got tools like ChatGPT and GitHub Copilot to help out.

While taking a data-related course, I came across regular expressions. So, I decided to write this post as a way to help myself remember them. Even though this post uses Python for examples, the concepts of regular expressions are pretty much the same across other programming languages too.

## General Tokens
At a basic level, knowing these tokens can help you match a lot of patterns. While there are other tokens as well, I’ve left them out because they’re not used very often.
- `\d`: Matches any digit from **0** to **9**.
- `\s`: Matches **space**, **tab**, and **new line** characters.
- `\w`: Matches any word character, including **a** to **z**, **A** to **Z**, 0 to 9, and _.
- `.`: Matches everything except the **new line** character.

You might have noticed that the tokens are all lowercase. If these tokens are switchched to uppercase, they’ll behave in the opposite way:
- `\D`: Matches any character except digits (**0** to **9**).
- `\S`: Matches any character except **space**, **tab**, and **new line**.
- `\W`: Matches any character except word characters (**a** to **z**, **A** to **Z**, **0** to **9**, and **_**).

Here’s the Python code for demonstration. When you run it, the result will be `['H', 'i', '1', '2', '3']`. The first parameter of the `re.findall()` function is the regular expression.
```python
import re

print(re.findall(r'\w', 'Hi 123'))
```
If the regular expression is changed to `r'\w\w'`, the result will be `['Hi', '12']`. This matches any two consecutive word characters. If it's changed to `r'\w\s\d'`, it will return `['i 1']`. This matches a word character `\w`, followed by a space `\s`, and then a digit `\d`.

If you want to match exactly 9 digits, is it necessary to write \d nine times? To match a range, such as 3 to 5 characters, how should it be written? In such cases, you need to use a **quantifier**.
