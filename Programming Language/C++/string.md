---
tags:
  - CPP
---
# string的函数成员

```
string str = "hello world";
```

- 长度

```
str.size();
str.length();
```

- 判空

```
 //返回bool值
 str.empty();
```

- 在指定位置插入字符串

```
//在指定位置插入指定数量的char
str.insert(2,3,'c');
//在指定位置插入字符串
str.insert(2,"This is a string");
```

- 截取子串

```
//从指定位置截取指定数量的子串
string sub = str.substr(1,2);
//从指定位置向后截取子串
string sub = str.substr(1);
```

- 寻找子串位置

```
//返回第一个子串的首字母位置
int pos = str.find("target");
//倒序寻找
int pos = str.rfind("target");
```

> [!NOTE] 未找到子串时返回静态成员string::npos



