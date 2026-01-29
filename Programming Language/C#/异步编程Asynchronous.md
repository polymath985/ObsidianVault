---
tags:
  - CSharp
  - 异步编程
---

# C# 异步编程 (async/await)

异步编程是现代C#开发的核心特性，它允许程序在等待I/O操作或其他长时间运行的任务时不阻塞当前线程。

```csharp
// 异步方法的基本语法
public async Task<string> FetchDataAsync()
{
    using (var httpClient = new HttpClient())
    {
        // await关键字会异步等待操作完成
        var response = await httpClient.GetAsync("https://api.example.com/data");
        var content = await response.Content.ReadAsStringAsync();
        return content;
    }
}
```

## 常见概念

异步编程更像是对[[多线程Multi-Threading]]的封装，但它专注于I/O密集型操作的优化。

```csharp
// 传统同步方式 - 会阻塞线程
public string FetchDataSync()
{
    Thread.Sleep(2000); // 模拟耗时操作
    return "数据获取完成";
}

// 异步方式 - 不会阻塞调用线程
public async Task<string> FetchDataAsync()
{
    await Task.Delay(2000); // 模拟异步耗时操作
    return "数据获取完成";
}
```

### 异步执行流程

异步方法会在await启用时切换到另外一个线程等待，任务完成后借助同步上下文机制回到另一个线程继续执行。

```csharp
public async Task DemonstrateAsyncFlow()
{
    Console.WriteLine($"开始执行 - 线程ID: {Thread.CurrentThread.ManagedThreadId}");
    
    // 异步等待操作
    await Task.Delay(1000);
    
    Console.WriteLine($"异步操作完成 - 线程ID: {Thread.CurrentThread.ManagedThreadId}");
    
    // 可能在不同的线程上继续执行
    await SomeOtherAsyncOperation();
    
    Console.WriteLine($"所有操作完成 - 线程ID: {Thread.CurrentThread.ManagedThreadId}");
}

private async Task SomeOtherAsyncOperation()
{
    await Task.Run(() => 
    {
        Thread.Sleep(500);
        Console.WriteLine($"在后台线程执行 - 线程ID: {Thread.CurrentThread.ManagedThreadId}");
    });
}
```

这里通过输出线程ID可以观察到异步操作在不同线程间的切换过程。

### WPF中的异步编程

```csharp
// WPF按钮点击事件的异步处理，避免阻塞UI线程
private async void Button_Click(object sender, RoutedEventArgs e)
{
    // 禁用按钮防止重复点击
    var button = sender as Button;
    button.IsEnabled = false;
    
    try
    {
        // 执行异步操作，不会阻塞UI
        var result = await LongRunningOperationAsync();
        MessageBox.Show($"操作完成: {result}");
    }
    finally
    {
        // 重新启用按钮
        button.IsEnabled = true;
    }
}

private async Task<string> LongRunningOperationAsync()
{
    // 模拟长时间运行的操作
    await Task.Delay(3000);
    return "操作成功完成";
}
```
# 语法

使用Task便可返回一个Task，但是在没有Async的情况下不能使用await，直接调用仍然会阻塞主线程。
底层上，async关键字将Task封装成了一个状态机来管理异步执行流程。

## 基本异步方法定义

```csharp
// 阻塞式同步调用 - 不推荐
public Task<int> BadAsyncMethod()
{
    Thread.Sleep(2000); // 阻塞当前线程
    return Task.FromResult(42);
}

// 正确的异步方法定义
public async Task<int> GoodAsyncMethod()
{
    await Task.Delay(2000); // 不阻塞当前线程
    return 42;
}
```

使用`async`关键字标记函数，同时推荐在函数名后添加`Async`后缀，便可以在其中使用`await`

```csharp
// 推荐的异步方法命名和实现
public async Task<string> ProcessDataAsync(string input)
{
    // 可以使用await等待其他异步操作
    var processedData = await TransformDataAsync(input);
    var result = await SaveDataAsync(processedData);
    return result;
}

// 使用Task.Run()将同步方法封装为异步
public async Task<int> CalculateAsync(int[] numbers)
{
    // 将CPU密集型操作移到后台线程
    var result = await Task.Run(() => 
    {
        return numbers.Sum(x => x * x); // 计算平方和
    });
    return result;
}
```

## 异步主函数

C# 7.1开始支持异步Main方法：

```csharp
// 异步主函数的几种写法
public static async Task Main(string[] args)
{
    await SomeAsyncOperation();
    Console.WriteLine("程序执行完成");
}

// 带返回值的异步主函数
public static async Task<int> Main(string[] args)
{
    try
    {
        await SomeAsyncOperation();
        return 0; // 成功
    }
    catch (Exception ex)
    {
        Console.WriteLine($"错误: {ex.Message}");
        return 1; // 失败
    }
}
```

## Task泛型和返回值

Task的泛型相当于一个语法糖，将本应return的TaskT封装为一个结果，通过return就可以使外部调用时直接通过await拿到结果：

```cs
// Task<T>的使用示例
public async Task<User> GetUserAsync(int userId)
{
    var userData = await DatabaseService.GetUserDataAsync(userId);
    var user = new User 
    { 
        Id = userData.Id, 
        Name = userData.Name 
    };
    
    // 直接返回User对象，框架会自动包装成Task<User>
    return user;
}

// 调用时直接获取结果
var user = await GetUserAsync(123);
Console.WriteLine(user.Name); // 直接使用User对象
```
# 同步

无法通过Lock锁来实现异步方法的线程同步，由于异步通常会在另一个线程回归，在一个线程上锁，在另一个线程释放是不符合逻辑的。

```csharp
// 错误的做法 - 不能在异步方法中使用lock
private readonly object _lock = new object();

public async Task BadSyncMethod()
{
    lock (_lock) // 编译错误！不能在async方法中使用lock
    {
        await SomeAsyncOperation(); // await不能在lock内使用
    }
}

// 正确的做法 - 使用SemaphoreSlim进行异步同步
private readonly SemaphoreSlim _semaphore = new SemaphoreSlim(1, 1);

public async Task GoodSyncMethod()
{
    await _semaphore.WaitAsync(); // 异步等待信号量
    try
    {
        // 临界区代码
        await SomeAsyncOperation();
    }
    finally
    {
        _semaphore.Release(); // 释放信号量
    }
}

// 使用AsyncLock的更优雅的方式
public async Task ElegantSyncMethod()
{
    using (await _asyncLock.LockAsync())
    {
        // 临界区代码
        await SomeAsyncOperation();
    }
}
```

## 异步编程最佳实践

```csharp
// 1. 始终使用ConfigureAwait(false)在类库中
public async Task<string> LibraryMethodAsync()
{
    var result = await SomeAsyncOperation().ConfigureAwait(false);
    return result;
}

// 2. 避免async void，除非是事件处理程序
public async void Button_Click(object sender, EventArgs e) // 唯一允许的async void
{
    await ProcessClickAsync();
}

public async Task ProcessClickAsync() // 推荐的方式
{
    // 异步处理逻辑
}

// 3. 使用Task.WhenAll并行执行多个异步操作
public async Task<string[]> GetMultipleDataAsync()
{
    var task1 = GetDataAsync("source1");
    var task2 = GetDataAsync("source2");
    var task3 = GetDataAsync("source3");
    
    // 并行等待所有任务完成
    var results = await Task.WhenAll(task1, task2, task3);
    return results;
}
```

