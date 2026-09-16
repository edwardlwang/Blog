---
title: C++ 迭代器详解：从 unordered_map 的 find 与 operator[] 说起
date: 2026-09-10 15:31:48
tags:
  - C++
  - iterator
---

在刷算法题时，我们经常会用到 `unordered_map`。一个常见的场景是判断某个单词是否在哈希表中，并减少它的计数。很多人会写出这样的代码：

```cpp
if (hash_words[temp_word] == 0) {
    return false;
} else {
    hash_words[temp_word]--;
}
```

这段代码能跑，但性能并不好。更好的写法是用 `find`：

```cpp
auto it = remaining.find(temp_word);
if (it == remaining.end() || it->second == 0) {
    return false;
}
it->second--;
```

为什么 `find` 版本更快？要回答这个问题，我们需要理解 C++ 迭代器。这篇文章就从迭代器的基础讲起，逐步深入到 `unordered_map` 的底层行为。

---

## 一、迭代器是什么

**迭代器（iterator）是一个“泛化的指针”**。它提供了一种统一的方式来遍历容器中的元素，而不需要关心容器底层的数据结构。

你可以把它理解为一个“位置标记”：

- `begin()` 返回指向第一个元素的迭代器
- `end()` 返回指向**最后一个元素之后**的迭代器（不是最后一个元素本身）
- `*it` 解引用，拿到当前元素
- `++it` 移动到下一个元素

```cpp
vector<int> v = {1, 2, 3};
for (auto it = v.begin(); it != v.end(); ++it) {
    cout << *it << " ";  // 输出 1 2 3
}
```

迭代器让算法和容器解耦。比如 `std::find` 可以作用于 `vector`、`list`、`map`，因为它们都提供了迭代器。

---

## 二、迭代器的分类

C++ 标准把迭代器分为 5 类，能力从弱到强：

| 类别           | 支持操作                          | 典型容器                |
| -------------- | --------------------------------- | ----------------------- |
| 输入迭代器     | `*it`, `++it`, `==`               | `istream_iterator`      |
| 输出迭代器     | `*it = x`, `++it`                 | `ostream_iterator`      |
| 前向迭代器     | 可多次读、只能前进                | `forward_list`          |
| 双向迭代器     | 可前进、可后退 `--it`             | `list`, `map`, `set`    |
| 随机访问迭代器 | 可 `it + n`, `it[n]`, `it2 - it1` | `vector`, `deque`, 数组 |

**`unordered_map` 的迭代器是前向迭代器**：只能 `++`，不能 `--`，也不能 `it + n`。这就是为什么哈希表里不能像 `vector` 那样随意跳着访问。

---

## 三、`unordered_map` 迭代器的内部结构

`unordered_map` 底层是**哈希桶 + 链表**（每个桶挂一条链）。它的迭代器要能：

1. 在当前桶的链表里往后走；
2. 走到链表末尾时，跳到下一个非空桶的第一个节点。

所以一个 `unordered_map` 迭代器内部通常持有：

- 指向当前节点的指针；
- 指向整个哈希表（桶数组）的指针。

这样它才知道怎么“跨桶”跳转。

```cpp
unordered_map<string, int> m = {{"a", 1}, {"b", 2}};
auto it = m.begin();   // 指向某个桶里的第一个节点
++it;                  // 沿链表或跳到下一个桶
```

---

## 四、`it->first` 和 `it->second`

对于 `unordered_map`，每个元素其实是 `pair<const Key, Value>`。

```cpp
auto it = remaining.find(temp_word);
// it 指向的是 pair<const string, int>
it->first;    // 等价于 (*it).first，是 key（const string）
it->second;   // 等价于 (*it).second，是 value（int）
```

`->` 是 `(*it).` 的语法糖。因为 `*it` 得到的是 `pair`，用 `.` 访问成员；而迭代器本身是“指针”，所以用 `->`。

**注意 key 是 `const`**：你不能写 `it->first = "xxx"`，因为改 key 会破坏哈希表结构。

---

## 五、`end()` 是哨兵，不能解引用

`end()` **不指向任何元素**，它是一个“尾后标记”。

```cpp
auto it = m.find("不存在");
if (it == m.end()) {
    // 没找到
}
```

这就是为什么 `find` 失败时返回 `end()`。你不能解引用 `end()`，那和访问野指针一样是未定义行为。

```cpp
*it;          // it == end() 时 UB
it->second;   // 同样 UB
```

---

## 六、迭代器失效

**迭代器本质上是“指向容器内部某个位置”的句柄。当容器结构变化时，原来指向该位置的迭代器可能失效。**

不同容器的失效规则不同：

