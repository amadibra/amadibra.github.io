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


Memory Release

I have explained above how an object created by your program being store into 2 different memories. Your RAM however will definitely have limited amount of memory, if your program keep storing new object your RAM won't be able to take it anymore at some point. Thats why all program have their way to release memory back to your operating system. Here are some methods used by MRI to decide which object can be deleted, and release the memory used by the object into operational system.

Reference Counting

This is common method used by many programming language to do garbage collection. What this method does is to keep count for every object created by the program, everytime the reference added the count will be increase, everytime the reference removed the count will be decreased. By the time the reference count reach to 0, the GC will delete the object and release the memory back to operating system.
```
def queryData(db)
if __name__ == "__main__":
	dbString = "ahmadibrahim:1234@127.0.0.1:5432/blog_db"
	db = initializeDB(dbstring)
	BlogManager = initBlogManager(db)
```
you can see simple example from code snipet above. Just pretend that i have intializeDB() that will init db connection, also initBlogManager() that init new blogManager object that have interface to some blog functions, i do not have the real implementation of those 2 functions but lets just pretend its there.
You can see `db` object being created after the db initialize, and then it being used by `BlogManager` to init itself, hence the count of `db` object become 1
```
db = 1
```

```
if __name__ == "__main__":
	dbString = "ahmadibrahim:1234@127.0.0.1:5432/blog_db"
	db = initializeDB(dbstring)
	BlogManager = initBlogManager(db)
	ArticleManager = initArticleManager(db)
```
Now i added another object which is `ArticleManager` which need the `db` object too to be initiated, hence the count of `db` object become 2
```
if __name__ == "__main__":
	dbString = "ahmadibrahim:1234@127.0.0.1:5432/blog_db"
	db = initializeDB(dbstring)
	BlogManager = initBlogManager(db)
	ArticleManager = initArticleManager(db)
	del BlogManager
	del ArticleManger
```
in above example i deleted 2 objects that have reference to `db` object, so the reference count will decrease to 0, at this point Garbage collector will pickup `db` object, delete it and release the memory back to operating system.


Mark and Sweep
Another method of releasing memory, as the name explains this method consist of 2 steps, one is mark, the other is sweep. Mark is when MRI choose any objects to be deleted from memory, it decide which object to be deleted by tracing it connection to the root. You can imagine root as the main program, whenever an object created, root will extend a connection line into that object.
Root -> A -> B
Take an example above, Root created object A, then A created object B, the connections become like above. When garbage collection mark and sweep algorithm exectuted during this condition, no object will be delted. No memory will be released.
Now lets say, object got deleted by the program, the connection become like:
root B
B not getting deleted by the program, but doesnt have connection with root anymore, so when garbage collection mark and sweep executed during this condition, B will get deleted and the memory will be released. this method is fine and all, but 1 major thing that need to be noted about this is it "stop the world". Whenever this being executed, the program will be stopped, your program wont be able to process anything, and tracing all the connection from root to leaves everytime, taking so much time to be finished. So there's a need to improvise this methid which going to be explained in the next method.

Generational Mark and Sweep
This is the same method as i explained above, except with an improvement. The improvement is based on assumption (or fact) that "Object generally die young", so most of object generally not used anymore after first few usages / calls. If the object still have connection with the root after 1 generation, usually it means the object meant to be long-lived one and does not need to be deleted until the end of program live. So what is the improvement? everytime mark and sweep finished, any object survive during the sweep will be moved to "old generation". Whenever mark and sweep getting executed, it does not need to check any object inside "old generation", as explained before, any object survive usually means it need to live until the end of program. With this improvement mark and sweep that "stop the world" does not need to take so much time to check all the objects.


To be continued to part 2
