---
tags:
  - TypeScript
  - 接口
---

```typescript
// 接口基础定义
interface Person {
    name: string;
    age: number;
    email?: string;  // 可选属性
    readonly id: number;  // 只读属性
}

// 类实现接口
class Employee implements Person {
    readonly id: number;
    name: string;
    age: number;
    email?: string;
    department: string;  // 类可以有额外的属性
    
    constructor(id: number, name: string, age: number, department: string) {
        this.id = id;
        this.name = name;
        this.age = age;
        this.department = department;
    }
}

// 接口可以描述函数
interface GreetFunction {
    (name: string): string;
}

const greet: GreetFunction = (name) => `Hello, ${name}!`;
```

感觉和`C#`差不多，不过类的实现接口需要`implements`关键字

```typescript
// 接口继承使用 extends 关键字
interface Animal {
    name: string;
    age: number;
}

interface Dog extends Animal {
    breed: string;
    bark(): void;
}

interface Cat extends Animal {
    color: string;
    meow(): void;
}

// 多重继承
interface Pet extends Animal {
    owner: string;
}

interface ServiceDog extends Dog, Pet {
    serviceType: string;
}

const myServiceDog: ServiceDog = {
    name: "Max",
    age: 3,
    breed: "Golden Retriever",
    owner: "张三",
    serviceType: "Guide Dog",
    bark() {
        console.log("Woof!");
    }
};
```
接口继承使用`extends`关键字

```typescript
// 类型别名 vs 接口
// 类型别名可以为任意类型指定别名
type StringOrNumber = string | number;
type UserID = string;
type Point = { x: number; y: number };

// 接口只能描述对象类型
interface IPoint {
    x: number;
    y: number;
}

// 类型别名可以使用联合类型、交叉类型等
type Status = "loading" | "success" | "error";
type UserWithStatus = Person & { status: Status };

// 接口可以声明合并（同名接口会自动合并）
interface MergeExample {
    name: string;
}

interface MergeExample {
    age: number;
}

// 现在 MergeExample 包含 name 和 age
const merged: MergeExample = {
    name: "张三",
    age: 25
};
```
类型别名也可以达到相同的效果，不过其还能为任意类型指定别名

