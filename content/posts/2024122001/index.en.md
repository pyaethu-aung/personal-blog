+++
title = "Regular Expressions 101"
date = '2024-12-20T02:41:08+07:00'
draft = false
lang = "en"
slug = "regular-expressions-101"
tags = ["regex", "python"]
+++

[မြန်မာဘာသာဖြင့် ဖတ်ရှုရန်]({{< relref path="2024122001/index.my.md" lang="my" >}})

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

## General Quantifiers
Check below for the most commonly used and simplest quantifiers:
- `?`: **Zero** or **one** occurrence.
- `*`: **Zero** or **more** occurrences.
- `+`: **One** or **more** occurrences.
- `{5}`: Exactly **five** occurrences.
- `{5,9}`: Between **five** and **nine** occurrences.
- `{5,}`: **five** or **more** occurrences.

```python
import re

print(re.findall(r'\d+', 'Hi 123 4567 89101112'))
```
In this Python example, when you match one or more digits `\d+`, the result will be `['123', '4567', '89101112']` because it matches consecutive digits.

If you match exactly 4 digits using `r'\d{4}'`, the result will be `['4567', '8910', '1112']`, since it matches only sequences of exactly 4 digits.

If you use `r'\d{4,}'`, the result will be `['4567', '89101112']`, as it matches sequences of 4 or more digits.

One important thing to note here is that quantifiers apply **immediately to the left**. The quantifier will only affect the character or token directly to its left. This means it will only apply to the character or token it’s placed next to, not to the whole pattern.
```python
import re

print(re.findall(r'1\w+', '1a1b1c1d1e1f1g'))
```
In this Python code, the result will be `['1a1b1c1d1e1f1g']`, not `['1a', '1b', '1c', '1d', '1e', '1f', '1g']`. This happens because the `+` quantifier applies to `\w` immediately to the left of it. The quantifier doesn’t affect both `1` and `\w` together. Instead, the regular expression matches a sequence where a `1` is followed by one or more word characters, and it continues matching the entire string as a single match.

## Excaping Special Character
```python
import re

print(re.findall(r'.\s', 'This is first sentence. And this is second sentence.'))
```
In this Python code, the intention is to check if there’s a **full stop** followed by a **space**. However, since the dot `.` is a special character in regular expressions to match any character, the result will be `['s ', 's ', 't ', '. ', 'd ', 's ', 's ', 'd ']`. This happens because `.` matches any character, so it’s matching every occurrence of a character followed by a space.

To specifically match a **full stop** followed by a **space**, you need to escape the dot by using `\`. So, using `r'\.\s'` will give the correct result: `['. ']`. This ensures that the dot is treated literally as a full stop rather than a wildcard character.

## Last But Not Least
Two other useful tokens are `^` and `$`:
- `^`: Matches the start of the string.
- `$`: Matches the end of the string.

```python
import re

print(re.findall(r'hello_\d+', 'hello_world hello_123'))
```
In this Python code, the result will be `['hello_123']`. If you want to check if the string starts with the pattern `hello_` followed by one or more digits, you should use the pattern `r'^hello_\d+'`.

Similarly, if you want to check if the string ends with `hello_` followed by one or more digits, you should use the pattern `r'hello_\d+$'`. The `^` ensures the string starts with the pattern, and the `$` ensures it ends with the pattern.

## Python Functions
Since this post focuses mainly on regular expressions, I’ve used the `re.findall()` function. However, you can also experiment with the following Python functions depending on the specific needs:
- `re.match()`: Checks for a match only at the start of the string.
- `re.search()`: Searches the entire string for the first match.
- `re.sub()`: Replaces occurrences of a pattern with a specified string.
- `re.split()`: Splits the string at each match of the pattern.

## Outroduction
As the title, **Regular Expression 101**, mastering regular expressions requires plenty of practice. I recommend experimenting with matching patterns like email addresses and ID numbers to strengthen your skills. In future posts, I’ll cover more complex patterns, such as password validation, to help you tackle even more challenging use cases.
