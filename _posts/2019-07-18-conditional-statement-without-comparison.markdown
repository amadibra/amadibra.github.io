---
layout: post
title:  "[SE] Conditional statement without comparison, simple but risky"
date:   2019-07-18 20:30:00 +0700
categories: Software Engineering
---

In this post i want to discuss about conditional statement specifically the one without comparison.

as we all know conditional statement is one of very basic logic that oftenly used
in many situation, one of the syntax is like this
```javascript
if(somevariable == somevalue) {
  // doing something
}
```

This logic usually used to check whether some variable value is equal to
expected value or not, thus it also possible to use this to check
whether some variable is exists, initialized, defined, or not.
for example in javascript if you want to check whether some variable
defined or not, you can write something like this:
```javascript
if(somevariable != undefined){
  // do something
}
```

Even with most of the high level programming language you can
just write something like this
```javascript
if(somevariable){
  //do something
}
```

However doing something like above is not recommended, why?
depends on what programming language you use, but this statement not only reject
undefined or null value, it also reject default value of data type.
For example if you wrote code like above example,
in javascript if `somevarible` value is `0` (default value of int) it will rejected
by those condition therefor it would not process command inside statement, same
thing happened if `somevariable` value is empty string `''`.
Sometimes developer not aware about this case where most of the time it will
produce some bug to their program.

So here i list some of programming language behaviour with conditional statement
without comparison.

Javascript:
```javascript
if(somevariable) {
  // undefined
  // null
  // 0
  // ''
  // will not process to this
}
```

Python:
```python
if somevariable:
  # None
  # 0
  # ''
  # []
  # {}
  # will not process to this
```

So be careful with the usage of this syntax, know your programming language behaviour
and you'll be safe.
