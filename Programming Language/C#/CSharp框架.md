---
tags:
  - CSharp
---

# C# 框架体系

C# 作为微软生态的核心编程语言，拥有丰富的框架生态系统：

## .NET Framework
- **.NET Framework 4.8** - Windows平台的传统框架
- **ASP.NET Web Forms** - 传统的Web开发模式
- **WPF (Windows Presentation Foundation)** - Windows桌面应用开发
- **WinForms** - Windows窗体应用程序

## .NET Core / .NET 5+
- **.NET 6/7/8** - 跨平台的现代.NET实现
- **ASP.NET Core** - 高性能的Web框架
- **Blazor** - 使用C#构建交互式Web UI
- **MAUI** - 跨平台移动和桌面应用开发

## 主要应用框架

### Web开发
```csharp
// ASP.NET Core Web API 示例
[ApiController]
[Route("api/[controller]")]
public class UsersController : ControllerBase
{
    [HttpGet]
    public async Task<ActionResult<IEnumerable<User>>> GetUsers()
    {
        var users = await _userService.GetAllUsersAsync();
        return Ok(users);
    }
    
    [HttpPost]
    public async Task<ActionResult<User>> CreateUser(CreateUserRequest request)
    {
        var user = await _userService.CreateUserAsync(request);
        return CreatedAtAction(nameof(GetUser), new { id = user.Id }, user);
    }
}
```

### 桌面应用
```csharp
// WPF 应用程序示例
public partial class MainWindow : Window
{
    public MainWindow()
    {
        InitializeComponent();
        DataContext = new MainViewModel();
    }
}

// MVVM 模式的 ViewModel
public class MainViewModel : INotifyPropertyChanged
{
    private string _title = "C# Desktop App";
    
    public string Title
    {
        get => _title;
        set
        {
            _title = value;
            OnPropertyChanged();
        }
    }
    
    public ICommand SaveCommand { get; }
    
    public event PropertyChangedEventHandler PropertyChanged;
    
    protected virtual void OnPropertyChanged([CallerMemberName] string propertyName = null)
    {
        PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
    }
}
```

## 微服务和云原生
- **gRPC** - 高性能RPC框架
- **Docker支持** - 容器化部署
- **Kubernetes集成** - 容器编排
- **Azure Functions** - 无服务器计算

```csharp
// gRPC 服务示例
public class GreeterService : Greeter.GreeterBase
{
    public override Task<HelloReply> SayHello(HelloRequest request, ServerCallContext context)
    {
        return Task.FromResult(new HelloReply
        {
            Message = "Hello " + request.Name
        });
    }
}
```

## C# 底层运行原理

### JIT 编译器 (Just-In-Time Compiler)

C# 采用了独特的两阶段编译模式：

1. **编译时**：C# 源码 → 中间语言(IL/MSIL)
2. **运行时**：IL → 机器码 (通过JIT编译器)

```csharp
// C# 源码示例
public class Calculator
{
    public int Add(int a, int b)
    {
        return a + b;
    }
}

/* 编译后的IL代码(简化版):
.method public hidebysig instance int32 Add(int32 a, int32 b) cil managed
{
    .maxstack 2
    ldarg.1    // 加载参数a
    ldarg.2    // 加载参数b  
    add        // 执行加法
    ret        // 返回结果
}
*/
```

#### JIT 编译过程

```csharp
// JIT编译的触发时机
public static void Main()
{
    var calc = new Calculator();
    
    // 第一次调用Add方法时，JIT编译器将IL编译为机器码
    int result1 = calc.Add(5, 3); // JIT编译发生
    1
    // 后续调用直接执行已编译的机器码，速度更快
    int result2 = calc.Add(10, 20); // 直接执行机器码
}
```

### CLR (Common Language Runtime) 架构

