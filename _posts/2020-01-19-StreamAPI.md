---
title: Stream API
tags: Java8 Stream
sidebar: 
  nav: docs-zh
---

Java 8 引入了全新的 Stream API，可以使用声明的方式来处理数据，极大地方便了集合操作。
### Stream对象的创建
#### Stream对象分为两种，一种串行的流对象，一种并行的流对象。
```
// 为集合创建串行流对象
Stream<User> stream = userList.stream();
// 为集合创建并行流对象
tream<User> parallelStream = userList.parallelStream();
```

### filter
#### 对Stream中的元素进行过滤操作，当设置条件返回true时返回相应元素。
```
List<User> dirList = userList.stream()
    .filter(user -> user.getAge() == 20)
    .collect(Collectors.toList());
```

### map
#### 对Stream中的元素进行转换处理后获取。比如可以将User对象转换成Long对象。 我们经常会有这样的需求：需要把某些对象的id提取出来，然后根据这些id去查询其他对象，这时可以使用此方法。
```
List<Long> idList = userList.stream()
    .map(user -> user.getId())
    .collect(Collectors.toList());
```

### limit
#### 从Stream中获取指定数量的元素。
```
// 获取前5个对象组成的集合
List<User> firstFiveList = userList.stream()
    .limit(5)
    .collect(Collectors.toList());
```

### count
#### 仅获取Stream中元素的个数。
```
long count = userList.stream()
    .filter(permission -> user.getAge() == 20)
    .count();
```

### sorted
#### 对Stream中元素按指定规则进行排序。
```
List<User> sortedList = userList.stream()
    .sorted((user1,user2)->{return user1.getAge().compareTo(user2.getAge());})
    .collect(Collectors.toList());
```

### skip
#### 跳过指定个数的Stream中元素，获取后面的元素
```
// 跳过前5个元素，返回后面的
List<User> skipList = userList.stream()
    .skip(5)
    .collect(Collectors.toList());
```

### 用collect方法将List转成map
#### 有时候我们需要反复对List中的对象根据id进行查询，我们可以先把该List转换为以id为key的map结构，然后再通过map.get(id)来获取对象，这样比较方便
```
// 将User列表以id为key，以权限对象为值转换成map
Map<Long, User> userMap = userList.stream()
    .collect(Collectors.toMap(user -> user.getId(), user -> user));
```
### 例子
给定["1","2","bilibili","of","codesheep","5","at","BILIBILI","codesheep","23","CHEERS","6"] 找出所有 长度>=5的字符串，并且忽略大小写、去除重复字符串，然后按字母排序，最后用“爱心❤”连接成一个字符串输出！
```
String result = list.stream()// 首先将列表转化为Stream流
         .filter( i ->!isNum(i))// 首先筛选出字母型字符串
         .filter(i ->i.length()>=5)// 其次筛选出长度>=5的字符串
         .map(i ->i.toLowerCase())// 字符串统一转小写
         .distinct()// 去重操作来一下
         .sorted(Comparator.naturalOrder())// 字符串排序来一下
         .collect(Collectors.joining("❤")); // 连词成句来一下，完美！
```