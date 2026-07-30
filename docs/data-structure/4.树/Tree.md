---
title: 树
createTime: 2025/11/18 15:14:15
permalink: /data-structure/tree/
---

# 二叉搜索树（Binary Search Tree）

## 一、树的基本术语

树是一种**非线性**数据结构，由节点（Node）和边（Edge）组成。以下是树的核心术语：

| 术语 | 英文 | 定义 |
|------|------|------|
| **根节点** | Root | 树的最顶层节点，没有父节点 |
| **叶子节点** | Leaf | 没有子节点的节点 |
| **父节点** | Parent | 某个节点的直接上级节点 |
| **子节点** | Child | 某个节点的直接下级节点 |
| **兄弟节点** | Sibling | 共享同一个父节点的节点 |
| **祖先** | Ancestor | 从根节点到该节点路径上的所有节点 |
| **子孙** | Descendant | 该节点下方子树中的所有节点 |
| **高度** | Height | 从节点到最远叶子节点的边数（树根的高度即树高） |
| **深度** | Depth | 从根节点到该节点的边数 |
| **层** | Level | 根节点在第 1 层，子节点依次递增 |
| **子树** | Subtree | 树中任一节点及其所有后代构成一棵子树 |
| **度** | Degree | 节点拥有的子树的个数 |

> 注意：有些教材将高度定义为节点数而非边数，两者相差 1。本文采用**边数**定义。

## 二、二叉搜索树（BST）的性质

二叉搜索树是二叉树的一种，它满足以下 **BST 性质**：

1. **左子树**中所有节点的值 **<** 根节点的值
2. **右子树**中所有节点的值 **>** 根节点的值
3. 左、右子树也分别是二叉搜索树（递归定义）
4. 通常不允许重复节点（或规定重复时放在左/右子树）

```
        8
       / \
      3   10
     / \    \
    1   6    14
       / \   /
      4   7 13
```

> 上图中，对任意节点，其左子树所有值均小于它，右子树所有值均大于它。

二叉搜索树的核心优势在于：**查找、插入、删除的平均时间复杂度为 O(log n)**，与二分查找类似。

---

## 三、节点结构定义

### C++ 版本

```cpp title="C++ — BstNode 结构"
struct BstNode {
    int data;
    BstNode* left;
    BstNode* right;
    
    BstNode(int val) : data(val), left(nullptr), right(nullptr) {}
};
```

### Go 版本

```go title="Go — TreeNode 结构"
type TreeNode struct {
    Data  int
    Left  *TreeNode
    Right *TreeNode
}

func NewTreeNode(val int) *TreeNode {
    return &TreeNode{Data: val, Left: nil, Right: nil}
}
```

---

## 四、插入操作（Insert）

插入操作遵循 BST 性质，递归地找到合适的位置：

- 若树为空，直接创建新节点
- 若待插值 ≤ 当前节点值，递归插入左子树
- 否则递归插入右子树

### C++ 实现（递归）

```cpp title="C++ — Insert"
BstNode* Insert(BstNode* root, int data) {
    if (root == nullptr) {
        return new BstNode(data);
    }
    if (data <= root->data) {
        root->left = Insert(root->left, data);
    } else {
        root->right = Insert(root->right, data);
    }
    return root;
}
```

### Go 实现（递归）

```go title="Go — Insert"
func Insert(root *TreeNode, data int) *TreeNode {
    if root == nil {
        return NewTreeNode(data)
    }
    if data <= root.Data {
        root.Left = Insert(root.Left, data)
    } else {
        root.Right = Insert(root.Right, data)
    }
    return root
}
```

---

## 五、搜索操作（Search）

搜索从根节点开始，利用 BST 性质逐层缩小范围：

- 目标值 == 当前节点 → 找到
- 目标值 < 当前节点 → 搜索左子树
- 目标值 > 当前节点 → 搜索右子树
- 到达空节点 → 不存在

### C++ 实现

