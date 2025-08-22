---
tags:
  - CSharp
  - readonly
  - const
---
# 相同点

- 声明常量
![[Pasted image 20250502101757.png]]
*图：常量声明*

# 不同点

- `readonly` 可以在构造函数中初始化，而`const`必须在声明时初始化
![[Pasted image 20250502104225.png]]
---
- `readonly`只能用于声明类的字段，而`const`还可以用来声明局部常量
![[Pasted image 20250502104344.png]]
---
- `readonly`是语法，`const`则本质上是字面常量
![[Pasted image 20250502104601.png]]
![[Pasted image 20250502104629.png]]
*图：通过反射修改`readonly`字段*

`const`只能通过类型获取，它类似于一个静态属性
![[Pasted image 20250502104830.png]]
编译器聪明的计算出了常量×2的值，将字面量直接赋值给了变量，而不是访问`ConstValue`
这里很像C++里面的[[const]]和右[[引用]]，字面量本身没有确定的地址，通过`const`才能限制不被修改

![[Pasted image 20250502105009.png]]

---
- `readonly`的初始化不必是编译时常量，`const`则必须是字面常量
在使用`readonly`时，未必需要直接赋值，还可以赋值静态属性(不是编译时常量)
![[Pasted image 20250502105623.png]]
*图：对readonly赋值静态属性`Empty`*

在对`const`赋值时必须是编译时常量
![[Pasted image 20250502105926.png]]
*图：只能对编译时常量赋值(字面量)*

![[Pasted image 20250502110032.png]]
*图：使用const对const赋值*

---
# 新特性

- C# 7.2: readonly struct & ref readonly method

![[Pasted image 20250502110528.png]]

![[Pasted image 20250502110645.png]]

- C# 8.0：readonly members

![[Pasted image 20250502110751.png]]

![[Pasted image 20250502110920.png]]

- C# 10.0：const string + string interpolation

![[Pasted image 20250502111024.png]]

# 总结

![[Pasted image 20250502111054.png]]