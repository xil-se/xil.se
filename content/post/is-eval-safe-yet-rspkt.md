+++
authors = "rspkt"
categories = [ "security" ]
date = "2017-01-14T16:00:00+01:00"
description = ""
title = "Is eval safe yet? Arbitrary code execution in Python"
+++

TL;DR
-----

Found use of `eval` in the Python `gettext` module. Bypassed weak security
checks. Gained arbitrary code execution.

Background
----------

In [_Beyond PEP 8_](https://youtu.be/wf-BqAjZb8M?t=2917), an excellent talk by
Raymond Hettinger, he jokingly comments that `namedtouple` is implemented using
`eval` ([or `exec`, to be exact](https://github.com/python/cpython/blob/master/Lib/collections/__init__.py#L303)),
and that it's defensible given the right circumstances. While this is sometimes
true, I figured I'd clone the cpython repository and grep for any usages of
`eval`-like functions.

What caught my attention was the function `c2py` in the `gettext` module. The
`gettext` module contains bindings for the [`gettext` system](https://www.gnu.org/software/gettext/),
which is an internationalization and localization system. One of the
responsibilities of `gettext` is to convert between plural forms in different
languages. The plural forms for a language are defined using a
[C-like DSL](https://www.gnu.org/savannah-checkouts/gnu/gettext/manual/html_node/Plural-forms.html).
The DSL is basically a nested ternary expression with different enumerated
outcomes, one for each plural form of the specific language. The variable in the
ternary expression is always named `n`, and represents the count for which a
plural form should be returned.

For English, the `gettext` plural form is written as:

```
n != 1
```

The above expression will evaluate to either plural form 0 when `n` is singular,
or 1 when plural. For other languages, several plural forms exist. Take for
example [Russian](http://docs.translatehouse.org/projects/localization-guide/en/latest/l10n/pluralforms.html):

```
n % 10 == 1 && n % 100 != 11 ? 0 : n % 10 >= 2 && n % 10 <= 4 && (n % 100 < 10 || n % 100 >= 20) ? 1 : 2
```


Eval to the rescue
------------------

Due to the expressiveness required by this language, writing a parser that
evaluates these plural forms is not trivial. We could however try to translate
these C-like expressions to Python and throw `eval` at it. The Russian plural
form rewritten in Python would look like this:

```
(0 if n % 10 == 1 and n % 100 != 11 else (1 if n % 10 >= 2 and n % 10 <= 4 and (n % 100 < 10 or n % 100 >= 20) else 2))
```

[This is exactly what was done in `c2py` in the `gettext` module](https://github.com/python/cpython/blob/69f0c06c3965ace34ccc8b9a932a8bfd842a52da/Lib/gettext.py#L63).
Just like the name of the function suggests, the C-like syntax is converted to
Python, and is subsequently evaluated to a lambda function with `eval`:

```
eval('lambda n: int(%s)' % plural)
```

A valid call to c2py could look something like this:

```
apples = ['Cox Orange', 'Granny Smith']
["apple", "apples"][gettext.c2py('n != 1')(len(apples))]

"apples"
```


Security
--------

No one in their right mind would call `eval` on user input without having the
security checks necessary in place. The security checks implemented in `c2py`
makes sure that `n` is the only allowed variable in the plural expressions. It
also does some general input validation, like checking that parentheses are
balanced.

My first discovery here was that the validation didn't prevent expressions
that were using `n` as a function. As long as there is a token that the Python
`tokenize` module classifies as a `NAME` token with the name `n`, the security
checks will pass. The below code snippet successfully spawned a shell:

```
gettext.c2py('n()')(lambda: os.system('sh'))
```

This isn't really a high risk issue, since the input to the lambda function
most likely will be an integer given by some count-logic, and not user input.


Exploitation
------------

Not being able to take the function-call bug any further, I kept investigating
ways in which I could break the security checks. After a lot of trial-and-error,
I found a way to confuse the Python tokenizer. If our entire input is
interpreted as a string, the input should pass the security checks. Furthermore,
it doesn't even have to be a valid string, since the security checks pass even
though the tokenizer returns a result with `ERRORTOKEN`. The following input
string will pass the security checks but break during evaluation:

```
gettext.c2py('"eval(foo) ""')
```

Next on the agenda is to make sure the invalid string we entered is actually
interpreted as a valid expression. Since we've bypassed the tokenizer, the
code that translates and builds the Python expression will assume that all
parentheses are matched. If we insert a left parentheses after the first double
quote, and some boolean operator before the empty string, we'll get a valid
expression:

```
gettext.c2py('"(eval(foo) && ""')(0)

----> 1 gettext.c2py('"(eval(foo) && ""')(0)
   gettext.pyc in <lambda>(n)
   NameError: global name 'foo' is not defined
```

From here, spawning a shell is trivial. The `c2py` module imports the `os`
module, so all we need to do to get full access to the host system is

```
gettext.c2py('"(os.system(\'sh\') & ""')(0)

$
```


Enter Python 3.7
----------------

Giving the tokenizer-based security a second though, I wondered how it would
react to the [new literal string interpolation](https://www.python.org/dev/peps/pep-0498/)
introduced in Python 3.7. (Not so) surprisingly, this string gets no special
attention by the tokenizer, and calling `c2py` with an interpolated string just
works:

```
gettext.c2py('f"{os.system(\'sh\')}"')(0)

$
```


Conclusion
----------

This bug was first disclosed to security@python.org, and later on added to the
[Python issue tracker](http://bugs.python.org/issue28563) due to its low-risk
nature. Generally these plural forms are specified in
[PO files](https://www.gnu.org/software/gettext/manual/html_node/PO-Files.html)
which is an unlikely attack vector.

The issue was resolved by implementing an actual parser for the `gettext` plural
form language.