```cpp title="C++ — Search"
bool Search(BstNode* root, int data) {
    if (root == nullptr) return false;
    if (root->data == data) return true;
    if (data < root->data) {
        return Search(root->left, data);
    } else {
        return Search(root->right, data);
    }
}
```

### Go 实现

```go title="Go — Search"
func Search(root *TreeNode, data int) bool {
    if root == nil {
        return false
    }
    if root.Data == data {
        return true
    }
    if data < root.Data {
        return Search(root.Left, data)
    }
    return Search(root.Right, data)
}
```

---

## 六、查找最小值与最大值

BST 中最小的节点在最左边，最大的节点在最右边。

### 6.1 FindMin（迭代 + 递归）

#### C++ 版本

```cpp title="C++ — FindMin (迭代)"
int FindMin(BstNode* root) {
    if (root == nullptr) {
        throw std::runtime_error("Tree is empty");
    }
    while (root->left != nullptr) {
        root = root->left;
    }
    return root->data;
}
```

```cpp title="C++ — FindMin (递归)"
int FindMinRecursive(BstNode* root) {
    if (root == nullptr) {
        throw std::runtime_error("Tree is empty");
    }
    if (root->left == nullptr) {
        return root->data;
    }
    return FindMinRecursive(root->left);
}
```

#### Go 版本

```go title="Go — FindMin (迭代)"
func FindMin(root *TreeNode) int {
    if root == nil {
        panic("Tree is empty")
    }
    for root.Left != nil {
        root = root.Left
    }
    return root.Data
}
```

```go title="Go — FindMin (递归)"
func FindMinRecursive(root *TreeNode) int {
    if root == nil {
        panic("Tree is empty")
    }
    if root.Left == nil {
        return root.Data
    }
    return FindMinRecursive(root.Left)
}
```

### 6.2 FindMax（迭代 + 递归）

#### C++ 版本

```cpp title="C++ — FindMax (迭代)"
int FindMax(BstNode* root) {
    if (root == nullptr) {
        throw std::runtime_error("Tree is empty");
    }
    while (root->right != nullptr) {
        root = root->right;
    }
    return root->data;
}
```

```cpp title="C++ — FindMax (递归)"
int FindMaxRecursive(BstNode* root) {
    if (root == nullptr) {
        throw std::runtime_error("Tree is empty");
    }
    if (root->right == nullptr) {
        return root->data;
    }
    return FindMaxRecursive(root->right);
}
```

#### Go 版本

```go title="Go — FindMax (迭代)"
func FindMax(root *TreeNode) int {
    if root == nil {
        panic("Tree is empty")
    }
    for root.Right != nil {
        root = root.Right
    }
    return root.Data
}
```

```go title="Go — FindMax (递归)"
func FindMaxRecursive(root *TreeNode) int {
    if root == nil {
        panic("Tree is empty")
    }
    if root.Right == nil {
        return root.Data
    }
    return FindMaxRecursive(root.Right)
}
```

---

## 七、树的高度（FindHeight）

树的高度 = 根节点到最远叶子节点的**边数**。

- 空树高度为 -1
- 叶子节点高度为 0
- 非叶子节点高度 = 1 + max(左子树高度, 右子树高度)

```
        8           高度 = 3
       / \
      3   10        高度 = 1
     / \    \
    1   6    14     高度 = 0
       / \
      4   7         高度 = 0
```

### C++ 实现

```cpp title="C++ — FindHeight"
#include <algorithm>

int FindHeight(BstNode* root) {
    if (root == nullptr) return -1;  // 空树
    int leftHeight = FindHeight(root->left);
    int rightHeight = FindHeight(root->right);
    return std::max(leftHeight, rightHeight) + 1;
}
```

### Go 实现

```go title="Go — FindHeight"
func FindHeight(root *TreeNode) int {
    if root == nil {
        return -1
    }
    leftHeight := FindHeight(root.Left)
    rightHeight := FindHeight(root.Right)
    if leftHeight > rightHeight {
        return leftHeight + 1
    }
    return rightHeight + 1
}
```

