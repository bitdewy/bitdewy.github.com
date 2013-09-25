---
layout: post
title: "C++11 std::unordered_map"
date: 2012-08-21 17:54
comments: true
categories: [C++, C++11]
---

##历史

在 C++ 中，第一个被广泛使用的哈希表实现是 SGI STL 中的，hash_map, hash_set, hash_multimap, hash_mutiset。
由于哈希表是非常常用的数据结构，逐渐的各厂家实现的标准库中都引入了该数据结构。

例如，GCC 的 lisbstdc++ , 以及微软的 MSVC 标准库。在这些公用的名字后面，有不同的实现，呃，不同的实现。它们在接口、能力、内在数据结构和支持操作的相关效率方面不同。写出使用哈希表的可移植代码是可能的，但不可能像使用标准库中的容器一样容易。（知道标准的重要性了吧。）好在 `hash_*` 这一组类加入了 C++ TR1 , 可惜由于名字被非标准的各家实现占用，只能退而求其次，改名为 `unordered_*`。


C++11 中，不排序的关联容器已经正式进入标准库。现在再选择的话，可以毫不犹豫的放弃非标准的 `hash_*` 而选择标准库中的 `unordered_*` 系类了。boost <[boost/unordered_map.hpp][boost_unordered_map]> 中同样有实现。

  [boost_unordered_map]: http://www.boost.org/doc/libs/1_53_0/boost/unordered_map.hpp

<!-- more -->
##选择合适的数据结构

对于一个程序员来说，分析具体的问题，选择适合的数据结构，有时候比算法更重要。好的数据结构可以帮助程序员解决大部分效率问题。Soctt Meyers 在《Effective STL》的第一条就提到：“仔细选择你的容器”。

`std::unordered_map` 与 `std::map` 的区别是, `std::map` 是按照 `operator<` 比较判断元素是否相同，以及比较元素的大小，然后选择合适的位置插入到树中。所以，如果对 map 进行遍历（中序遍历）的话，输出的结果是有序的。顺序就是按照 `operator<` 定义的大小排序。

而 `std::unordered_map` 是计算元素的 hash 值，根据 hash 值判断元素是否相同。所以，对 `unordered_map` 进行遍历，结果是无序的。

用法的区别是，`std::map` 的 key 需要定义 `operator<` 。 而 `std::unordered_map` 需要定义 `hash_value` 函数并且重载 `operator==`。
对于内置类型，如 `std::string`, 这些都不用操心。对于自定义的类型做 key, 就需要自己重载 `operator<` 或者 `hash_value()` 了。

选择时,请基于时间和空间的综合考虑, *当不关心空间, 也不需要结果排好序时, 可以选择 `unordered_map` 获得更好的时间效率。*

`std::map` 对应与 java 中的 `TreeMap`, 而 `std::unordered_map` 对应于 java 中的 `HashMap`. 对于标准库中的散列表无法取名为 `hash_*`, 只能叫 `unordered_*`, 我只能说… 标准库进展太慢，结果好白菜都让猪给拱了……遗憾啊……