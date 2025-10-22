---
title: Link01
createTime: 2025/10/22 18:54:06
permalink: /data-structure/kvnqzqh4/
---
## 链表头部插入节点

这是一个测试

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

@tab InsertAtHead.go
```go
func Insert(root *BstNode, data int) *BstNode {
	if root == nil {
		root = GetBstNode(data)
	} else if data <= root.data {
		root.left = Insert(root.left, data)
	} else {
		root.right = Insert(root.right, data)
	}
	return root
}
```
:::

