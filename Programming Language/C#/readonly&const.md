---
tags:
  - CSharp
  - readonly
  - const
---
# 相同点

- 声明常量

```csharp
// const 常量声明
public class ConstExample
{
    public const int MAX_SIZE = 100;
    public const string APP_NAME = "MyApplication";
    
    public void ShowConstants()
    {
        Console.WriteLine($"最大尺寸: {MAX_SIZE}");
        Console.WriteLine($"应用名称: {APP_NAME}");
    }
}

// readonly 常量声明
public class ReadonlyExample
{
    public readonly int MaxConnections;
    public readonly DateTime CreatedTime;
    
    public ReadonlyExample(int maxConnections)
    {
        MaxConnections = maxConnections;
        CreatedTime = DateTime.Now;
    }
}
```
*图：常量声明*

# 不同点

- `readonly` 可以在构造函数中初始化，而`const`必须在声明时初始化

```csharp
public class InitializationExample
{
    // const 必须在声明时初始化
    public const int COMPILE_TIME_CONSTANT = 42;
    
    // readonly 可以在声明时或构造函数中初始化
    public readonly int RuntimeConstant;
    public readonly string ConfigValue;
    
    public InitializationExample(int value, string config)
    {
        RuntimeConstant = value;  // 构造函数中初始化
        ConfigValue = config;
    }
    
    // 编译错误：const 不能在构造函数中赋值
    // public InitializationExample() 
    // {
    //     COMPILE_TIME_CONSTANT = 100;  // 错误！
    // }
}
```
---
- `readonly`只能用于声明类的字段，而`const`还可以用来声明局部常量

```csharp
public class ScopeExample
{
    // readonly 只能用于字段声明
    public readonly int FieldValue = 10;
    
    public void MethodExample()
    {
        // const 可以声明局部常量
        const int LOCAL_CONSTANT = 5;
        const string LOCAL_MESSAGE = "Hello";
        
        // 错误：readonly 不能用于局部变量
        // readonly int localReadonly = 20;  // 编译错误
        
        Console.WriteLine($"局部常量: {LOCAL_CONSTANT}");
        Console.WriteLine($"字段值: {FieldValue}");
    }
}
```
---
- `readonly`是语法，`const`则本质上是字面常量

```csharp
public class NatureExample
{
    public readonly int ReadonlyValue = 100;
    public const int ConstValue = 200;
    
    public void ReflectionTest()
    {
        // 可以通过反射修改 readonly 字段（尽管不推荐）
        var field = typeof(NatureExample).GetField("ReadonlyValue");
        field.SetValue(this, 999);
        
        Console.WriteLine($"修改后的 readonly 值: {ReadonlyValue}");
    }
}

// 演示通过反射修改 readonly
class Program
{
    static void Main()
    {
        var example = new NatureExample();
        Console.WriteLine($"原始值: {example.ReadonlyValue}");
        
        // 使用反射修改
        var fieldInfo = typeof(NatureExample).GetField("ReadonlyValue");
        fieldInfo.SetValue(example, 888);
        
        Console.WriteLine($"反射修改后: {example.ReadonlyValue}");
    }
}
```

```csharp
// const 的编译时优化演示
public class CompileTimeExample
{
    public const int ConstValue = 10;
    
    public void ShowOptimization()
    {
        // 编译器会直接将 ConstValue * 2 优化为字面量 20
        int result = ConstValue * 2;  // 编译后直接变成 int result = 20;
        
        Console.WriteLine($"结果: {result}");
    }
}
```
*图：通过反射修改`readonly`字段*

`const`只能通过类型获取，它类似于一个静态属性

```csharp
public class AccessExample
{
    public const int ConstField = 100;
    public static readonly int StaticReadonlyField = 200;
    public readonly int InstanceReadonlyField = 300;
    
    public static void ShowAccess()
    {
        // const 通过类型名直接访问（编译时替换）
        Console.WriteLine(AccessExample.ConstField);
        
        // static readonly 通过类型名访问（运行时）
        Console.WriteLine(AccessExample.StaticReadonlyField);
        
        // instance readonly 需要实例
        var instance = new AccessExample();
        Console.WriteLine(instance.InstanceReadonlyField);
    }
}
```
编译器聪明的计算出了常量×2的值，将字面量直接赋值给了变量，而不是访问`ConstValue`
这里很像C++里面的[[const]]和右[[引用]]，字面量本身没有确定的地址，通过`const`才能限制不被修改