---

## 八、树的遍历（Traversal）

### 8.1 前序遍历（Preorder: 根 → 左 → 右）

**应用：** 复制一棵树、序列化输出前缀表达式

```
        8
       / \
      3   10
     / \    \
    1   6    14

前序结果: 8 → 3 → 1 → 6 → 10 → 14
```

#### C++ 实现

```cpp title="C++ — Preorder"
#include <iostream>

void Preorder(BstNode* root) {
    if (root == nullptr) return;
    std::cout << root->data << " ";
    Preorder(root->left);
    Preorder(root->right);
}
```

#### Go 实现

```go title="Go — Preorder"
import "fmt"

func Preorder(root *TreeNode) {
    if root == nil {
        return
    }
    fmt.Print(root.Data, " ")
    Preorder(root.Left)
    Preorder(root.Right)
}
```

### 8.2 中序遍历（Inorder: 左 → 根 → 右）

**应用：** 输出 BST 的升序序列（BST 的中序遍历是严格递增的）

```
中序结果: 1 → 3 → 6 → 8 → 10 → 14
```

#### C++ 实现

```cpp title="C++ — Inorder"
void Inorder(BstNode* root) {
    if (root == nullptr) return;
    Inorder(root->left);
    std::cout << root->data << " ";
    Inorder(root->right);
}
```

#### Go 实现

```go title="Go — Inorder"
func Inorder(root *TreeNode) {
    if root == nil {
        return
    }
    Inorder(root.Left)
    fmt.Print(root.Data, " ")
    Inorder(root.Right)
}
```

### 8.3 后序遍历（Postorder: 左 → 右 → 根）

**应用：** 删除整棵树（先删子节点）、后缀表达式求值

```
后序结果: 1 → 6 → 3 → 14 → 10 → 8
```

#### C++ 实现

```cpp title="C++ — Postorder"
void Postorder(BstNode* root) {
    if (root == nullptr) return;
    Postorder(root->left);
    Postorder(root->right);
    std::cout << root->data << " ";
}
```

#### Go 实现

```go title="Go — Postorder"
func Postorder(root *TreeNode) {
    if root == nil {
        return
    }
    Postorder(root.Left)
    Postorder(root.Right)
    fmt.Print(root.Data, " ")
}
```

### 8.4 层序遍历（Level Order / BFS）

使用队列，逐层从左到右遍历。

```
层序结果: 8 → 3 → 10 → 1 → 6 → 14
```

#### C++ 实现

```cpp title="C++ — LevelOrder (队列 + 迭代)"
#include <queue>

void LevelOrder(BstNode* root) {
    if (root == nullptr) return;
    std::queue<BstNode*> q;
    q.push(root);
    
    while (!q.empty()) {
        BstNode* current = q.front();
        q.pop();
        std::cout << current->data << " ";
        
        if (current->left != nullptr)
            q.push(current->left);
        if (current->right != nullptr)
            q.push(current->right);
    }
}
```

#### Go 实现

```go title="Go — LevelOrder (队列 + 迭代)"
type Queue []*TreeNode

func (q *Queue) Push(n *TreeNode) { *q = append(*q, n) }
func (q *Queue) Pop() *TreeNode {
    if len(*q) == 0 { return nil }
    n := (*q)[0]
    *q = (*q)[1:]
    return n
}

func LevelOrder(root *TreeNode) {
    if root == nil { return }
    var q Queue
    q.Push(root)
    for len(q) > 0 {
        current := q.Pop()
        fmt.Print(current.Data, " ")
        if current.Left != nil { q.Push(current.Left) }
        if current.Right != nil { q.Push(current.Right) }
    }
}
```

---

## 九、删除节点（Delete）

删除节点是 BST 最复杂的操作，需分三种情况考虑。

### 三种情况

**情况 1：删除叶子节点** — 直接删除父节点的对应指针置空。

