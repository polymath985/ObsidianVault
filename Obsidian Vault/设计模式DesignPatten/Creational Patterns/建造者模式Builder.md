---
tags:
  - 设计模式
  - CSharp
---
# 定义

建造者模式是将一个复杂的对象的构建与他的表示分离，使得同样的构建过程可以创建不同的表示。创建者模式隐藏了复杂对象的创建过程，他把复杂对象的创建过程加以抽象，通过子类继承或者重载的方式，动态的创建复杂的，具有复合属性的对象。

```csharp
// 产品类 - 复杂对象
public class Computer
{
    public string CPU { get; set; }
    public string Memory { get; set; }
    public string Storage { get; set; }
    public string GraphicsCard { get; set; }
    
    public void ShowSpecs()
    {
        Console.WriteLine($"Computer Specs:");
        Console.WriteLine($"CPU: {CPU}");
        Console.WriteLine($"Memory: {Memory}");
        Console.WriteLine($"Storage: {Storage}");
        Console.WriteLine($"Graphics Card: {GraphicsCard}");
    }
}

// 抽象建造者
public abstract class ComputerBuilder
{
    protected Computer computer;
    
    public ComputerBuilder()
    {
        computer = new Computer();
    }
    
    public abstract void BuildCPU();
    public abstract void BuildMemory();
    public abstract void BuildStorage();
    public abstract void BuildGraphicsCard();
    
    public Computer GetResult()
    {
        return computer;
    }
}

// 具体建造者 - 游戏电脑
public class GamingComputerBuilder : ComputerBuilder
{
    public override void BuildCPU()
    {
        computer.CPU = "Intel Core i9-12900K";
    }
    
    public override void BuildMemory()
    {
        computer.Memory = "32GB DDR4-3200";
    }
    
    public override void BuildStorage()
    {
        computer.Storage = "1TB NVMe SSD";
    }
    
    public override void BuildGraphicsCard()
    {
        computer.GraphicsCard = "NVIDIA RTX 4080";
    }
}
```

# 优点

- 一定程度上，消除了类爆炸问题
- 职责分离，由单独一个生产线组装产品

# 缺点

- 配件配置变得固定了，不能随意组合
- 对于大多数场景依然过于复杂，比如，未必每一个配置的手机都需要一个生产线，组装手机也未必需要一个单独的生产线

# 简化

将`Build`方法放在基类而非`Director`中，简化使用，相当于提取了共同方法，实现代码复用，但丧失了一定灵活度（可以用`virtual`?)

```csharp
// PhoneBuilder抽象类定义
protected Phone Phone;

public PhoneBuilder()
{
    Phone = new Phone();
}

public abstract void BuildCpu();
public abstract void BuildMem();  
public abstract void BuildScreen();

public Phone Build()
{
    BuildCpu();
    BuildMem();
    BuildScreen();
    return Phone;
}
```
*图：简化过的`Builder`*

```csharp
// 使用高端和低端PhoneBuilder的示例
class Program
{
    static void Main(string[] args)
    {
        PhoneBuilder highPhoneBuilder = new HighPhoneBuilder();
        Phone highPhone = highPhoneBuilder.Build();
        highPhone.Show();
        
        Console.WriteLine();
        PhoneBuilder lowPhoneBuilder = new LowPhoneBuilder();
        Phone lowPhone = lowPhoneBuilder.Build();
        lowPhone.Show();
    }
}
```
*图：主函数调用情况*

```csharp
class Program
{
    static void Main(string[] args)
    {
        IPhoneBuilder phoneBuilder = new PhoneBuilder();
        Phone phone = phoneBuilder
            .BuildCpu(cpu => { cpu.Type = "8核16线程"; })
            .BuildMem(mem => { mem.Type = "32G"; })
            .BuildScreen(screen => { screen.Type = "10寸"; })
            .Build();
        phone.Show();
    }
}
```

# 进一步优化

为了实现接口隔离，可以再将继承的父类替换为接口`IBuilder`

通过委托结合`return this`实现链式编程，太优雅了！

```csharp
// 建造者模式链式编程示例
class Program
{
    static void Main(string[] args)
    {
        IPhoneBuilder phoneBuilder = new PhoneBuilder();
        Phone phone = phoneBuilder
            .BuildCpu(cpu => { cpu.Type = "8核16线程"; })
            .BuildMem(mem => { mem.Type = "32G"; })
            .BuildScreen(screen => { screen.Type = "10寸"; })
            .Build();
        phone.Show();
    }
}
```
*图：新的Builder类*

```csharp
protected Phone Phone;

public PhoneBuilder()
{
    Phone = new Phone();
}

public abstract void BuildCpu();
public abstract void BuildMem();
public abstract void BuildScreen();

public Phone Build()
{
    BuildCpu();
    BuildMem();
    BuildScreen();
    return Phone;
}
```
*图：调用实现构造*