---
tags:
  - Python
---
# 定义

迭代器通常是对某个数据集合元素的直接遍历

## 引例

在python中，for循环不同于C++,C#等语言，其本身就是在遍历迭代器

```python
# Python 中的 for 循环本质上是迭代器遍历
numbers = [1, 2, 3, 4, 5]

# 这个 for 循环实际上是在遍历迭代器
for num in numbers:
    print(num)

# 等价于手动使用迭代器
iterator = iter(numbers)
try:
    while True:
        num = next(iterator)
        print(num)
except StopIteration:
    pass

# range 也产生一个迭代器
for i in range(5):
    print(i)  # 输出 0, 1, 2, 3, 4

# range 的迭代器本质
range_iter = iter(range(5))
print(next(range_iter))  # 0
print(next(range_iter))  # 1
print(next(range_iter))  # 2
```

在引例的for循环中，`range`产生一个迭代器，并通过`for`遍历，从而达到模仿其他语言`for`循环的效果

# 定义迭代器

### 不使用yield语法糖

```python
# 不使用 yield 实现的迭代器类
class NumberSequence:
    def __init__(self, start, end):
        self.start = start
        self.end = end
        self.current = start
    
    def __iter__(self):
        return self  # 返回迭代器对象本身
    
    def __next__(self):
        if self.current < self.end:
            result = self.current
            self.current += 1
            return result
        else:
            raise StopIteration  # 迭代结束时抛出异常

# 使用自定义迭代器
seq = NumberSequence(1, 5)
for num in seq:
    print(num)  # 输出: 1, 2, 3, 4

# 或者手动迭代
seq2 = NumberSequence(10, 13)
iterator = iter(seq2)
print(next(iterator))  # 10
print(next(iterator))  # 11
print(next(iterator))  # 12
# print(next(iterator))  # 抛出 StopIteration
```
*图：不使用yield实现的迭代器*

当一个类实现了`__iter__`和`__next__`方法，便可以被迭代，作为迭代器传入

其中：
- `__iter__`需要返回本身`self`
- `__next__`返回迭代过程中的目标结果，即你想让外部迭代访问到的结果

### 使用yield

```python
# 使用 yield 关键字定义生成器函数
def number_generator(start, end):
    current = start
    while current < end:
        yield current  # yield 暂停函数执行，返回值给调用者
        current += 1

# 使用生成器
gen = number_generator(1, 5)
for num in gen:
    print(num)  # 输出: 1, 2, 3, 4

# 斐波那契数列生成器
def fibonacci(n):
    a, b = 0, 1
    count = 0
    while count < n:
        yield a
        a, b = b, a + b
        count += 1

# 使用斐波那契生成器
fib = fibonacci(8)
for num in fib:
    print(num)  # 输出: 0, 1, 1, 2, 3, 5, 8, 13

# 生成器表达式（类似列表推导式但返回生成器）
squares = (x**2 for x in range(10))
for square in squares:
    print(square)  # 输出: 0, 1, 4, 9, 16, 25, 36, 49, 64, 81
```

使用`yield`，将迭代器封装在一个函数之中，可以直接通过调用函数进行迭代，`yield`返回的即为想要的结果

















