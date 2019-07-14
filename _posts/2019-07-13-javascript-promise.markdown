---
layout: post
title:  "[SE][JAVASCRIPT] Promise in JS, is it necessary?"
date:   2019-07-13 17:30:00 +0700
categories: Software Engineering
---

In this post i want to discuss about promise concept in javascript,
i'll start with describing what promise is and continue with discussion on 
how to use and will be closed with conclusion is this concept necessary or not.

First of all lets take a look on what is promise in javascript

according to `https://developer.mozilla.org`
> The Promise object is used for deferred and asynchronous computations.
> A Promise represents an operation that hasn't completed yet,
> but is expected in the future.

So promise is used to handle asynchronous computation in javascript.
Its different concept from XHR or AJAX in JQuery but its tightly correlated.
Simply you can translate promise as a callback to asynchronous process
such as XHR or AJAX. Lets move to how to use it so that we can have better picture.


Lets say i want to change my page title 3 seconds after user visit my page.
I'll use `setTimeout` to wait 3 seconds as this is simplest asynchronous
process that we can try in JS.

The code will be like this:
```javascript
var title = document.querySelector('title');
setTimeout(() => {
  title.innerHTML = 'Changed Title';
}, 3000);
```

another thing is i want to give alert message if the title changed,
in this case we can make use of promise to fulfill this problem.
We can wrap above function inside of promise and use those promise object
to set alert message.

The code will be like this:
```javascript
var tryPromise = new Promise((resolve, reject) => {
  var title = document.querySelector('title');
  setTimeout(() => {
    title.innerHTML = 'Changed Title';
    resolve();
  }, 3000);
});

tryPromise.then(() => {
  alert('title changed');
});
```

As you can see in snippet above, we can use promise object to set a callback
for asynchronous process inside the promise. In the case above is an example
of successfull asynchronous process where we use `resolve()` inside promise function 
and `then()` when set the callback, aside from that we also can set rejected
path by using `reject()` inside promise function and `catch()` to set callback.

I'll assume we already have better understanding on promise after the example.
So my question is do we really need that?

Lets take above case for example, can we achieve that without promise?
Of course, simplest way is just to move our callback inside setTimeout
```javascript
  var title = document.querySelector('title');
  setTimeout(() => {
    title.innerHTML = 'Changed Title';
    alert('title changed');
  }, 3000);
```

So promise can (almost) always be replaced by putting our desired function
inside callback, we can use `if else` or `try catch` to replace promise
`reject` and `finally` capability.

But, you will face the necessary of using promise when you building large
codebase and want to follow SOLID principle. It can make your code lookmore
clean and structured, i'll explain about this in the next post.

So in conclusion, we dont really need promise for simple asynchronous callback
process, XHR and AJAX have capability to do it without promise, but it will
be very useful when you build large JS codebase with many file / class
and you need to set a callback for asynchronous process across several class.
In this case promise will help us tt create instead of one function calling
several class for callback, we can pass promise as depedency injection
to keep our codebase clean.
