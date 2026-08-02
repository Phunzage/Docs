---
title: 链表反转的迭代与递归实现
tags:
  - 数据结构
  - 算法
  - 面试
createTime: 2025/07/30 10:00:00
permalink: /blog/data-structure/linkedlist-reverse/
---

> 面试官：请实现单链表的反转，要求写出迭代和递归两种解法。
>
> 这是数据结构与算法面试中**出现频率最高**的题目之一（LeetCode 206. 反转链表），也是后续很多复杂链表题（如 K 个一组反转、回文链表）的基础。今天我们就用三指针迭代和递归两种思路把它彻底讲透。

## 问题描述

给定单链表的头节点 `head`，反转链表，返回反转后的新头节点。

```text
输入：1 → 2 → 3 → 4 → 5 → NULL
输出：5 → 4 → 3 → 2 → 1 → NULL
```

::: tip 链表节点定义
本文所有代码基于如下节点结构体（数据域 `data` / `Val` + 指针域 `next` / `Next`）：

::: code-tabs
@tab:active common.h
```cpp
struct Node {
    int data;
    Node* next;
};
```
@tab Go
```go
// 链表节点结构体
type ListNode struct {
    Val  int       // 节点值
    Next *ListNode // 指向下一节点的指针
}
```
:::
:::

## 迭代法（三指针）

迭代法的核心思想是：**遍历链表的同时，逐个把每个节点的 `next` 指针掉转方向**。由于单链表只能单向访问，一旦修改了当前节点的 `next`，就会丢失原本的下一个节点，因此必须提前用一个指针把它保存下来——这就是"三指针"的由来。

### 指针含义

| 指针 | 作用 |
|------|------|
| `current` | 当前正在处理的节点 |
| `prev` | 当前节点的前一个节点（初始为 `NULL`） |
| `next` | 当前节点的下一个节点（提前保存，防止丢失） |

### ASCII 图解

```text
初始状态：
  NULL      1 ──▶ 2 ──▶ 3 ──▶ 4 ──▶ NULL
  ▲prev    ▲current

第 1 轮循环（先保存，再反转，最后移动）：
  next = current->next      // next 指向 2，防止丢失
  current->next = prev      // 1 的指针掉头指向 NULL
  prev = current            // prev 前移到 1
  current = next            // current 前移到 2

  NULL ◀── 1     2 ──▶ 3 ──▶ 4 ──▶ NULL
           ▲prev ▲current

第 2 轮循环后：
  NULL ◀── 1 ◀── 2     3 ──▶ 4 ──▶ NULL
                    ▲prev ▲current

…… 循环直到 current 为 NULL，此时 prev 指向原链表的尾节点：

  NULL ◀── 1 ◀── 2 ◀── 3 ◀── 4    prev
                                     ▲prev (新头节点)
```

### 函数参数说明

- `head`：当前链表的头节点指针
- 返回值：反转后的链表头节点指针（即原链表的尾节点）

### 操作原理

1. **初始化指针**：设置三个指针，`prev = NULL`，`current = head`
2. **遍历反转**：当 `current` 不为 `NULL` 时循环执行——
   - 先用 `next` 保存 `current` 的下一个节点
   - 将 `current->next` 指向 `prev`（掉转方向）
   - 三个指针整体向前移动一步
3. **更新头节点**：循环结束时，`current` 为 `NULL`，`prev` 正好是原尾节点，`head = prev`

### 关键理解点

- **三指针技巧**：`prev`、`current`、`next` 三个指针协同工作，`prev` 记录已反转部分，`current` 推进遍历，`next` 防止断链
- **顺序重要性**：必须先保存 `next`，再修改 `current->next`，否则原链表在当前位置之后的部分会全部丢失
- **终止条件**：当 `current` 为 `NULL` 时循环结束，此时 `prev` 恰好指向原链表的尾节点，它就是新头节点
- **完整性保持**：整个反转过程中链表始终完整连接，没有中间断裂状态

::: code-tabs
@tab:active C++
```cpp
Node* Reverse(Node* head) {
    // 设置三个节点：当前节点、前一个节点和后一个节点
    // 当前节点指向 head，前一个节点设置为 NULL
    Node* prev, * next, * current;
    prev = NULL;
    current = head;
    // 当前节点不为 NULL 时循环
    while (current != NULL) {
        // 保存下一个节点，防止修改指针后丢失
        next = current->next;
        // 核心：当前节点的下一个节点指向前一个节点 prev
        current->next = prev;
        // 移动：prev 和 current 分别后移
        prev = current;
        current = next;
    }
    // 此时 current 为 NULL，prev 为原尾节点（新头节点）
    head = prev;
    return head;
}
```
@tab Go
```go
// 反转链表（迭代法）
func reverse(head *ListNode) *ListNode {
    var prev *ListNode
    current := head
    for current != nil {
        next := current.Next // 先保存下一个节点
        current.Next = prev  // 掉转指针方向
        prev = current       // prev 前移
        current = next       // current 前移
    }
    return prev // prev 即新头节点
}
```
:::

## 递归法

