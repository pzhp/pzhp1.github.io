---
layout: post
title:  "Concurrency pattern"
date:   2018-4-14 16:24:00
categories: C++,javascript,go
---

In this paper, we disucss the concurrency pattern changed from callback, promise/future, to async/await and channel. Many program language supoort these patterns partially or fully. We will study the key idea across C++, Javascript, Go etc and focus on what and why. 

# Callback
Before talked about callback, give an example about push/pull. in some extent, callback is push.
```C++
// main thread
auto result = sendTasktoWorkThread(task); // execute in other thread
doOtherthing();
auto value = result.get(); // [pull the result]
doRemainedthing(value);
```

```C++
// when task done, call callback in worker thread directly
// or send event to main thread which execute call callback then [push the result]
sendTasktoWorkThread(task, callback); 
doOtherthing();
eventloop();
```
Let's see an example involved register many callbacks.
```c++
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
## Prons and cons:
- callback hell as previous example show: 
  it's really counter-intuitive thing to register callback.
- lifecyle manage: 
  need keep the object involved in callback not destructed before call in lanugage not supporting garbage collect.
- executor/thread schedule:
  schedule the callback in original main thread, or in current worker thread. If run in current worker thread, should have additional mutex to avoid data race in the callback.
  
# Promise/future
[folly/fututre api](https://github.com/facebook/folly/blob/master/folly/futures/Future.h) 
[example](https://www.oschina.net/translate/futures-for-c-11-at-facebook)
```html
<html>
<head> 
<script>
function taskA() {
    console.log("Task A");
    throw new Error("throw Error Task A")
}
function taskB() {
    console.log("Task B");
}
function onRejected(error) {
    console.log("Catch Error: ", error);
}
function finalTask() {
    console.log("Final Task");
    throw new Error("throw Error Final task")
}

var promiseB = new Promise((resolve, reject) => reject(new Error('error msg')))
promiseB
.then(success => console.log('onfulfilled ', success), err => console.log('onRejected ', err))
.then(taskA)
.then(taskB)
.catch(onRejected)
.then(finalTask);
</script>
</head>
<body>
</body>
</html>
```

![folly/future](https://github.com/pzhp/pzhp.github.io/blob/master/images/promise_future.png)

## Pros and cons:
- chained then with arguments

# Async/await
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

## Prons and cons
- use sync logic to write async code

# Channel
``` Go
package main
import "fmt"
import "time"

func main() {
    c1 := make(chan string)
    go func() {
        c1 <- "ping from 1"
    }()

    c2 := make(chan int)
    go func() {
        c2 <- 2
    }()

    // time.Sleep(100 * time.Millisecond)
    // case is chosen randomly
    for i := 0; i < 2; i++ {
        select {
        case msg1 := <-c1:
            fmt.Println(msg1)
        case msg2 := <-c2:
            fmt.Println(msg2)
        case <-time.After(time.Second * 30):
            fmt.Println("timeOut")
            // default: // if no default, block
            //  fmt.Println("non block")
        }
    }
}
```

``` C++
// consider use c++11 implement previous code
std::future<std::string> f = std::async(std::launch::async, []() { 
	return dotask();
	})
std::string ret = f.get(); // block
// f.wait_for(); // support time out
```
C++ future only support set value once, but channel support tranfer data like a stream.

# Summary
![Outline](https://github.com/pzhp/pzhp.github.io/blob/master/images/concurrency_pattern.png)

remained issues: executor/thread schedule, exception etc.



