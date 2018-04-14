# callback


# promise/future

folly/future:
``` C++ 
[future api](https://github.com/facebook/folly/blob/master/folly/futures/Future.h)
// group：value, result(Try<T>), isReady, hasValue/Exception, poll(Optional<Try<T>>), raise, cancel
// group: via, then, onError...
// group: timeout
// group: combine: filter/reduce
// group: get
inline Future<T> via(
    Executor* executor,
    int8_t priority = Executor::MID_PRI) &;

template <typename F, typename R = futures::detail::callableResult<T, F>>
typename R::Return then(F&& func);

template <class F>
typename std::enable_if<
    !futures::detail::callableWith<F, exception_wrapper>::value &&
        !futures::detail::callableWith<F, exception_wrapper&>::value &&
        !futures::detail::Extract<F>::ReturnsFuture::value,
    Future<T>>::type
onError(F&& func);

// func is always executed 
template <class F>
Future<T> ensure(F&& func);

template <class F>
Future<T> onTimeout(Duration, F&& func, Timekeeper* = nullptr);

/// Throw the given exception if this Future does not complete within the
/// given duration from now. The optional Timeekeeper is as with
/// futures::sleep().
template <class E>
Future<T> within(Duration, E exception, Timekeeper* = nullptr);

//
Future<T> delayed(Duration, Timekeeper* = nullptr);

T get();
T get(Duration dur);
Future<T>& wait(Duration) &;

/*Group*/
template <class F>
Future<T> filter(F&& predicate);

/// Like reduce, but works on a Future<std::vector<T / Try<T>>>, for example
/// the result of collect or collectAll
template <class I, class F>
Future<I> reduce(I&& initial, F&& func);

```
![asf](https://github.com/pzhp/pzhp.github.io/blob/master/images/promise_future.png)
# async/await


# channel
