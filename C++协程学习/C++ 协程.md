# 协程概览
```cpp
#include <coroutine>
#include <future>
#include <iostream>
#include <thread>
#include <unistd.h>

// 协程结果需要实现相关的trait, 也就是这个promise_type
struct Result {
  struct promise_type {
    std::suspend_never initial_suspend() { return {}; }
    std::suspend_never final_suspend() noexcept { return {}; }
    Result get_return_object() {
      std::cout << "create result" << std::endl;
      return {};
    }

    void return_void() {} //有值返回实现return_value
    
    void unhandled_exception() {
      std::cout << "unhandled exception" << std::endl;
      // exception_ = std::current_exception();
    }
  };
};

// 构造协程体，需要实现await_ready, await_suspend, await_resume, 三个接口
struct Awaiter {
  int value;
  bool await_ready() { return false; }

  //await_suspend 返回值
  // void or true, 表示当前协程挂起后会将执行权归还
  // false， 恢复当前协程
  // 别的 coroutine_handle对象，返回的对象的协程被恢复执行
  // 抛异常， 当前协程恢复并抛出异常
  
  void await_suspend(std::coroutine_handle<> coroutine_handle) {
    std::async([=]() {
      using namespace std::chrono_literals;
      std::this_thread::sleep_for(1s);
      coroutine_handle.resume();
    });
  }
  
  int await_resume() { return value; }
};

Result Coroutine();
int main() { Coroutine(); }
Result Coroutine() {
  std::cout << 1 << std::endl;
  std::cout << co_await Awaiter{.value = 1000} << std::endl;
  std::cout << 2 << std::endl;
  std::cout << 3 << std::endl;
  co_await std::suspend_always{};
  std::cout << 4 << std::endl;
};
```

```c
#include<stdio.h>
printf("Hello, world");
```


```java
System.out.println("Hello, world");
```