```csharp
// 演示 readonly 和 const 的初始化差异
public class InitializationDifference
{
    // readonly 可以使用运行时值初始化
    public readonly string ReadonlyAppName = Environment.MachineName;
    public readonly DateTime ReadonlyStartTime = DateTime.Now;
    public static readonly string StaticReadonlyEmpty = string.Empty;
    
    // const 必须使用编译时常量
    public const string ConstAppVersion = "1.0.0";
    public const int ConstMaxRetries = 3;
    
    // 错误示例：const 不能使用运行时值
    // public const DateTime ConstCurrentTime = DateTime.Now;  // 编译错误
    // public const string ConstMachineName = Environment.MachineName;  // 编译错误
}
```

---
- `readonly`的初始化不必是编译时常量，`const`则必须是字面常量
在使用`readonly`时，未必需要直接赋值，还可以赋值静态属性(不是编译时常量)

```csharp
// readonly 可以赋值静态属性
public class ReadonlyStaticProperty
{
    // 可以将静态属性赋值给 readonly 字段
    public static readonly string EmptyString = string.Empty;
    public static readonly Guid EmptyGuid = Guid.Empty;
    public readonly List<string> EmptyList = new List<string>();
    
    public ReadonlyStaticProperty()
    {
        // 构造函数中也可以赋值
        EmptyList.Add("初始化项");
    }
}
```
*图：对readonly赋值静态属性`Empty`*

在对`const`赋值时必须是编译时常量

```csharp
// const 只能使用字面量或其他 const
public class ConstLiterals
{
    // 正确：字面量
    public const int MaxSize = 100;
    public const string AppName = "MyApp";
    public const bool IsDebug = true;
    
    // 正确：使用其他 const 常量
    public const int BufferSize = MaxSize * 2;
    public const string FullAppName = AppName + " v1.0";
    
    // 错误：不能使用非编译时常量
    // public const string CurrentTime = DateTime.Now.ToString();  // 编译错误
    // public const int RandomValue = new Random().Next();         // 编译错误
}
```
*图：只能对编译时常量赋值(字面量)*

```csharp
// 使用 const 对 const 赋值的示例
public class ConstToConst
{
    public const int BaseValue = 10;
    public const int DerivedValue = BaseValue * 3;  // 30
    public const string BaseMessage = "Hello";
    public const string FullMessage = BaseMessage + " World";  // "Hello World"
    
    public static void ShowValues()
    {
        Console.WriteLine($"基础值: {BaseValue}");
        Console.WriteLine($"派生值: {DerivedValue}");
        Console.WriteLine($"完整消息: {FullMessage}");
    }
}
```
*图：使用const对const赋值*

---
# 新特性

- C# 7.2: readonly struct & ref readonly method

```csharp
// readonly struct - 整个结构体不可变
public readonly struct Point
{
    public readonly int X;
    public readonly int Y;
    
    public Point(int x, int y)
    {
        X = x;
        Y = y;
    }
    
    // readonly 结构体的方法都是隐式 readonly 的
    public double DistanceFromOrigin()
    {
        return Math.Sqrt(X * X + Y * Y);
    }
}

// ref readonly 方法参数和返回值
public class RefReadonlyExample
{
    private readonly Point[] points = new Point[100];
    
    // ref readonly 返回值 - 避免复制大型值类型
    public ref readonly Point GetPointAt(int index)
    {
        return ref points[index];
    }
    
    // ref readonly 参数 - 避免复制但保证不被修改
    public double CalculateDistance(in Point p1, in Point p2)
    {
        int deltaX = p1.X - p2.X;
        int deltaY = p1.Y - p2.Y;
        return Math.Sqrt(deltaX * deltaX + deltaY * deltaY);
    }
}
```