```
删除 1:       8             8
             / \           / \
            3   10   →    3   10
           /     \       /     \
          1      14     ∅      14
```

**情况 2：删除只有一个子节点的节点** — 让父节点直接指向该节点的子节点。

```
删除 10:      8             8
             / \           / \
            3   10   →    3   14
           / \    \      / \
          1   6    14   1   6
```

**情况 3：删除有两个子节点的节点** — 找到**右子树的最小值（中序后继）** 或**左子树的最大值（中序前驱）**，用该值替换当前节点，然后递归删除中序后继/前驱。

```
删除 3:       8             8
             / \           / \
            3   10   →    4   10
           / \    \      / \    \
          1   6    14   1   6    14
              /
             4
```

> 上例中，节点 3 的右子树最小值为 4（中序后继），将 4 复制到节点 3，然后删除原节点 4。

### C++ 实现

```cpp title="C++ — Delete"
BstNode* FindMinNode(BstNode* root) {
    while (root->left != nullptr) root = root->left;
    return root;
}

BstNode* Delete(BstNode* root, int data) {
    if (root == nullptr) return nullptr;
    
    // 先找到要删除的节点
    if (data < root->data) {
        root->left = Delete(root->left, data);
    } else if (data > root->data) {
        root->right = Delete(root->right, data);
    } else {
        // 找到目标节点，处理三种情况
        // 情况 1 & 2: 叶子节点或只有一个子节点
        if (root->left == nullptr) {
            BstNode* temp = root->right;
            delete root;
            return temp;
        } else if (root->right == nullptr) {
            BstNode* temp = root->left;
            delete root;
            return temp;
        }
        // 情况 3: 有两个子节点 → 找右子树最小值
        BstNode* successor = FindMinNode(root->right);
        root->data = successor->data;
        root->right = Delete(root->right, successor->data);
    }
    return root;
}
```

### Go 实现

```go title="Go — Delete"
func findMinNode(root *TreeNode) *TreeNode {
    for root.Left != nil {
        root = root.Left
    }
    return root
}

func Delete(root *TreeNode, data int) *TreeNode {
    if root == nil {
        return nil
    }
    if data < root.Data {
        root.Left = Delete(root.Left, data)
    } else if data > root.Data {
        root.Right = Delete(root.Right, data)
    } else {
        // 找到目标节点
        if root.Left == nil {
            return root.Right
        } else if root.Right == nil {
            return root.Left
        }
        // 有两个子节点：用中序后继替换
        successor := findMinNode(root.Right)
        root.Data = successor.Data
        root.Right = Delete(root.Right, successor.Data)
    }
    return root
}
```

---

## 十、验证二叉搜索树（isBST）

给定一棵二叉树，判断它是否是有效的 BST。不能简单地只检查当前节点与左右子节点的值——**必须确保整个左子树都小于根节点，整个右子树都大于根节点**。

### 方法一：传递范围约束（推荐）

递归时传递允许的值范围 `(minVal, maxVal)`，初始为 `(-∞, +∞)`。

```
        8 (-∞, +∞)
       / \
(-∞,8)3   10(8,+∞)
     / \      \
(-∞,3)1   6(3,8)  14(10,+∞)
         /
      (3,6)4
```

#### C++ 实现

```cpp title="C++ — isBST (范围法)"
#include <climits>

bool IsBstUtil(BstNode* root, int minVal, int maxVal) {
    if (root == nullptr) return true;
    if (root->data < minVal || root->data > maxVal) return false;
    return IsBstUtil(root->left, minVal, root->data - 1)
        && IsBstUtil(root->right, root->data + 1, maxVal);
}

bool IsBinarySearchTree(BstNode* root) {
    return IsBstUtil(root, INT_MIN, INT_MAX);
}
```

#### Go 实现

