---
tags:
  - CSharp
  - 设计模式
aliases:
  - 单例模式
---

# 定义

在C#中，没有类似于"全局变量"的声明，由此，单例模式应运而生

单例模式是一个类的静态实例封装，可以被全局随时访问，管理被管理的对象

全局唯一性，全局可访问性

**`Instance` `Shared` `Default`**

# 使用

## C Sharp

### 传统的双重检查锁定单例

在C# 6.0之前，我们通过双重判断加锁的形式实现线程安全的单例：

```csharp
// 传统的双重检查锁定单例
public sealed class TraditionalSingleton
{
    private static TraditionalSingleton instance;
    private static readonly object lockObject = new object();
    
    private TraditionalSingleton()
    {
        // 私有构造函数
    }
    
    public static TraditionalSingleton Instance
    {
        get
        {
            if (instance == null)  // 第一次检查
            {
                lock (lockObject)  // 加锁确保线程安全
                {
                    if (instance == null)  // 第二次检查
                    {
                        instance = new TraditionalSingleton();
                    }
                }
            }
            return instance;
        }
    }
    
    public void DoSomething()
    {
        Console.WriteLine("执行某些操作");
    }
}
```

在外部第一次尝试获取实例时，`Instance`会首先判断是否为空，若是，则进入锁，锁的作用是在两个线程同时访问`Instance`时，确保下一步是否new的准确性。第二个`if`则在两个线程排队等候时，进行判断，防止出现new两次的情况。

### 饿汉式单例

相比于直接在类中new出一个`Instance`这样做有以下优点：
- 懒加载：在外部未访问该类的实例时，不加载`Instance`，从而提升程序初始化速度
- 更强的灵活性和安全性：如在Unity中，通过索引器形式可以补充如果`Instance`为空则实例化并加载脚本的过程，防止因空引用导致出错(使用隐式加载确保安全性)

```csharp
// 饿汉式单例 - 类加载时就创建实例
public sealed class EagerSingleton
{
    // 在类加载时立即创建实例
    private static readonly EagerSingleton instance = new EagerSingleton();
    
    // 私有构造函数，防止外部实例化
    private EagerSingleton()
    {
        Console.WriteLine("EagerSingleton 构造函数被调用");
    }
    
    // 公共访问点
    public static EagerSingleton Instance
    {
        get { return instance; }
    }
    
    public void DoSomething()
    {
        Console.WriteLine("EagerSingleton 执行操作");
    }
}
```

### C# 6.0 自动属性初始化器

C# 6.0为我们带来了新的语法糖，**自动属性初始化器**

在定义属性时，对于只读自动属性（只有 `get` 访问器 ），可以直接在属性声明后使用 `= 初始化表达式` 的形式进行初始化。例如：

```csharp
// 使用C# 6.0自动属性初始化器的单例
public sealed class ModernSingleton
{
    // 自动属性初始化器 - 只读属性在声明时初始化
    public static ModernSingleton Instance { get; } = new ModernSingleton();
    
    // 私有构造函数
    private ModernSingleton()
    {
        Console.WriteLine("ModernSingleton 实例被创建");
    }
    
    public void DoSomething()
    {
        Console.WriteLine("ModernSingleton 执行操作");
    }
}

// 使用示例
class Program
{
    static void Main(string[] args)
    {
        // 直接通过属性访问实例
        var singleton = ModernSingleton.Instance;
        singleton.DoSomething();
        
        // 验证是同一个实例
        var anotherRef = ModernSingleton.Instance;
        Console.WriteLine($"Is same instance: {ReferenceEquals(singleton, anotherRef)}");
    }
}
```

使用时，通过 `Singleton.Instance` 就可以获取到该类的唯一实例，无需像之前版本那样在静态构造函数或其他地方单独初始化

有趣的是，这样做实际上是线程安全的，因为C# CLR 的特性 C# 运行时会确保静态字段的初始化是线程安全的。
当首次访问类的静态成员时 ，CLR（公共语言运行时）会负责按顺序初始化静态字段。CLR 内部会进行同步处理，保证多个线程不会同时对静态字段进行初始化操作，也就避免了多个线程同时创建单例实例的情况。

### 存在的问题

通过索引器的方式，似乎实现了"懒加载"，可实际上，它真的有那么"懒"吗？

```csharp
// 测试静态初始化的时机
public class SingletonTest
{
    static SingletonTest()
    {
        Console.WriteLine("静态构造函数被调用");
    }
    
    public SingletonTest()
    {
        Console.WriteLine("普通构造函数被调用");
    }
    
    // 静态实例 - 会在类首次被使用时立即初始化
    public static readonly SingletonTest Instance = new SingletonTest();
    
    // 其他静态成员
    public static string OtherStaticField = "其他静态字段";
}

// 测试代码
class Program
{
    static void Main(string[] args)
    {
        Console.WriteLine("程序开始");
        
        // 访问其他静态成员
        Console.WriteLine($"访问其他静态字段: {SingletonTest.OtherStaticField}");
        
        // 此时Instance已经被创建了
        Console.WriteLine("获取Instance");
        var instance = SingletonTest.Instance;
        
        Console.WriteLine("程序结束");
    }
}
```

