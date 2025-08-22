---
tags:
  - Python
---

```python
# 创建集合的几种方式
# 1. 使用花括号创建非空集合
fruits = {"苹果", "香蕉", "橘子", "苹果"}  # 重复元素会被自动去除
print(fruits)  # {'苹果', '香蕉', '橘子'}

# 2. 使用 set() 函数创建
numbers = set([1, 2, 3, 2, 1])
print(numbers)  # {1, 2, 3}

# 3. 从字符串创建集合（每个字符成为一个元素）
chars = set("hello")
print(chars)  # {'h', 'e', 'l', 'o'}

# 4. 创建空集合（必须用 set()）
empty_set = set()
print(type(empty_set))  # <class 'set'>

# 注意：不能用空大括号创建集合
empty_dict = {}
print(type(empty_dict))  # <class 'dict'>
```

```python
# 集合的基本操作

# 添加元素
colors = {"红色", "绿色", "蓝色"}
colors.add("黄色")
print(colors)  # {'红色', '绿色', '蓝色', '黄色'}

# 批量添加元素
colors.update(["紫色", "橙色"])
print(colors)  # {'红色', '绿色', '蓝色', '黄色', '紫色', '橙色'}

# 删除元素
colors.remove("红色")  # 如果元素不存在会抛出 KeyError
colors.discard("黑色")  # 如果元素不存在不会报错
print(colors)

# 集合运算
set1 = {1, 2, 3, 4, 5}
set2 = {4, 5, 6, 7, 8}

# 并集
union_set = set1 | set2
print(f"并集: {union_set}")  # {1, 2, 3, 4, 5, 6, 7, 8}

# 交集
intersection_set = set1 & set2
print(f"交集: {intersection_set}")  # {4, 5}

# 差集
difference_set = set1 - set2
print(f"差集: {difference_set}")  # {1, 2, 3}

# 对称差集（异或）
symmetric_diff = set1 ^ set2
print(f"对称差集: {symmetric_diff}")  # {1, 2, 3, 6, 7, 8}

# 判断包含关系
subset = {1, 2}
print(f"{subset} 是 {set1} 的子集: {subset.issubset(set1)}")  # True
print(f"{set1} 是 {subset} 的超集: {set1.issuperset(subset)}")  # True
```

> [!NOTE] 注意
> 不能用空大括号{}创建集合，因为会创建一个空字典
