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
