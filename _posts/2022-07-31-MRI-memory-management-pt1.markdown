---
layout: post
title:  "[SE] MRI memory management part 1"
date:   2022-07-31 16:34:00 +0800
categories: Software Engineering
---

RAM, must be common term you often heard, whether you software engineer or not, as long as you have used computer regularly, this word should at least came to your ears once. Most people know RAM is coupled with the performance of your computer, the bigger the better, the more an application use RAM the heavier the application. Now what does the application use the RAM for? why do some application use more RAM than the others? as a developer how to make your application RAM efficient? here i want to take a look a bit about how an application use RAM, as i'm not yet an expert about this ofcourse its gonna be just a surface knowledge around it, you can try to find and learn yourself more details from other sources.

What is RAM? its a short for Random Access Memory, what does it do to application? it allocates blocks of memory to an application, how big will it allocate? it will allocates as much as what application ask, unless limited by the OS. How do application utilise memory given by RAM? it depends on the application, different application have different way of utilising memory. Can you explain in details how they utilise it? ofcourse it is not gonna possible to explain how all type of application / program utilising the memory, because its gonna be too long article, also i do not have much knowledge of how most of the application / program utilising it (this is the main reason actually :)). So in this article i will only explain about memory management on MRI (Matz's Ruby Interpreter), most common ruby distrubution, how they allocate and free up memory given by RAM.

Memory management can be separated into 3 parts, allocating, storing, releasing. I will start this article with the middle part which is storing.

Memory Store

whether you create a program in ruby, python, java, or any program there must be some way your proram manage to store data into memory given by RAM, and most of them use the same methodology. The are 2 type of memory stores, 1 is stack memory, the other is heap memory.

Stack Memory
What is stack memory? stack memory is where the program store function call sequence and all of its local variable. e.g you create program 
```
def sum10(a)
	ten = 10
	return a + ten
end

def process(a,b,c)
	return a + b + sum10(c)
end

process(1,2,3)
```
see in above code, I created 2 functions, 'sum10' and 'process' and in the main program i'm calling `process(1,2,3)`, its very simple program, and I believe you guys already know the call stack happening in that program, it will be like:
`sum10(3) creating variable ten and return a value`
`process(1,2,3) calling sum10(3)`
`main calling process(1,2,3)`
see the stack above, it created by appending any function call to top of the stack, and as how stack works, it will executed from top to bottom. What it means to memory? stack memory will be used everytime a new function called, a block of memory will be used to store the function call as well as the local variable inside the function. But the moment the function executed, that 1 block of stack memory will be free up immediately.
`sum10(3) creating variable ten and return a value` (excetued)
`process(1,2,3) calling sum10(3)` (executing)
`main calling process(1,2,3)` (waiting)
so when the first execution has been finished (see stack above as current state) the top of the stack has been removed and the memory 'borrowed' for that stack's block will be freed. For now thats the only thing i can explain about stack memory, i'll put up more details when i learn more about this. So next to cover is heap memory

Heap Memory
On the other side of stack memory, there's also heap memory to store objects created by your program. The difference is instead of storing function call stack and its local variable, heap memory will long lived objects, such as long running function (e.g web server) or global variable. Hence, in general heap memory usually will be having more size compare to stack memory, because many of the object that stored inside are long live one. (code example will ve attached soon)