```go title="Go — isBST (范围法)"
import "math"

func isBstUtil(root *TreeNode, minVal, maxVal int) bool {
    if root == nil {
        return true
    }
    if root.Data < minVal || root.Data > maxVal {
        return false
    }
    return isBstUtil(root.Left, minVal, root.Data-1) &&
        isBstUtil(root.Right, root.Data+1, maxVal)
}

func IsBST(root *TreeNode) bool {
    return isBstUtil(root, math.MinInt, math.MaxInt)
}
```

### 方法二：中序遍历法

利用 BST 中序遍历为升序的性质，记录前驱节点的值并检查。

```go title="Go — isBST (中序遍历法)"
func IsBSTInorder(root *TreeNode) bool {
    var prev *int // 记录前驱节点值
    var inorder func(*TreeNode) bool
    inorder = func(node *TreeNode) bool {
        if node == nil {
            return true
        }
        if !inorder(node.Left) {
            return false
        }
        if prev != nil && node.Data <= *prev {
            return false
        }
        prev = &node.Data
        return inorder(node.Right)
    }
    return inorder(root)
}
```

---

## 十一、完整示例

### C++ 完整代码

```cpp title="C++ — BST 完整示例"
#include <iostream>
#include <queue>
#include <algorithm>
#include <climits>
#include <stdexcept>

struct BstNode {
    int data;
    BstNode* left;
    BstNode* right;
    BstNode(int val) : data(val), left(nullptr), right(nullptr) {}
};

BstNode* Insert(BstNode* root, int data) {
    if (root == nullptr) return new BstNode(data);
    if (data <= root->data)
        root->left = Insert(root->left, data);
    else
        root->right = Insert(root->right, data);
    return root;
}

bool Search(BstNode* root, int data) {
    if (root == nullptr) return false;
    if (root->data == data) return true;
    return data < root->data
        ? Search(root->left, data)
        : Search(root->right, data);
}

int FindMin(BstNode* root) {
    if (root == nullptr) throw std::runtime_error("Empty tree");
    while (root->left) root = root->left;
    return root->data;
}

int FindMax(BstNode* root) {
    if (root == nullptr) throw std::runtime_error("Empty tree");
    while (root->right) root = root->right;
    return root->data;
}

int FindHeight(BstNode* root) {
    if (root == nullptr) return -1;
    return std::max(FindHeight(root->left), FindHeight(root->right)) + 1;
}

void Preorder(BstNode* root) {
    if (root == nullptr) return;
    std::cout << root->data << " ";
    Preorder(root->left);
    Preorder(root->right);
}

void Inorder(BstNode* root) {
    if (root == nullptr) return;
    Inorder(root->left);
    std::cout << root->data << " ";
    Inorder(root->right);
}

void Postorder(BstNode* root) {
    if (root == nullptr) return;
    Postorder(root->left);
    Postorder(root->right);
    std::cout << root->data << " ";
}

void LevelOrder(BstNode* root) {
    if (root == nullptr) return;
    std::queue<BstNode*> q;
    q.push(root);
    while (!q.empty()) {
        BstNode* cur = q.front(); q.pop();
        std::cout << cur->data << " ";
        if (cur->left) q.push(cur->left);
        if (cur->right) q.push(cur->right);
    }
}

BstNode* FindMinNode(BstNode* root) {
    while (root->left) root = root->left;
    return root;
}

BstNode* Delete(BstNode* root, int data) {
    if (root == nullptr) return nullptr;
    if (data < root->data)
        root->left = Delete(root->left, data);
    else if (data > root->data)
        root->right = Delete(root->right, data);
    else {
        if (root->left == nullptr) {
            BstNode* t = root->right;
            delete root;
            return t;
        } else if (root->right == nullptr) {
            BstNode* t = root->left;
            delete root;
            return t;
        }
        BstNode* s = FindMinNode(root->right);
        root->data = s->data;
        root->right = Delete(root->right, s->data);
    }
    return root;
}

bool IsBstUtil(BstNode* r, int minV, int maxV) {
    if (r == nullptr) return true;
    if (r->data < minV || r->data > maxV) return false;
    return IsBstUtil(r->left, minV, r->data - 1)
        && IsBstUtil(r->right, r->data + 1, maxV);
}

bool IsBST(BstNode* root) {
    return IsBstUtil(root, INT_MIN, INT_MAX);
}

int main() {
    BstNode* root = nullptr;
    root = Insert(root, 8);
    root = Insert(root, 3);
    root = Insert(root, 10);
    root = Insert(root, 1);
    root = Insert(root, 6);
    root = Insert(root, 14);
    root = Insert(root, 4);
    root = Insert(root, 7);
    root = Insert(root, 13);

    std::cout << "Inorder: ";  Inorder(root);  std::cout << "\n";
    std::cout << "Height:  " << FindHeight(root) << "\n";
    std::cout << "Min:     " << FindMin(root) << "\n";
    std::cout << "Max:     " << FindMax(root) << "\n";
    std::cout << "isBST:   " << (IsBST(root) ? "true" : "false") << "\n";
    std::cout << "Search 6: " << (Search(root, 6) ? "found" : "not found") << "\n";

    root = Delete(root, 3);
    std::cout << "After delete(3): ";  Inorder(root);  std::cout << "\n";
    return 0;
}
```