```csharp
// 使用示例
var example = new RefReadonlyExample();
ref readonly Point point = ref example.GetPointAt(0);
// point.X = 10; // 编译错误：不能修改 readonly 引用

var p1 = new Point(0, 0);
var p2 = new Point(3, 4);
double distance = example.CalculateDistance(in p1, in p2);
```

- C# 8.0：readonly members

```csharp
// readonly 成员方法
public struct MutablePoint
{
    public int X { get; set; }
    public int Y { get; set; }
    
    public MutablePoint(int x, int y)
    {
        X = x;
        Y = y;
    }
    
    // readonly 方法 - 不会修改实例状态
    public readonly double DistanceFromOrigin()
    {
        return Math.Sqrt(X * X + Y * Y);
    }
    
    // readonly 属性
    public readonly string Coordinates => $"({X}, {Y})";
    
    // 非 readonly 方法 - 可以修改状态
    public void MoveTo(int newX, int newY)
    {
        X = newX;
        Y = newY;
    }
}
```

```csharp
// readonly 成员的好处
public class ReadonlyMemberBenefits
{
    public void Example()
    {
        var point = new MutablePoint(3, 4);
        
        // readonly 方法可以安全调用
        double distance = point.DistanceFromOrigin();
        string coords = point.Coordinates;
        
        // 编译器知道这些方法不会修改 point
    }
}
```

- C# 10.0：const string + string interpolation

```csharp
// C# 10.0 支持 const 字符串插值（编译时求值）
public class ConstStringInterpolation
{
    private const string AppName = "MyApplication";
    private const string Version = "2.0";
    
    // C# 10.0 新特性：const 字符串插值
    private const string FullVersion = $"{AppName} v{Version}";
    private const string WelcomeMessage = $"Welcome to {AppName}!";
    
    // 也支持更复杂的表达式
    private const int Major = 2;
    private const int Minor = 1;
    private const string VersionString = $"Version {Major}.{Minor}";
    
    public static void ShowInfo()
    {
        Console.WriteLine(FullVersion);      // "MyApplication v2.0"
        Console.WriteLine(WelcomeMessage);   // "Welcome to MyApplication!"
        Console.WriteLine(VersionString);    // "Version 2.1"
    }
}
```

# 总结

```csharp
// 总结表格的代码示例
public class Summary
{
    // const: 编译时常量，静态，值类型和字符串
    public const int CompileTimeConstant = 100;
    public const string CompileTimeString = "常量字符串";
    
    // readonly: 运行时常量，可实例化，任何类型
    public readonly DateTime RuntimeConstant = DateTime.Now;
    public readonly List<string> RuntimeCollection = new List<string>();
    
    // static readonly: 静态运行时常量
    public static readonly string StaticRuntimeConstant = Environment.MachineName;
    
    public Summary(int value)
    {
        // readonly 可以在构造函数中赋值
        RuntimeConstant = DateTime.Now.AddDays(value);
        
        // const 不能在构造函数中赋值
        // CompileTimeConstant = value;  // 编译错误
    }
}

/*
比较总结：
╔══════════════╦════════════╦═══════════════╦═══════════════════╦═════════════════╗
║    特性      ║    const   ║    readonly   ║  static readonly  ║      说明       ║
╠══════════════╬════════════╬═══════════════╬═══════════════════╬═════════════════╣
║ 初始化时机   ║  编译时    ║   运行时      ║      运行时       ║ const最早       ║
║ 初始化位置   ║  声明时    ║ 声明或构造函数 ║   静态构造函数    ║ readonly更灵活  ║
║ 存储位置     ║   无       ║   实例        ║      静态         ║ const直接内联   ║
║ 支持类型     ║ 基本+字符串 ║    任何类型   ║     任何类型      ║ readonly更全面  ║
║ 性能         ║   最高     ║     中等      ║       中等        ║ const零开销     ║
╚══════════════╩════════════╩═══════════════╩═══════════════════╩═════════════════╝
*/
```