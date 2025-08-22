---
tags:
  - CSharp
  - 异步编程
---
![[Pasted image 20250415125059.png]]

## 常见概念

异步编程更像是对[[多线程Multi-Threading]]的封装

![[Pasted image 20250415125329.png]]


![[60553212F0FC598356AF94B6E0059E33.png]]
异步方法会在await启用时切换到另外一个线程等待，任务完成后借助同步上下文机制回到另一个线程继续执行。

![[1A0712C6959B7FC0DC2B3444DBB6E9A8.jpg]]
![[7BDCDD1879D31FEE973A4B3B16E04671.png]]
这里通过helper类提供的输出线程id方法监视线程，可以看到结果。

![[843754690B41F8CC4A8F152E110C1448.jpg]]
用于WPF开发过程中的按钮点击，避免阻塞UI线程
# 语法

使用Task便可返回一个Task，但是在没有Async的情况下不能使用await，直接调用仍然会阻塞主线程
底层上，async关键字似乎将Task封装成了一个状态机

![[Pasted image 20250422205803.png]]

![[Pasted image 20250422210107.png]]
*图：阻塞式调用

使用`async`关键字标记函数，同时推荐标记函数名，便可以在其中使用`await`

![[Pasted image 20250422210437.png]]

![[Pasted image 20250422210601.png]]
*图：使用Task.Run()将同步方法封装

![[Pasted image 20250422212545.png]]

*图：主函数也能async异步调用

Task的泛型相当于一个语法糖，将本应return的Task<>封装为一个结果，通过return 就可以使外部调用时直接通过await拿到结果

![[Pasted image 20250422211105.png]]

# 同步

无法通过Lock锁来实现线程同步，由于异步通常会在另一个线程回归，在一个线程上锁，再另一个线程释放是不符合逻辑的

![[Pasted image 20250422212126.png]]