### Go 完整代码

```go title="Go — BST 完整示例"
package main

import (
    "fmt"
    "math"
)

type TreeNode struct {
    Data  int
    Left  *TreeNode
    Right *TreeNode
}

func NewTreeNode(val int) *TreeNode {
    return &TreeNode{Data: val, Left: nil, Right: nil}
}

func Insert(root *TreeNode, data int) *TreeNode {
    if root == nil {
        return NewTreeNode(data)
    }
    if data <= root.Data {
        root.Left = Insert(root.Left, data)
    } else {
        root.Right = Insert(root.Right, data)
    }
    return root
}

func Search(root *TreeNode, data int) bool {
    if root == nil {
        return false
    }
    if root.Data == data {
        return true
    }
    if data < root.Data {
        return Search(root.Left, data)
    }
    return Search(root.Right, data)
}

func FindMin(root *TreeNode) int {
    if root == nil {
        panic("Empty tree")
    }
    for root.Left != nil {
        root = root.Left
    }
    return root.Data
}

func FindMax(root *TreeNode) int {
    if root == nil {
        panic("Empty tree")
    }
    for root.Right != nil {
        root = root.Right
    }
    return root.Data
}

func FindHeight(root *TreeNode) int {
    if root == nil {
        return -1
    }
    lh := FindHeight(root.Left)
    rh := FindHeight(root.Right)
    if lh > rh {
        return lh + 1
    }
    return rh + 1
}

func Preorder(root *TreeNode) {
    if root == nil {
        return
    }
    fmt.Print(root.Data, " ")
    Preorder(root.Left)
    Preorder(root.Right)
}

func Inorder(root *TreeNode) {
    if root == nil {
        return
    }
    Inorder(root.Left)
    fmt.Print(root.Data, " ")
    Inorder(root.Right)
}

func Postorder(root *TreeNode) {
    if root == nil {
        return
    }
    Postorder(root.Left)
    Postorder(root.Right)
    fmt.Print(root.Data, " ")
}

type Queue []*TreeNode

func (q *Queue) Push(n *TreeNode) { *q = append(*q, n) }
func (q *Queue) Pop() *TreeNode {
    if len(*q) == 0 {
        return nil
    }
    n := (*q)[0]
    *q = (*q)[1:]
    return n
}

func LevelOrder(root *TreeNode) {
    if root == nil {
        return
    }
    var q Queue
    q.Push(root)
    for len(q) > 0 {
        cur := q.Pop()
        fmt.Print(cur.Data, " ")
        if cur.Left != nil {
            q.Push(cur.Left)
        }
        if cur.Right != nil {
            q.Push(cur.Right)
        }
    }
}

func findMinNode(root *TreeNode) *TreeNode {
    for root.Left != nil {
        root = root.Left
    }
    return root
}

func Delete(root *TreeNode, data int) *TreeNode {
    if root == nil {
        return nil
    }
    if data < root.Data {
        root.Left = Delete(root.Left, data)
    } else if data > root.Data {
        root.Right = Delete(root.Right, data)
    } else {
        if root.Left == nil {
            return root.Right
        } else if root.Right == nil {
            return root.Left
        }
        succ := findMinNode(root.Right)
        root.Data = succ.Data
        root.Right = Delete(root.Right, succ.Data)
    }
    return root
}

func isBstUtil(root *TreeNode, minVal, maxVal int) bool {
    if root == nil {
        return true
    }
    if root.Data < minVal || root.Data > maxVal {
        return false
    }
    return isBstUtil(root.Left, minVal, root.Data-1) &&
        isBstUtil(root.Right, root.Data+1, maxVal)
}

func IsBST(root *TreeNode) bool {
    return isBstUtil(root, math.MinInt, math.MaxInt)
}

func main() {
    var root *TreeNode
    for _, v := range []int{8, 3, 10, 1, 6, 14, 4, 7, 13} {
        root = Insert(root, v)
    }

    fmt.Print("Inorder:  "); Inorder(root); fmt.Println()
    fmt.Println("Height:  ", FindHeight(root))
    fmt.Println("Min:     ", FindMin(root))
    fmt.Println("Max:     ", FindMax(root))
    fmt.Println("isBST:   ", IsBST(root))
    fmt.Println("Search 6:", Search(root, 6))

    root = Delete(root, 3)
    fmt.Print("After delete(3): "); Inorder(root); fmt.Println()
}
```

