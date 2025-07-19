---
draft: false
date: 2024-04-01T17:00:00
authors: [Visin]
categories:
  - Pruebas
  - Blog
tags:
  - Blogs
---

# Post de pruebas

Este es un post de pruebas con MarkDown

<!-- more -->

## Code

### Ejemplos

#### Code for a specific language

Some more code with the `py` at the {--comienzo--} start:

``` py
import tensorflow as tf
def whatever()
```

#### With a title

``` py title="bubble_sort.py"
def bubble_sort(items):
    for i in range(len(items)):
        for j in range(len(items) - 1 - i):
            if items[j] > items[j + 1]:
                items[j], items[j + 1] = items[j + 1], items[j]
```

#### With line numbers

``` py linenums="1"
def bubble_sort(items):
    for i in range(len(items)):
        for j in range(len(items) - 1 - i):
            if items[j] > items[j + 1]:
                items[j], items[j + 1] = items[j + 1], items[j]
```

#### Highlighting lines

``` py hl_lines="2 3"
def bubble_sort(items):
    for i in range(len(items)):
        for j in range(len(items) - 1 - i):
            if items[j] > items[j + 1]:
                items[j], items[j + 1] = items[j + 1], items[j]
```

Some `code` goes here.(1)
{ .annotate }

1. :man_raising_hand: I'm an annotation!

## Icons and Emojs

:smile:

:fontawesome-regular-face-laugh-wink:

:fontawesome-regular-user:

:fontawesome-brands-twitter:{ .twitter }

:octicons-heart-fill-24:{ .heart }