```csharp
// CLR管理的内存区域示例
public class MemoryDemo
{
    // 静态字段存储在方法区
    private static string StaticField = "存储在方法区";
    
    // 实例字段存储在堆上
    private string InstanceField = "存储在托管堆";
    
    public void MethodDemo()
    {
        // 值类型局部变量存储在栈上
        int localInt = 42;
        
        // 引用类型在栈上存储引用，在堆上存储对象
        string localString = "引用存储在栈，对象存储在堆";
        
        // 装箱：值类型 → 引用类型（在堆上创建对象）
        object boxedInt = localInt;
        
        // 拆箱：引用类型 → 值类型
        int unboxedInt = (int)boxedInt;
    }
}
```

### 垃圾回收机制 (Garbage Collection)

```csharp
// GC分代回收示例
public class GCDemo
{
    public static void DemonstrateGC()
    {
        // 创建大量临时对象（第0代）
        for (int i = 0; i < 1000; i++)
        {
            var temp = new byte[1024]; // 很快成为垃圾
        }
        
        // 手动触发GC（通常不建议）
        GC.Collect(0); // 只回收第0代
        GC.Collect(); // 回收所有代
        
        // 检查GC信息
        Console.WriteLine($"第0代回收次数: {GC.CollectionCount(0)}");
        Console.WriteLine($"第1代回收次数: {GC.CollectionCount(1)}");
        Console.WriteLine($"第2代回收次数: {GC.CollectionCount(2)}");
        
        // 查看内存使用
        long memoryBefore = GC.GetTotalMemory(false);
        GC.Collect();
        GC.WaitForPendingFinalizers();
        long memoryAfter = GC.GetTotalMemory(true);
        
        Console.WriteLine($"GC前内存: {memoryBefore} bytes");
        Console.WriteLine($"GC后内存: {memoryAfter} bytes");
    }
}
```

### AOT 编译 (Ahead-of-Time)

.NET 现在也支持AOT编译，直接生成机器码：

```csharp
// 发布为AOT的配置 (在.csproj文件中)
/*
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net8.0</TargetFramework>
    <PublishAot>true</PublishAot>
    <InvariantGlobalization>true</InvariantGlobalization>
  </PropertyGroup>
</Project>
*/

// AOT优化的代码写法
[MethodImpl(MethodImplOptions.AggressiveInlining)]
public static int FastAdd(int a, int b) => a + b;

// 使用源生成器避免反射（AOT友好）
[JsonSerializable(typeof(Person))]
public partial class PersonJsonContext : JsonSerializerContext { }

public class Person
{
    public string Name { get; set; }
    public int Age { get; set; }
}
```

### IL代码分析工具

```csharp
// 可以使用ildasm.exe或dotPeek查看IL代码
public class ILDemo
{
    // 简单方法的IL分析
    public int SimpleMethod(int x)
    {
        return x * 2 + 1;
    }
    
    // 属性的IL实现
    private int _value;
    public int Value
    {
        get => _value;        // 编译为get_Value()方法
        set => _value = value; // 编译为set_Value(int value)方法
    }
    
    // 泛型方法的IL特点
    public T GenericMethod<T>(T input) where T : class
    {
        return input ?? default(T);
    }
}
```

### 性能优化原理

```csharp
public class PerformanceDemo
{
    // 值类型避免堆分配
    public struct Point // 结构体存储在栈上，性能更好
    {
        public int X { get; set; }
        public int Y { get; set; }
    }
    
    // Span<T>和Memory<T>避免不必要的分配
    public static void EfficientStringProcessing(ReadOnlySpan<char> input)
    {
        // 零分配的字符串处理
        foreach (char c in input)
        {
            // 处理字符，不创建新字符串
        }
    }
    
    // 对象池模式重用对象
    private static readonly ObjectPool<StringBuilder> StringBuilderPool 
        = new DefaultObjectPool<StringBuilder>(new StringBuilderPooledObjectPolicy());
    
    public static string BuildString(string[] parts)
    {
        var sb = StringBuilderPool.Get();
        try
        {
            foreach (var part in parts)
                sb.Append(part);
            return sb.ToString();
        }
        finally
        {
            StringBuilderPool.Return(sb);
        }
    }
}
```