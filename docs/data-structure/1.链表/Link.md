---
title: 链表
createTime: 2025/10/22 18:54:06
permalink: /data-structure/kvnqzqh4/
---

链表是一种线性的数据结构，其中的每个元素都是一个节点对象，各个节点通过“引用”相连接。引用记录了下一个节点的内存地址，通过它可以从当前节点访问到下一个节点。

## 构造链表结构体

链表节点结构体由数据部分 **data** 和指向下一个节点的节点部分 **next** 构成,其中节点部分存放的是下一个节点的地址

::: code-tabs
@tab:active common.h
```cpp
struct Node {
    int data;
    Node* next;
};
```
:::

## 链表头部插入节点

链表头部插入是一种基础操作，用于在链表的起始位置添加新节点。通过此操作，新节点会成为链表的新头节点。

### 函数参数说明
- `head`：当前链表的头节点指针
- `x`：要插入节点中存储的数值
- 返回值：插入后的新头节点指针

### 操作原理
1. **创建新节点**：在堆内存中分配新节点 `temp`，并设置其数据值为 `x`
2. **连接节点**：将新节点的 `next` 指针指向原头节点
3. **更新头节点**：将新节点设为链表的新头节点

### 关键理解点
- **统一处理逻辑**：无论链表是否为空，操作逻辑都保持一致
  - 空链表：`head` 为 `NULL`，新节点的 `next` 自然指向 `NULL`
  - 非空链表：新节点的 `next` 指向原头节点，形成连接
- **内存管理**：使用 `new` 在堆中动态分配节点内存
- **指针更新**：必须返回新的头节点指针，因为头节点已改变
::: code-tabs
@tab:active InsertAtHead.cpp
```cpp
Node* InsertAtHead(Node* head, int x) {
	Node* temp = new Node();
	temp->data = x;
	/*
	==第一种情况，链表为空
	temp->next = NULL;
	temp = head;
	==第二种情况，链表不为空
	if(head != NULL) temp->next = head;
	head = temp;
	*/
	//由于链表为空时，head = NULL，故可以整合为：
	temp->next = head;
	head = temp;
	return head;
}
```
:::

## 任意位置插入节点

在链表任意位置插入节点是一种灵活的操作，允许在指定位置添加新节点，从而更精确地控制链表结构。

### 函数参数说明
- `head`：当前链表的头节点指针
- `data`：要插入节点中存储的数值
- `n`：要插入的位置（从1开始计数）
- 返回值：插入后的链表头节点指针

### 操作原理
1. **创建新节点**：在堆内存中分配新节点 `temp1`，设置数据值并初始化 `next` 为 `NULL`
2. **位置判断**：
   - 位置1（头部）：直接调用头部插入逻辑
   - 其他位置：遍历找到插入位置的前一个节点
3. **节点连接**：调整指针关系，将新节点插入到指定位置

### 关键理解点
- **位置编号**：链表位置通常从1开始计数，与数组索引不同
- **边界处理**：头部插入需要特殊处理，因为涉及头指针更新
- **遍历技巧**：使用 `n-2` 的循环次数来定位到插入位置的前一个节点
- **指针操作顺序**：必须先将新节点指向后续节点，再让前驱节点指向新节点

::: code-tabs
@tab:active InsertAtPosition.cpp
```cpp
Node* InsertAtPosition(Node* head, int data, int n) {
	Node* temp1 = new Node();
	temp1->data = data;
	temp1->next = NULL;
	//当插入位置是头部时
	//新定义的temp1节点的下一个节点是头节点
	//再把头节点重新赋值为新节点temp1
	if (n == 1) {
		temp1->next = head;
		head = temp1;
		return head;
	}
	//当插入位置不是头部时
	//新定义一个节点temp2，用于指向要插入位置的前一个节点
	Node* temp2 = head;
	for (int i = 0; i < n - 2; i++) {	//头部插入已判断，temp2默认指向第一个节点故在第二个位置插入也不需要判断
		temp2 = temp2->next;
	}
	//核心
	//新节点temp1的下一个节点是temp2的下一个节点
	//temp2的下一个节点再是temp1
	temp1->next = temp2->next;
	temp2->next = temp1;
	return head;
}
```
:::

## 任意位置删除节点

删除指定位置节点是链表的常见操作，用于动态调整链表内容并释放内存资源。

### 函数参数说明
- `head`：当前链表的头节点指针
- `n`：要删除的位置（从1开始计数）
- 返回值：删除后的链表头节点指针

### 操作原理
1. **边界检查**：确保链表不为空
2. **位置判断**：
   - 位置1（头部）：直接更新头指针并释放原头节点
   - 其他位置：遍历找到要删除节点的前一个节点
