# ``pwk``: **P**ython **W**ith **K**urly braces
**Making Python more one-liner-esque**


<br/>

## Motivation

We love Python. We love them bash one-liners. We want to do one-liners in Python.


Now, this is already possible even in many non-trivial cases:

* We can use, for example, the ternary operators (which however feel somewhat stilted especially if nested), or list comprehensions. 

* Some contructs (``try``/``except``) require more hacky workarounds -- like controlling your own block indentation via some ``bash -c '..'`` wrapper.

All of this reqires us to rewrite code snippets, and at least a modicum of mental effort we wanted to avoid with one-liners in the first place. 

``pwk`` allows to denote blocks in Python one-liners with kurly, née curly, braces. It generates corresponding multi-line Python with proper indentation on the fly and ``exec()``-utes the resulting code. You can thus write one-liners in pwk/Python like this:

    if "braces"=="bad": { print("Be gone!"); exit(99) } else: { print("Howdy!") }


<br/>
<br/>

## Usage

### HELLO WORLD

We feel compelled to start with:

    $ pwk 'print("Brace yourself, World!")'
    Brace yourself, World!



<br/>

### CONDITIONS

Let's look at the above example again:

    $ pwk 'if "braces"=="bad": { print("Be gone!"); exit(99) } else: { print("Howdy!") }'
    Howdy!

We can use the debug option ``-d`` to show the actual Python code that *would* be executed:

    $ pwk 'if "braces"=="bad": { print("Be gone!"); exit(99) } else: { print("Howdy!") }'  -d
    if "braces" == "bad" :
        print ( "Be gone!" )
        exit ( 99 )
    else :
        print ( "Howdy!" )


<br/>

### FUNCTIONS

We can define and use our own functions:

    $ pwk 'def s2i(s): { return int(s) } print(s2i("41")+1)'
    42

as per

    $ pwk 'def s2i(s): { return int(s) } print(s2i("41")+1)'  -d
    def s2i ( s ) :
        return int ( s )
    print ( s2i ( "41" ) + 1 )

Feel free to use imports (note: ``sys``, ``io``, ``os``, and ``tokenize`` are already imported and available):

    $ pwk 'import numpy as np; print(np.sin(3.14))'
    0.0015926529164868282


<br/>

### STANDARD INPUT & EXCEPTIONS

We can use ``pwk`` in an ``awk``-like way. Here, we list all subfolders in the root directory, and ignore exceptions on files not being directories:

    $ ls / | pwk 'for s in sys.stdin: { try: { print(os.listdir("/"+s.strip())) } except: pass }'

Because there is no code after the for-block, we can omit its braces:

    $ ls / | pwk 'for s in sys.stdin:   try: { print(os.listdir("/"+s.strip())) } except: pass'

(To the best of my knowledge, there is as of 2020 no purely pythonic way to write one-lined ``try``-blocks.)


</br>

### VARIABLES & THE ENVIRONMENT

We don't want to use bash variable replacement in our pwk-code string -- there are just to many special characters floating around. Instead, we can use named variables and lists of variables (TODO: typing via <varname:type>):

    $ pwk -v my_variable $USER  'print(my_variable)'

(Of course, use double quotes around environment variables containing spaces.)

You can use multiple variables. And the position of variable declarations doesn't matter -- you can also write:

    $ pwk 'print(s1+", "+s2)'  -v s1 $USER  -v s2 $SHELL
    martin, /bin/bash

The resulting code to be executed is, unsurprisingly:

    $ pwk 'print(s1+", "+s2)'  -v s1 $USER  -v s2 $SHELL  -d
    u = "martin"
    s = "/bin/bash"
    print ( u + ", " + s )


<br/>

### VARIABLE LISTS

``pkw`` supports variable lists with ``-V``:

    $ pwk 'print(ls1[0]); print(ls2[-1])'  -V ls1 a b c d  -V ls2 x y z
    a
    z

If lists precede the pwk-code, use ``-c``:

    $ pwk -V ls1 a b c d  -V ls2 x y z  -c 'print(ls1[0]); print(ls2[-1])'

Without the ``-c``, you'd get:

    $ pwk -V ls1 a b c d  -V ls2 x y z     'print(ls1[0]); print(ls2[-1])'  -d
    ls1 = ["a", "b", "c", "d"]
    ls2 = ["x", "y", "z", "print(ls1[0]); print(ls2[-1])"]

(This is not necessary for individual variables, as their argument count is known.)

Variable lists work nicely with bash wildcards:

    $ pwk 'print(my_list)'  -V my_list /bin/who*
    ['/bin/who', '/bin/whoami']

<br/>

### DICTIONARIES

Dictionaries should work (except, for now, dicts with dicts as values, but so be it):

    $ pwk 'if True:  { d={0:"foo", 1:"bar"} ; for i in range(2): { print(d[i]) } } else: exit(99)'
    foo
    bar
    $ pwk 'if False: { d={0:"foo", 1:"bar"} ; for i in range(2): { print(d[i]) } } else: exit(99)'
    $ echo $?
    99


<br/>
<br/>

## Background

### INSTALL

``pwk`` is a single-file app. Just download and ``chmod`` it, and you should be good to go.


<br/>

### HISTORY

* v0.1 -- 2020-11-22 -- Alpha release


<br/>

### METHOD

``pwk`` is itself a Python3 tool. It reformats the given pwk-line into into multi-line Python using ``tokeinze``. It discards open braces that follow a ``:``, which indents and starts a block. The corresponding closing brace de-indents. Other braces (like in dictionaries) are (hopefully) left alone.


<br/>

### THOUGHTS

Actually, I would really like to have the option to surround blocks with braces right in basic Python. I am, though, not on the edge of my seat waiting for that to happen (see also: ``from __future__ import braces``; [*thx, user OJFord*]).

My main reason is fairly prosaic: in business code, for example, heavy with assert-like checks and early exits/exceptions, I'd like to compress this "subordinate" code portion into as little horizontal space as possible in order to get it out of way and mind.

Methinks that the Python community abhors braces more than the vacuum. In fact, we'd only need a "close block" identifier -- possibly the ``°``-character? I find braces more elegant. While I love Python, as stated at the beginning, I also love C, in my view as elegant as can be.


<br/>

### CAVEAT & OUTLOOK

**This is alpha code!**

I cooked this up yesterday (2020-11-21), on WSL2/Ubuntu, and would fully expect bugs (it *does* seem to run on macOS). There are quite a few kinks I'd like to work on -- especially to give users earlier syntax errors (instead of passing invalid code to ``exec()``). Also, typed variables/variable lists are planned.


<br/>

### FEEDBACK & BUG REPORTS

Reach us at info@umlet.com.

Enjoy!

<br/>
<br/>

---
*GPL3, Martin Auer, 2020, in lockdown*