---

## 十二、复杂度和应用

### 时间复杂度

| 操作 | 平均情况 | 最坏情况 |
|------|----------|----------|
| **Search** | O(log n) = O(h) | O(n) |
| **Insert** | O(log n) = O(h) | O(n) |
| **Delete** | O(log n) = O(h) | O(n) |
| **FindMin / FindMax** | O(log n) = O(h) | O(n) |
| **FindHeight** | O(n) | O(n) |
| **Preorder / Inorder / Postorder** | O(n) | O(n) |
| **LevelOrder (BFS)** | O(n) | O(n) |

> 其中 h 为树的高度。平衡二叉树中 h = O(log n)；退化为链表的 BST 中 h = O(n)。

### 空间复杂度

- 递归操作：O(h) — 递归调用栈深度
- 迭代遍历：O(1)（栈/队列辅助除外）
- BFS 层序遍历：O(n) — 队列最宽层

### 实际应用

| 应用场景 | 说明 |
|----------|------|
| **数据库索引** | B 树 / B+ 树基于 BST 思想，优化磁盘 I/O |
| **符号表** | 编译器中的符号表管理 |
| **优先级队列** | 结合 BST 实现有序集合 |
| **表达式解析** | 表达式树的后序遍历生成逆波兰表达式 |
| **文件系统** | 文件目录的组织和搜索 |
| **哈希表冲突解决** | 链地址法中使用 BST 替代链表优化碰撞链 |
| **自动补全 / 拼写检查** | Trie 的变体与 BST 结合使用 |
| **区间查询** | 区间树、线段树等 BST 变体 |

### BST 的局限性

普通 BST 在顺序插入时（如 1 → 2 → 3 → 4 → 5）会退化为链表，导致所有操作退化为 O(n)。为解决此问题，出现了**自平衡 BST**：

- **AVL 树**：严格平衡，左右子树高度差 ≤ 1
- **红黑树**：近似平衡，广泛应用于 C++ STL `map`/`set`、Java `TreeMap`
- **B 树 / B+ 树**：多路平衡搜索树，数据库和文件系统基石

> **一句话总结**：二叉搜索树是理解所有高级树结构的基础。掌握 BST 的递归思维（分治、范围约束、中序后继）后，AVL、红黑树、B 树等变体都能迎刃而解。