| 操作      | `vector`             | `unordered_map`                                  |
| --------- | -------------------- | ------------------------------------------------ |
| 插入元素  | 可能全部失效（扩容） | **只有 rehash 时失效**，其余情况其他迭代器仍有效 |
| 删除元素  | 被删元素之后的都失效 | **只有被删元素的迭代器失效**                     |
| `clear()` | 全部失效             | 全部失效                                         |

回到文章开头的例子。为什么 `find` 版本更安全：

```cpp
auto it = remaining.find(temp_word);
if (it == remaining.end() || it->second == 0) return false;
it->second--;   // 只修改 value，不改变哈希表结构 → 迭代器依然有效
```

而 `operator[]` 版本：

```cpp
hash_words[temp_word]    // 若 key 不存在，会插入 → 可能触发 rehash
hash_words[temp_word]--  // 重新查找
```

**插入可能触发 rehash，导致所有迭代器失效**。虽然代码里没保存迭代器所以不会崩，但哈希表结构变化本身就有开销。

---

## 七、`find` 返回迭代器的意义

```cpp
auto it = remaining.find(temp_word);
```

`find` 返回迭代器而不是 `bool` 或 `int`，是为了让你**既能判断是否存在，又能直接操作找到的元素**：

```cpp
if (it != remaining.end()) {
    it->second--;          // 直接改，无需再查找
    cout << it->first;     // 还能拿到 key
}
```

如果 `find` 只返回 `bool`，你要修改还得再查一次——这正是 `operator[]` 写法的问题：查了两次。

### `operator[]` 的隐藏代价

`unordered_map::operator[]` 的行为是：

> 如果 key 不存在，就**插入**一个 `{key, 默认值}` 的键值对，并返回默认值的引用。

也就是说，当你写 `hash_words[temp_word]` 而 `temp_word` 不在表里时，它会：

1. 计算 `temp_word` 的哈希值；
2. 在桶里查找；
3. **没找到 → 分配一个新节点，插入 `{temp_word, 0}`**；
4. 返回这个 0 的引用。

这个插入是有实际开销的：**内存分配 + 哈希表结构修改（可能触发 rehash）**。

在你的算法里，如果遇到大量非法单词，`operator[]` 会不断往表里塞无用节点，导致表膨胀、查找变慢，甚至反复 rehash。而 `find` 版本只查不插，表的大小始终不超过 `words` 中不同单词的数量。

---

## 八、迭代器 vs 下标 vs 指针

```cpp
// vector 三种遍历方式等价
for (int i = 0; i < v.size(); i++) cout << v[i];
for (auto it = v.begin(); it != v.end(); ++it) cout << *it;
for (int x : v) cout << x;   // 范围 for，底层就是迭代器
```

`unordered_map` **没有下标访问**（`operator[]` 是“按键取值”，不是“按位置取值”），所以只能靠迭代器遍历：

```cpp
for (auto it = m.begin(); it != m.end(); ++it) {
    cout << it->first << " : " << it->second << "\n";
}
// 或范围 for
for (const auto& [key, val] : m) {
    cout << key << " : " << val << "\n";
}
```

---

## 九、`const_iterator`

如果你不希望遍历时修改元素，用 `const_iterator`：

```cpp
for (auto it = m.cbegin(); it != m.cend(); ++it) {
    // it->second = 10;  // 编译错误
}
```

当参数写成 `const unordered_map<string,int>& hash_words` 时，`find` 返回的就是 `const_iterator`。这也是为什么用 `find` 语义更准确——它天然尊重 `const`。

---

## 十、回到实际代码

```cpp
bool isSubstring(const string& s, const unordered_map<string, int>& hash_words, ...) {
    unordered_map<string, int> remaining = hash_words;  // 局部副本，可修改
    for (...) {
        string temp_word = s.substr(k, word_len);
        auto it = remaining.find(temp_word);   // 前向迭代器，指向找到的节点
        if (it == remaining.end() || it->second == 0) return false;
        it->second--;   // 通过迭代器直接修改 value，不改变哈希结构
    }
    return true;
}
```

这里的迭代器生命周期只在本次循环内，`it->second--` 不改变表结构，所以迭代器始终有效。整个流程只做一次哈希、一次查找、一次修改，比 `operator[]` 的两次查找 + 潜在插入要快得多。

---

## 总结

> 迭代器是容器元素的“通用句柄”，`find` 返回迭代器让你一次查找就能定位并操作元素；`end()` 是哨兵，不能解引用；不同容器迭代器能力不同，且结构变化时可能失效。理解这三点，STL 就算入门了。

在实际编码中，优先使用 `find` 而不是 `operator[]` 做“只读判断 + 修改值”的操作，既能避免无意义插入，又能减少哈希计算次数，还能让代码更安全、更清晰。
