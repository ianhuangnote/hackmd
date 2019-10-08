---
tags: BookStudy, Go
title: Concurrency in Go - Chap 1
description:
---
###### tags: `BookStudy` `Go`

# Concurrency in Go: Chap 1

## An introduction to concurrency

:small_blue_diamond: Concurrency is an interesting word because it means different things to different people in our field.

:small_blue_diamond: Concurrency, you may have heard below words
- Asynchronous
- Parallel
- Threaded

:small_blue_diamond: When most people use the word **concurrent**, they’re usually referring to **a process that occurs simultaneously with one or more processes**

It is also usually implied that **all of these processes are making progress at about the same time**.

:small_blue_diamond: In this chapter:
 - Reasons concurrency became such an important topic in computer science
 - Why concurrency is difficult and warrants careful study
 - Go can make programs clearer and faster by using its concurrency primitives.

## Moore’s Law, Web Scale, and the Mess We’re In

:small_blue_diamond: Moore’s Law -> Amdahl’s law
:small_blue_diamond: Embarrassingly parallel -> Scale horizontally -> 2000s Cloud computing -> Web Scale

## Why Is Concurrency Hard?

:small_blue_diamond: Concurrent code is **notoriously** difficult to get right. 
:small_blue_diamond: It usually takes a few iterations to get it working as expected, and even then it’s not uncommon for **bugs to exist in code for years** before some change in timing (heavier disk utilization, more users logged into the system, etc.) causes a previously undiscovered bug to rear its head.

### Race Conditions

A race condition occurs when two or more operations must execute in the correct order, but the program has not been written so that this order is guaranteed to be maintained.

:small_blue_diamond: a.k.a Data Race

:small_blue_diamond: What's the result?
```go=
var data int
go func() {
    data++
}()
if data == 0 {
    fmt.Printf("the value is %v.\n", data)
}
```

:small_blue_diamond: Most of the time, data races are introduced because the developers are **thinking about the problem sequentially**.

:small_blue_diamond: What's the result?
```go=
var data int
go func() { data++ }()
time.Sleep(1*time.Second)
if data == 0 {
    fmt.Printf("the value is %v.\n" data)
}
```

:checkered_flag: **The takeaway**
 - Should always target logical correctness
 - Introducing sleeps into your code can be a handy way to debug concurrent programs, but they are not a solution

### Atomicity

When something is considered atomic, or to have the property of atomicity, this means that within :one: **the context that it is operating**, it is :two: **indivisible**, or :three: **uninterruptible**.

:small_blue_diamond: The first thing that’s very important is the word “context.” **Something may be atomic in one context, but not another**.

:small_blue_diamond: Context Scope:
- the context of your process :x: the context of the operating system
- the context of the operating system :x:  the context of your machine
-  the context of your machine :x: the context of your application

:small_blue_diamond: When thinking about atomicity, very often the first thing you need to do is to define the context, or scope, the operation will be considered to be atomic in.

:small_blue_diamond: Blizzard case

:small_blue_diamond: **Indivisible** and **Uninterruptible**. These terms mean that within the context you’ve defined, something that is atomic will happen in its entirety without anything happening in that context simultaneously.

:small_blue_diamond: Is it atomic?
```go
i++
```

:small_blue_diamond: Retrieve i, Increment i, and store i. Combining them does not necessarily produce a larger atomic operation.

:small_blue_diamond: Making the operation atomic is dependent on which context you’d like it to be atomic within. 

- If your context is a program with no concurrent processes, then this code is atomic within that context. 

- If your context is a goroutine that doesn’t expose i to other goroutines, then this code is atomic.

### Memory Access Synchronization

:small_blue_diamond: Let’s say we have a data race: two concurrent processes are attempting to access the same area of memory, and the way they are accessing the memory is not atomic.

:small_blue_diamond: Identify three Critical Section
```go=
var data int
go func() { data++}()
if data == 0 {
    fmt.Println("the value is 0.")
} else {
    fmt.Printf("the value is %v.\n", data)
}
```

:small_blue_diamond: Memory access synchronization
```go=
var memoryAccess sync.Mutex
var value int
go func() {
    memoryAccess.Lock()
    value++
    memoryAccess.Unlock()
}()
memoryAccess.Lock()
if value == 0 {
    fmt.Printf("the value is %v.\n", value)
} else {
    fmt.Printf("the value is %v.\n", value)
}
memoryAccess.Unlock()
```
:small_blue_diamond: Every time we perform one of these operations, our program pauses for a period of time. This brings up two questions:
 - Are my critical sections entered and exited repeatedly?
 - What size should my critical sections be?

### Deadlocks, Livelocks, and Starvation

#### Deadlocks

A deadlocked program is one in which all concurrent processes are waiting on one another. In this state, the program will never recover without outside
intervention.

```go=
type value struct {
    mu sync.Mutex
    value int
}
var wg sync.WaitGroup
printSum := func(v1, v2 *value) {
    defer wg.Done()
    v1.mu.Lock()
    defer v1.mu.Unlock()
    time.Sleep(2*time.Second)
    v2.mu.Lock()
    defer v2.mu.Unlock()
    fmt.Printf("sum=%v\n", v1.value + v2.value)
}
var a, b value
wg.Add(2)
go printSum(&a, &b)
go printSum(&b, &a)
wg.Wait()

//fatal error: all goroutines are asleep - deadlock!
```