```csharp
/* 执行结果：
程序开始
普通构造函数被调用
静态构造函数被调用
访问其他静态字段: 其他静态字段
获取Instance
程序结束
*/
```

上面是代码的执行结果，普通构造函数被调用，这说明`Instance`被new了出来，调用了普通构造，同时，当静态成员被全部初始化后，调用了静态的构造函数。

可是，我们不希望再调用其他静态成员时初始化我们的`Instance`，这说明，我们的实例不够"懒"

### 解决方案

在.NET 4.0 中C# 为我们带来了`Lazy<>`懒加载特性，它可以将实例再次封装为工厂方法，从而使外界访问静态对象时只会初始化`Lazy<>`本身的实例，减小开支。

```csharp
// 使用Lazy<T>实现真正的懒加载单例
public class LazySingleton
{
    // 静态构造函数
    static LazySingleton()
    {
        Console.WriteLine("静态构造函数被调用");
    }
    
    // 普通构造函数
    private LazySingleton()
    {
        Console.WriteLine("普通构造函数被调用");
    }
    
    // 使用Lazy<T>实现懒加载
    private static readonly Lazy<LazySingleton> _instance = 
        new Lazy<LazySingleton>(() => new LazySingleton());
    
    public static LazySingleton Instance => _instance.Value;
    
    // 其他静态成员
    public static string OtherStaticField = "其他静态字段";
    
    public void DoSomething()
    {
        Console.WriteLine("执行某些操作");
    }
}
```

```csharp
// 测试Lazy懒加载
class Program
{
    static void Main(string[] args)
    {
        Console.WriteLine("程序开始");
        
        // 访问其他静态成员 - 此时不会创建单例实例
        Console.WriteLine($"访问其他静态字段: {LazySingleton.OtherStaticField}");
        
        Console.WriteLine("准备获取Instance");
        
        // 只有在实际访问Instance时才会创建实例
        var instance = LazySingleton.Instance;
        
        instance.DoSomething();
        
        Console.WriteLine("程序结束");
    }
}

/* 执行结果：
程序开始
静态构造函数被调用
访问其他静态字段: 其他静态字段
准备获取Instance
普通构造函数被调用
执行某些操作
程序结束
*/
```

从执行结果中我们可以看出，静态成员仍然被全部实例化，包括`Lazy`，但是不同的是，获取`Instance`时对应的普通构造并没有被调用，而是在外界获取`Instance`实例后才被调用，这说明"懒加载"被成功实现。

## Python

```python
# Python 中的单例模式实现

# 方法一：使用 __new__ 方法
class Singleton:
    _instance = None
    
    def __new__(cls):
        if cls._instance is None:
            cls._instance = super(Singleton, cls).__new__(cls)
        return cls._instance
    
    def __init__(self):
        if not hasattr(self, 'initialized'):
            self.initialized = True
            print("Singleton 初始化")

# 方法二：使用装饰器
def singleton(cls):
    instances = {}
    def get_instance(*args, **kwargs):
        if cls not in instances:
            instances[cls] = cls(*args, **kwargs)
        return instances[cls]
    return get_instance

@singleton
class DatabaseConnection:
    def __init__(self):
        print("数据库连接已创建")
    
    def query(self, sql):
        return f"执行查询: {sql}"

# 使用示例
s1 = Singleton()
s2 = Singleton()
print(f"是否为同一实例: {s1 is s2}")  # True

db1 = DatabaseConnection()
db2 = DatabaseConnection()
print(f"数据库连接是否相同: {db1 is db2}")  # True
```

# 优势

相比于直接使用静态类，单例模式有以下优势

- **实现懒加载** - 只有在需要时才创建实例
- **便于依赖注入和单元测试** - 可以通过接口抽象，方便模拟测试
- **便于序列化和反序列化** - 单例实例可以参与序列化过程
- **控制实例数量** - 确保全局只有一个实例
- **节约系统资源** - 避免重复创建相同功能的对象

## 使用场景

- **配置管理器** - 全局配置信息访问
- **日志记录器** - 统一的日志输出管理  
- **数据库连接池** - 管理数据库连接资源
- **缓存管理器** - 全局缓存访问
- **线程池管理** - 统一的线程资源管理

## 注意事项

- **线程安全** - 多线程环境下需要考虑同步问题
- **序列化问题** - 序列化后反序列化可能创建新实例
- **反射破坏** - 通过反射可能绕过单例限制
- **单元测试困难** - 全局状态可能影响测试独立性
- **违背单一职责** - 既要管理实例创建又要处理业务逻辑