3. **节点移除**：调整指针关系，跳过要删除的节点，然后释放内存

### 关键理解点
- **内存管理**：使用 `delete` 释放动态分配的节点内存，防止内存泄漏
- **头节点处理**：删除头节点需要特殊处理，因为涉及头指针更新
- **遍历定位**：使用 `n-2` 的循环次数定位到删除位置的前一个节点
- **指针重连**：必须先将前驱节点指向被删节点的后继，再释放被删节点

::: code-tabs
@tab:active Move.cpp
```cpp
Node* Move(Node* head, int n) {
	Node* temp1 = head;
	if (temp1 != NULL) {
		//temp1指向头节点
		//要删除第一个节点时，头节目的指向temp1的下一个节点，释放temp1
		if (n == 1) {
			head = temp1->next;
			delete(temp1);
			return head;
		}
		//要删除其他节点时,新建temp2指向temp1的下一个节点
		//删除头节点已判断,temp2默认指向temp1的下一个节点,故删除第二个节点不用判断
		for (int i = 0; i < n - 2; i++) {
			temp1 = temp1->next;
		}
		Node* temp2 = temp1->next;
		//删除核心代码
		//temp1的下一个节点是temp2的下一个节点,释放temp2节点
		temp1->next = temp2->next;
		delete(temp2);
	}
	return head;
}
```
:::

## 反转链表

反转链表是将链表节点顺序完全颠倒的操作，有两种实现方式：迭代法和递归法。

### 迭代方法

#### 函数参数说明
- `head`：当前链表的头节点指针
- 返回值：反转后的链表头节点指针

#### 操作原理
1. **初始化指针**：设置三个指针分别指向当前节点、前一个节点和后一个节点
2. **遍历反转**：逐个节点调整指针方向，将当前节点的 `next` 指向前一个节点
3. **指针移动**：每次循环后向前移动三个指针的位置
4. **更新头节点**：遍历完成后，原尾节点成为新头节点

#### 关键理解点
- **三指针技巧**：使用 `prev`、`current`、`next` 三个指针协同工作
- **顺序重要性**：必须先保存下一个节点，再修改当前节点的指针
- **终止条件**：当 `current` 为 `NULL` 时，`prev` 正好指向原链表的尾节点
- **完整性保持**：在整个反转过程中，链表始终保持完整连接

::: code-tabs
@tab:active Reverse.cpp
```cpp
Node* Reverse(Node* head) {
	//设置三个节点，当前节点，前一个结点和后一个节点
	//当前结点指向head，前一个结点设置为NULL
	Node* prev, * next, * current;
	prev = NULL;
	current = head;
	//当前节点不为NULL时
	while (current != NULL) {
		//设置
		//下一个节点next，设置为当前节点的下一个节点
		next = current->next;
		//核心
		//当前节点的下一个节点是前一个节点prev
		current->next = prev;
		//移动
		//前一个结点后移
		//当前节点后移
		prev = current;
		current = next;
	}
	//此时current为NULL，prev为最后一个节点，head指向prev
	head = prev;
	return head;
}
```
:::

### 递归方法

#### 函数参数说明
- `head`：当前链表的头节点指针
- 返回值：反转后的链表头节点指针

#### 操作原理
1. **递归基**：当到达尾节点时返回，该节点成为新头节点
2. **递归深入**：不断深入直到链表末尾
3. **回溯反转**：在递归返回过程中逐个反转节点指针方向
4. **指针调整**：将当前节点的下一个节点的 `next` 指向当前节点，形成反转

#### 关键理解点
- **递归思维**：将大问题分解为小问题，先处理子链表再处理当前节点
- **新头节点传递**：在递归过程中，新头节点从最深层一直传递到最外层
- **指针操作时机**：在递归返回阶段执行指针反转操作
- **栈空间使用**：递归调用使用系统栈，需要注意链表长度避免栈溢出

::: code-tabs
@tab:active ReverseRecursion.cpp
```cpp
Node* ReverseRecursion(Node* head) {
	//head在；此处用来遍历链表
	//当head的下一个节点是NULL，即到达尾节点时，返回尾节点的地址
	if (head->next == NULL) {
		return head;
	}
	//回到上一层调用，保存尾节点到一个新变量中
	//当前层中，（head节点的下一个节点）的下一个节点为head
	//head的下一个节点为NULL
	//向上层返回新的头节点，中途新的头节点不改变
	Node* newHead = ReverseRecursion(head->next);
	head->next->next = head;
	head->next = NULL;
	return newHead;
}
```
:::

