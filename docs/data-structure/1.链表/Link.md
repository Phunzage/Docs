---
title: 链表
createTime: 2025/10/22 18:54:06
permalink: /data-structure/kvnqzqh4/
---

链表是一种线性的数据结构，其中的每个元素都是一个节点对象，各个节点通过“引用”相连接。引用记录了下一个节点的内存地址，通过它可以从当前节点访问到下一个节点。

## 构造链表结构体
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

## 反转节点

### 迭代方法

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