递归法的思路和迭代法完全相反：迭代法是**从前往后**逐个反转，递归法是**先一路走到链表末尾**，再**从后往前**逐个反转指针。

### 思路拆解

1. **递归基**：当 `head->next` 为 `NULL`（到达尾节点）时，直接返回该节点——它将是反转后的新头节点
2. **递归深入**：对 `head->next` 的子链表继续调用反转，直到链表末尾
3. **回溯反转**：每一层返回时，执行 `head->next->next = head`，让当前节点的下一个节点"掉头"指向自己
4. **断开旧连接**：`head->next = NULL`，切断当前节点向后的旧指针
5. **新头节点传递**：递归返回的 `newHead` 从最深层一路原样传递到最外层，最终返回给调用者

```text
以 1 → 2 → 3 → 4 → NULL 为例：

递归深入：reverse(1) → reverse(2) → reverse(3) → reverse(4)
  reverse(4)：4->next 为 NULL，返回 4（新头节点）✅

回溯反转：
  reverse(3)：3->next->next = 3  即 4->next = 3
             3->next = NULL      即 3 → NULL
             结果：1 → 2 → 3 → NULL  和  4 → 3

  reverse(2)：2->next->next = 2  即 3->next = 2
             2->next = NULL
             结果：1 → 2 → NULL  和  4 → 3 → 2

  reverse(1)：1->next->next = 1  即 2->next = 1
             1->next = NULL
             最终结果：4 → 3 → 2 → 1 → NULL 🎉
```

### 函数参数说明

- `head`：当前链表的头节点指针
- 返回值：反转后的链表头节点指针（即原链表的尾节点）

### 操作原理

1. **递归基**：当到达尾节点时返回，该节点成为新头节点
2. **递归深入**：不断深入直到链表末尾
3. **回溯反转**：在递归返回过程中逐个反转节点指针方向
4. **指针调整**：将当前节点的下一个节点的 `next` 指向当前节点，形成反转

### 关键理解点

- **递归思维**：把大问题分解为小问题——先反转"去掉头节点后的子链表"，再把头节点接到子链表末尾
- **新头节点传递**：`newHead` 在递归过程中从最深层一直传递到最外层，中间不再改变
- **指针操作时机**：指针反转操作发生在**递归返回阶段**（回溯时），而不是深入阶段
- **栈空间使用**：递归调用使用系统栈，链表过长时可能栈溢出；同时 `head` 为空时需加判空保护（Go 版本已包含 `head == nil` 判断）

::: warning 关于递归的边界
递归版本在深入时依赖 `head->next` 访问下一个节点，因此**调用前必须保证链表非空**。Go 版本直接写成 `head == nil || head.Next == nil` 统一处理了空链表和单节点链表两种边界；C++ 版本若想更严谨，也可加上 `head == NULL` 的判空。
:::

::: code-tabs
@tab:active C++
```cpp
Node* ReverseRecursion(Node* head) {
    // head 在此处用来遍历链表
    // 当 head 的下一个节点是 NULL，即到达尾节点时，返回尾节点的地址
    if (head->next == NULL) {
        return head;
    }
    // 递归深入，保存尾节点（新头节点）到一个新变量中
    Node* newHead = ReverseRecursion(head->next);
    // 回溯反转：当前节点下一个节点的 next 指向当前节点
    head->next->next = head;
    // 断开当前节点向后的旧指针
    head->next = NULL;
    // 向上层返回新的头节点，中途 newHead 不改变
    return newHead;
}
```
@tab Go
```go
// 反转链表（递归法）
func reverseRecursion(head *ListNode) *ListNode {
    // 递归基：空链表或单节点链表直接返回
    if head == nil || head.Next == nil {
        return head
    }
    // 递归深入，newHead 始终是原尾节点
    newHead := reverseRecursion(head.Next)
    // 回溯反转：让下一个节点掉头指向当前节点
    head.Next.Next = head
    // 断开旧连接，防止成环
    head.Next = nil
    return newHead
}
```
:::

## 总结

两种方法都能在 `O(n)` 时间内完成反转，空间复杂度差别是核心权衡点：

| 对比维度 | 迭代法 | 递归法 |
|----------|--------|--------|
| 时间复杂度 | O(n) | O(n) |
| 空间复杂度 | **O(1)**，仅三个指针 | **O(n)**，递归调用栈 |
| 思路 | 从前往后，逐个掉转指针 | 从后往前，回溯时掉转指针 |
| 代码量 | 略多（循环 + 三指针） | 简洁（递归基 + 回溯） |
| 边界风险 | 指针顺序写错会断链 | 链表过长会栈溢出 |
| 面试建议 | ✅ 首选，空间最优 | 能写出体现对递归的理解 |

::: tip 面试小贴士
面试时建议**先写迭代法**（简单、无栈溢出风险），再补充递归法展示思路。一定要能口头讲清楚三指针的移动顺序——"先保存 next，再反转 current，最后整体前移"——这比背代码更重要。
:::

## 参考资料

- 原始笔记：[数据结构 · 链表（反转链表章节）](https://docs.phunzage.art/data-structure/1-Link/)
- LeetCode 206. 反转链表：https://leetcode.cn/problems/reverse-linked-list/