![](https://i.imgur.com/v4lM6c6.png)

:small_blue_diamond: The Coffman Conditions are as follows:
- Mutual Exclusion
A concurrent process holds exclusive rights to a resource at any one time.
- Wait For Condition
A concurrent process must simultaneously hold a resource and be waiting for an additional resource.
- No Preemption
A resource held by a concurrent process can only be released by that process, so it fulfills this condition.
- Circular Wait
A concurrent process (P1) must be waiting on a chain of other concurrent processes (P2), which are in turn waiting on it (P1), so it fulfills this final condition too.

#### Livelock
Livelocks are programs that are actively performing concurrent operations, but these operations do nothing to move the state of the program forward.

```go=
Alice is trying to scoot: left right left right left right left right left right
Alice tosses her hands up in exasperation!
Barbara is trying to scoot: left right left right left right left right left right
Barbara tosses her hands up in exasperation!
```

:small_blue_diamond: Two or more concurrent processes attempting to prevent a deadlock without coordination

:small_blue_diamond: Livelocks are a subset of a larger set of problems called starvation.

#### Starvation
Starvation is any situation where a concurrent process cannot get all the resources it needs to perform work.

```go=
var wg sync.WaitGroup
var sharedLock sync.Mutex
const runtime = 1*time.Second
greedyWorker := func() {
    defer wg.Done()
    var count int
    for begin := time.Now(); time.Since(begin) <= runtime; {
        sharedLock.Lock()
        time.Sleep(3*time.Nanosecond)
        sharedLock.Unlock()
        count++
    }
    fmt.Printf("Greedy worker was able to execute %v work loops\n", count)
}
politeWorker := func() {
    defer wg.Done()
    var count int
    for begin := time.Now(); time.Since(begin) <= runtime; {
        sharedLock.Lock()
        time.Sleep(1*time.Nanosecond)
        sharedLock.Unlock()
        sharedLock.Lock()
        time.Sleep(1*time.Nanosecond)
        sharedLock.Unlock()
        sharedLock.Lock()
        time.Sleep(1*time.Nanosecond)
        sharedLock.Unlock()
        count++
    }
    fmt.Printf("Polite worker was able to execute %v work loops.\n", count)
}
wg.Add(2)
go greedyWorker()
go politeWorker()
wg.Wait()

//Polite worker was able to execute 289777 work loops.
//Greedy worker was able to execute 471287 work loops
```

:small_blue_diamond: Starvation is also coming from outside the Go process
:small_blue_diamond: Starvation can also apply to CPU, memory, file handles, database connections: any resource that must be shared is a candidate for starvation.

### Determining Concurrency Safety

:small_blue_diamond: The most difficult aspect of developing concurrent code, the thing that underlies all the other problems:  :man::woman:

:small_blue_diamond: Calculate Pi 1
```go=
// CalculatePi calculates digits of Pi between the begin and end
// place.
func CalculatePi(begin, end int64, pi *Pi)
```
- How do I do so with this function?
- Am I responsible for instantiating multiple concurrent invocations of this function?
- It looks like all instances of the function are going to be operating directly on the instance of Pi whose address I pass in; am I responsible for synchronizing access to that memory, or does the Pi type handle this for me?

:small_blue_diamond: CalculatePi 2
```go=
// CalculatePi calculates digits of Pi between the begin and end
// place.
//
// Internally, CalculatePi will create FLOOR((end-begin)/2) concurrent
// processes which recursively call CalculatePi. Synchronization of
// writes to pi are handled internally by the Pi struct.
func CalculatePi(begin, end int64, pi *Pi)
```
- Who is responsible for the concurrency?
- How is the problem space mapped onto concurrency primitives?
- Who is responsible for the synchronization?
 
:small_blue_diamond: Calculate Pi 3
```go=
func CalculatePi(begin, end int64) <-chan uint
```
## Simplicity in the Face of Complexity

With Go’s concurrency primitives, you can more safely and clearly express your concurrent algorithms. 

The runtime and communication difficulties we’ve discussed are by no means solved by Go, but they have been made significantly easier.


<style>

html, body, .ui-content {
    background-color: #333;
    color: #ddd;
}

body > .ui-infobar {
    display: none;
}

.ui-view-area > .ui-infobar {
    display: block;
}

.markdown-body h1,
.markdown-body h2,
.markdown-body h3,
.markdown-body h4,
.markdown-body h5,
.markdown-body h6 {
    color: #ddd;
}

.markdown-body h1,
.markdown-body h2 {
    border-bottom-color: #ffffff69;
}

.markdown-body h1 .octicon-link,
.markdown-body h2 .octicon-link,
.markdown-body h3 .octicon-link,
.markdown-body h4 .octicon-link,
.markdown-body h5 .octicon-link,
.markdown-body h6 .octicon-link {
    color: #fff;
}

.markdown-body img {
    background-color: transparent;
}

.ui-toc-dropdown .nav>.active:focus>a, .ui-toc-dropdown .nav>.active:hover>a, .ui-toc-dropdown .nav>.active>a {
    color: white;
    border-left: 2px solid white;
}

.expand-toggle:hover, 
.expand-toggle:focus, 
.back-to-top:hover, 
.back-to-top:focus, 
.go-to-bottom:hover, 
.go-to-bottom:focus {
    color: white;
}


.ui-toc-dropdown {
    background-color: #333;
}

.ui-toc-label.btn {
    background-color: #191919;
    color: white;
}

.ui-toc-dropdown .nav>li>a:focus, 
.ui-toc-dropdown .nav>li>a:hover {
    color: white;
    border-left: 1px solid white;
}

.markdown-body blockquote {
    color: #bcbcbc;
}

.markdown-body table tr {
    background-color: #5f5f5f;
}

.markdown-body table tr:nth-child(2n) {
    background-color: #4f4f4f;
}

.markdown-body code,
.markdown-body tt {
    color: #eee;
    background-color: rgba(230, 230, 230, 0.36);
}

a,
.open-files-container li.selected a {
    color: #5EB7E0;
}


</style>