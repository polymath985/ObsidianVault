---
tags:
  - 设计模式
  - CSharp
---
# 定义

原型模式是用原型实例指定创建对象的种类，并通过拷贝这些原型创建新的对象。简单来说，首先创建一个实例，然后通过这个实例去拷贝（克隆）创建新的实例。

# 基本实现

只需要在建造者模式的基础上，在被构建的产品中写入`Clone`方法

```csharp
// 原型接口
public interface IPrototype<T>
{
    T Clone();
}

// 具体原型类
public class Resume : IPrototype<Resume>
{
    public string Name { get; set; }
    public string Gender { get; set; }
    public int Age { get; set; }
    public decimal ExpectedSalary { get; set; }
    public WorkExperience WorkExp { get; set; }
    
    public Resume(string name, string gender, int age)
    {
        Name = name;
        Gender = gender;
        Age = age;
        WorkExp = new WorkExperience();
    }
    
    // 实现克隆方法
    public Resume Clone()
    {
        // 浅拷贝示例
        return (Resume)this.MemberwiseClone();
    }
}
```

```csharp
// 工作经验类
public class WorkExperience
{
    public string Company { get; set; }
    public string Position { get; set; }
    public DateTime StartDate { get; set; }
    public DateTime EndDate { get; set; }
    
    public WorkExperience()
    {
        Company = "";
        Position = "";
        StartDate = DateTime.Now;
        EndDate = DateTime.Now;
    }
}

// 使用示例
class Program
{
    static void Main(string[] args)
    {
        Resume originalResume = new Resume("张三", "男", 25);
        originalResume.ExpectedSalary = 8000;
        originalResume.WorkExp.Company = "阿里巴巴";
        originalResume.WorkExp.Position = "软件工程师";
        
        // 克隆简历
        Resume clonedResume = originalResume.Clone();
        clonedResume.Name = "李四";
        clonedResume.ExpectedSalary = 9000;
        
        Console.WriteLine($"Original: {originalResume.Name}, Salary: {originalResume.ExpectedSalary}");
        Console.WriteLine($"Cloned: {clonedResume.Name}, Salary: {clonedResume.ExpectedSalary}");
    }
}
```
# 使用场景

- 需要创建一个包含大量公共属性，而只需要修改少量属性的对象时
- 当需要重复创建一个初始化需要消耗大量资源的对象时

# 基本实现的缺点

- 仍然通过直接`new`来生成新对象，性能开销极大
- `clone`方法违背开闭原则

# 解决方案

## MemberwiseClone方法

调用`object`父类之中`MemberWiseClone`方法，产生一个浅拷贝结果

```csharp
// 使用MemberwiseClone的浅拷贝示例
public class Person : ICloneable
{
    public string Name { get; set; }
    public int Age { get; set; }
    public Address HomeAddress { get; set; }  // 引用类型
    
    public Person(string name, int age)
    {
        Name = name;
        Age = age;
        HomeAddress = new Address();
    }
    
    // 使用MemberwiseClone实现浅拷贝
    public object Clone()
    {
        return this.MemberwiseClone();
    }
}

public class Address
{
    public string Street { get; set; }
    public string City { get; set; }
    
    public Address()
    {
        Street = "";
        City = "";
    }
}
```

```csharp
// 浅拷贝测试
class Program
{
    static void Main(string[] args)
    {
        Person person1 = new Person("张三", 30);
        person1.HomeAddress.Street = "中山路123号";
        person1.HomeAddress.City = "上海";
        
        // 浅拷贝
        Person person2 = (Person)person1.Clone();
        person2.Name = "李四";  // 值类型独立修改
        person2.Age = 25;
        
        // 修改引用类型会影响原对象
        person2.HomeAddress.Street = "南京路456号";
        
        Console.WriteLine($"Person1: {person1.Name}, {person1.Age}, {person1.HomeAddress.Street}");
        Console.WriteLine($"Person2: {person2.Name}, {person2.Age}, {person2.HomeAddress.Street}");
        // 输出显示两个对象的地址是相同的
    }
}
```
对字符串来说，每次修改会创建新的位置，改变地址，所以不会影响原型，但对其他引用类型就不行了

可是，这样是浅拷贝，外界更改可能会影响数据错乱

## 序列化方法

使用时需要给类及其成员打上`[Serializable]`

这样的话，一般将该类打上`sealed`防止被子类继承

```csharp
// 序列化克隆方法实现（深拷贝方式一）
public override ResumeBase Clone()
{
    using (MemoryStream stream = new MemoryStream())
    {
        BinaryFormatter bf = new BinaryFormatter();
        bf.Serialize(stream, this);
        stream.Position = 0;
        return bf.Deserialize(stream) as ResumeBase;
    }
}
```
*图：序列化克隆(方法似乎已过期？)*

```csharp
// 手动深拷贝克隆方法实现（深拷贝方式二）  
public override ResumeBase Clone()
{
    ItResume resume = new ItResume()
    {
        Name = this.Name,
        Gender = this.Gender,
        Age = this.Age,
        ExpectedSalary = this.ExpectedSalary,
        WorkExperence = new WorkExperence
        {
            Company = this.WorkExperence.Company,
            Detail = this.WorkExperence.Detail,
            StartDate = this.WorkExperence.StartDate,
            EndDate = this.WorkExperence.EndDate
        }
    };
    return resume;
}
```

