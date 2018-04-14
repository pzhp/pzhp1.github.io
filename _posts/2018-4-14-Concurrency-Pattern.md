---
layout: post
title:  "Concurrency pattern"
date:   2018-4-14 16:24:00
categories: C++,javascript,go
---

Concurrency is not only about execute task by multi-thread/coroutines, but also about execute task by event loop (javascript) or select/epoll function. In this paper, we disucss the concurrency pattern changed from callback, promise/future, to async/await and channel.

# Callback
Before talked about callback, give an example about poll.
```C++
// main thread
auto result = sendTasktoWorkThread(task); // execute in other thread
doOtherthing();
auto value = result.get(); // block
doRemainedthing(value);
```

```C++
// when task done, call callback in worker thread directly
// or send event to main thread which execute call callback then 
sendTasktoWorkThread(task, callback); 
doOtherthing();
eventloop();


// callback hell
#include <iostream>

#define DEFINE_STEP_N(n) \
template <typename F> \
void step##n(F f) { \
    std::cout << n << std::endl; \
    f(); \
}

DEFINE_STEP_N(1)
DEFINE_STEP_N(2)
DEFINE_STEP_N(3)
DEFINE_STEP_N(4)
DEFINE_STEP_N(5)

void fun() {
    std::cout << "last" << std::endl;
}

int main() {
   step1([]{
        step2([]{
            step3([]{
                step4([] {
                    step5(fun);
                });
            });
        });
   });
}

// output
1
2
3
4
5
last 
```

Prons:
- callback hell
- lifecyle manage
- executor/thread manage


# Promise/future
[folly/fututre api](https://github.com/facebook/folly/blob/master/folly/futures/Future.h) 
[example](https://www.oschina.net/translate/futures-for-c-11-at-facebook)

![folly/future](https://github.com/pzhp/pzhp.github.io/blob/master/images/promise_future.png)

## Cons:
- chained then with arguments

# Async/await
## Case study
[javascript promise and async/await](https://segmentfault.com/a/1190000007535316)
```javascript
function takeLongTime(n) {
    return new Promise(resolve => {
        // setTimeout mock time consuming task.
        // after n, will call the funtion
        setTimeout(() => resolve(n + 200), n);
    });
}

function step1(n) {
    console.log(`step1 with ${n}`);
    return takeLongTime(n);
}

function step2(n) {
    console.log(`step2 with ${n}`);
    return takeLongTime(n);
}

function step3(n) {
    console.log(`step3 with ${n}`);
    return takeLongTime(n);
}

// serialize execute: step1 => step2 => step3 
function doIt() {
    console.time("doIt");
    const time1 = 600;
    step1(time1)
        .then(time2 => step2(time2))
        .then(time3 => step3(time3))
        .then(result => {
            console.log(`result is ${result}`);
            console.timeEnd("doIt");
        });
}

// doIt();

console.log("................")

async function doItByAwait() {
    console.time("doIt");
    const time1 = 300;
	console.log("before step1")
    // block util step1() done
    const time2 = await step1(time1);
	console.log("before step2")
    const time3 = await step2(time2);
	console.log("before step3")
    const result = await step3(time3);
	console.log("before end")
    console.log(`result is ${result}`);
    console.timeEnd("doIt");
}

doItByAwait();
```
[async & await 的前世今生: C#](https://www.cnblogs.com/qixuejia/p/5740508.html)

## Understand from [continuation](https://en.wikipedia.org/wiki/Continuation)
```c++
step1(t1)
step2(t2)
step3(t3)

// transform

step1(t1, context1) // context1 is a context including next cod

```
## Pros
- use sync logic to write async code

# Channel

# Summary
![Outline](https://github.com/pzhp/pzhp.github.io/blob/master/images/concurrency_pattern.png)

remained issues: executor/thread schedule, exception etc.



