---
layout: post
title: "C++ ORM 框架 ODB"
date: 2012-08-21 22:03
comments: true
categories: [C++, ORM]
---

##什么是 ORM？

对象关系映射（Object Relational Mapping，简称ORM，或O/RM，或O/R mapping），是一种程序设计技术，用于实现面向对象编程语言里不同类型系统的数据之间的转换。从效果上说，它其实是创建了一个可在编程语言里使用的“虚拟对象数据库”。如今已有很多免费和收费的ORM产品，而有些程序员更倾向于创建自己的ORM工具。

面向对象是从软件工程基本原则（如耦合、聚合、封装）的基础上发展起来的，而关系数据库则是从数学理论发展而来的，两套理论存在显著的区别。为了解决这个不匹配的现象，对象关系映射技术应运而生。

简单的说：ORM相当于中继数据。通俗的说，ORM就是将对象和它们之间的关系映射成关系数据库的表，以及在数据库中它们的关系。目标就是确定一种持久化对象数据的有效策略，在考虑类之间继承结构的同时，存储各个对象的数据属性和对象间的关系。

<!-- more -->
##成熟的 ORM 产品

- Java: 广泛使用的 ORM 框架Hibernate
- Python: web框架 Django 中包含了 ORM
- C++: ODB，QxOrm，LiteSQL 等

这里有一个 ORM 产品的列表：[List of object-relational mapping software][ORMList]

  [ORMList]: http://en.wikipedia.org/wiki/List_of_object-relational_mapping_software


##ODB

ODB 是一个开源的，支持多平台，支持多数据库的 C++ 的 ORM 框架，可将 C++ 对象数据库表映射，进行轻松的数据库查询和操作。
下面是常规的 C++ 代码：

```cpp
class person
{
    //...
private:
 
    std::string email_;
 
    std::string name_;
    unsigned short age_;
};
```

下面是ODB所需的持久化之后的 C++ 代码， ODB可以根据下面的代码自动生成数据库：

```cpp
#pragma db object
class person
{
    //...
private:
    friend class odb::access;
    person() {}
 
#pragma db id
    std::string email_;
 
    std::string name_;
    unsigned short age_;
};
```

有了上面的声明，我们就可以对 person 对象进行一系列的数据库操作了，下面是一个 SQLite 的例子：

```cpp
odb::sqlite::database db("people.db");
 
person john("john@doe.org", "John Doe", 31);
person jane("jane@doe.org", "Jane Doe", 29);
 
odb::transaction t(db.begin());
 
db.persist(john);
db.persist(jane);
 
typedef odb::query<person> person_query;
 
for (auto& p: db.query<person>(person_query::age < 30))
    cout << p << endl;
 
jane.age(jane.age() + 1);
db.update(jane);
 
t.commit();
```

有了 ORM 所有的数据库的操作也变得面向对象了，代码中不会夹杂着 SQL语句，写起代码来应该会更顺手了。
