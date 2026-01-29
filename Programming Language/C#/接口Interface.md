---
tags:
  - CSharp
  - 接口
date: 2025-04-24
---
# 概念

C#中的借口是一个类似于类的东西，可以封装方法属性等内容，相比于类，他可以实现多继承，也可以用来作为约束传参的对象

# 语法

### 接口本身

```csharp
// C# 8.0+ 接口语法示例
public interface IAdvancedInterface
{
    // 属性 - 子类必须实现
    string Name { get; set; }
    int Count { get; }
    
    // 索引器
    string this[int index] { get; set; }
    
    // 公有方法 - 可以有默认实现
    void PublicMethod()
    {
        Console.WriteLine("默认的公有方法实现");
        PrivateMethod(); // 可以调用私有方法
    }
    
    // 抽象方法 - 子类必须实现
    void AbstractMethod();
    
    // 私有方法 - 必须有实现（只能在接口内部使用）
    private void PrivateMethod()
    {
        Console.WriteLine("私有方法实现");
    }
    
    // 受保护方法 - 可以在子类接口中实现
    protected void ProtectedMethod()
    {
        Console.WriteLine("受保护方法");
    }
    
    // 静态方法 - 必须有实现
    static void StaticMethod()
    {
        Console.WriteLine("静态方法");
    }
    
    // 静态属性
    static string StaticProperty => "静态属性值";
}
```
在.NET 8.0中，C#为我们带来了接口的各种语法，接口中可以写入属性，索引器，公有函数，私有函数，Protected函数，甚至是静态函数和静态属性。

对于属性，其不能含有实例字段，在编译器中，编译后的代码仅仅含有属性的签名，其相当于对子类的约束，子类必须在其中实现属性。

```csharp
// 接口与类的属性区别
public interface IExample
{
    // 接口中的属性只有签名，没有backing field
    string Property { get; set; }
}

public class ExampleClass : IExample
{
    // 类中实现时需要提供实际的存储
    private string _property;
    public string Property 
    { 
        get => _property; 
        set => _property = value; 
    }
}
```

对于函数，其可以是Public,`private`,Protected, Static, Virtual.

| 访问修饰符     | 是否可以不实现 | 原因                  |
| --------- | ------- | ------------------- |
| public    | ✓       | 可以在子类中实现            |
| private   | ×       | 其只有可能在接口内部调用，不实现无意义 |
| protected | ✓       | 可以在子类接口中实现          |
| static    | ×       | 本身即为实例字段，不能不实现      |
| virtual   | ×       | 使用base.时不能没有实例      |
### 继承类

- 类在继承接口后必须实现其中成员或者不包含默认方法的函数
- 在同一个类继承的接口中，若接口有相同的函数成员或者属性，必须借助显式实现

```csharp
// 显式实现示例
public interface IA
{
    void Method();
}

public interface IB
{
    void Method();
}

public class MyClass : IA, IB
{
    // 隐式实现 - 同时满足两个接口
    public void Method()
    {
        Console.WriteLine("隐式实现的方法");
    }
    
    // 或者使用显式实现来区分
    void IA.Method()
    {
        Console.WriteLine("IA的方法实现");
    }
    
    void IB.Method()
    {
        Console.WriteLine("IB的方法实现");
    }
}
```

显式实现后，方法前不能写访问修饰符，因为此时无法通过类来调用方法(目的是为了保证调用接口的确定性)，相当于都是`private`

```csharp
// 显示实现与隐式实现及其调用
public class TestClass : IA, IB
{
    // 隐式实现 - 可以通过类实例直接调用
    public void CommonMethod()
    {
        Console.WriteLine("隐式实现，可通过类实例调用");
    }
    
    // 显式实现 - 只能通过接口引用调用
    void IA.SpecialMethod()
    {
        Console.WriteLine("显式实现，只能通过IA接口调用");
    }
}

// 使用示例
var obj = new TestClass();
obj.CommonMethod(); // 直接调用

// 显式实现的方法只能通过接口调用
((IA)obj).SpecialMethod(); // 强制转换为接口后调用
```

**接口默认方法的多态特性**

接口中默认方法的实现类似于`virtual`虚方法，体现了以下特性：
- 子类实现会覆盖接口的默认实现
- 即使转换为接口类型，仍会调用子类的实现
- 同名方法可以同时实现多个接口的要求

```csharp
public interface IBase
{
    void Method() // 接口默认方法
    {
        Console.WriteLine("接口默认实现");
    }
}

public class Derived : IBase
{
    public void Method() // 子类实现覆盖默认实现
    {
        Console.WriteLine("子类实现");
    }
}

// 使用示例
var obj = new Derived();
obj.Method(); // 输出: 子类实现

IBase interfaceRef = obj;
interfaceRef.Method(); // 输出: 子类实现 (多态调用)
```

- 接口方法仍然可以被显示实现成不同的方法，来达成多态的目的

```csharp
public interface IA
{
    void CommonMethod();
}

public interface IB
{
    void CommonMethod();
}

public class MultiImplementation : IA, IB
{
    // 显式实现不同接口的同名方法
    void IA.CommonMethod()
    {
        Console.WriteLine("IA的实现");
    }
    
    void IB.CommonMethod()
    {
        Console.WriteLine("IB的实现");
    }
}

// 使用时根据接口类型调用不同实现
var obj = new MultiImplementation();
((IA)obj).CommonMethod(); // 输出: IA的实现
((IB)obj).CommonMethod(); // 输出: IB的实现
```


## 使用

- 进行类的约束，或者实现多态
- 通过接口进行类的部分提取，简化代码结构，提升复用性。相比于基类，接口更加灵活。

```csharp
// 接口的约束示例
public void ProcessCollection<T>(IEnumerable<T> collection) where T : IComparable<T>
{
    foreach (T item in collection)
    {
        Console.WriteLine($"Processing: {item}");
    }
    
    // 可以对集合进行排序，因为T实现了IComparable
    var sortedList = collection.OrderBy(x => x).ToList();
}

// 使用示例 - 同样的方法可以处理不同的集合类型
var array = new int[] { 3, 1, 4, 1, 5 };
var list = new List<string> { "c", "a", "b" };

ProcessCollection(array);  // 处理数组
ProcessCollection(list);   // 处理列表
```

在这个例子中，我们只关心集合的可枚举性和元素的可比较性，而不去关心它们具体的实现细节（如Count、Add、Remove等方法）。C#为我们提供了`IEnumerable<T>`和`IComparable<T>`提供通用约束。

```csharp
// 接口提供的抽象和复用性
public interface IRepository<T>
{
    Task<T> GetByIdAsync(int id);
    Task<IEnumerable<T>> GetAllAsync();
    Task<T> CreateAsync(T entity);
    Task UpdateAsync(T entity);
    Task DeleteAsync(int id);
}

// 不同的存储实现
public class DatabaseRepository<T> : IRepository<T>
{
    // 数据库实现
    public async Task<T> GetByIdAsync(int id) { /* 数据库查询 */ }
    // ... 其他方法
}

public class CacheRepository<T> : IRepository<T>
{
    // 缓存实现  
    public async Task<T> GetByIdAsync(int id) { /* 缓存查询 */ }
    // ... 其他方法
}
```

