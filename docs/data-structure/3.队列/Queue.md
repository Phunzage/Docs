---
title: 队列
createTime: 2025/11/20 18:59:13
permalink: /data-structure/ulsg8dli/
---

队列是一种遵循先进先出（FIFO, First In First Out）原则的线性数据结构，只允许在队尾进行插入操作（入队），在队头进行删除操作（出队）。

我们可以将队列类比为超市结账的排队队伍——先来的人先结账，后来的人排在队尾等待。我们将"人"替换为各种类型的元素（如整数、字符、对象等），就得到了队列这种数据结构。

### 核心特性
- **FIFO 原则**：最先进入队列的元素最先出队
- **双端操作**：入队（enqueue）在队尾，出队（dequeue）在队头
- **受限访问**：只能访问队头元素，无法直接访问中间元素
- **顺序保证**：严格保持操作顺序与到达顺序一致

### 常见类型
| 类型 | 特点 | 适用场景 |
|------|------|----------|
| 普通队列 | 基本 FIFO 结构 | 任务排队、缓冲区 |
| 循环队列 | 利用数组循环复用空间 | 固定大小缓冲区 |
| 双端队列 | 两端都可入队出队 | 滑动窗口、回文判断 |
| 优先队列 | 按优先级出队 | 任务调度、Dijkstra 算法 |

___

队列可以使用数组和链表来实现，下面分别介绍两种实现方式。

## 使用数组实现循环队列

数组实现队列时，如果简单用线性数组，出队操作会导致队头指针后移，造成数组前部空间的浪费。**循环队列**通过将数组首尾逻辑相连，解决了这一问题。

### 循环队列的设计原理

循环队列的核心思想是**模运算取余**，将数组视为一个环形结构。当指针到达数组末尾时，通过取余操作回到数组头部。

#### 关键变量
- **数组**：存储队列元素的连续内存空间
- **`front`**：队头指针，指向队头元素
- **`rear`**：队尾指针，指向队尾元素的下一个位置
- **`MAX_SIZE`**：数组最大容量

#### 空队与满队判断
- **队空**：`front == rear`
- **队满**：`(rear + 1) % MAX_SIZE == front`

::: warning
循环队列会浪费一个存储空间来区分队空和队满。当 `(rear + 1) % MAX_SIZE == front` 时，队列被认为已满，此时实际上还有一个空位未被使用。
:::

### 构造循环队列

#### 设计原理
- **固定容量**：预先分配固定大小的数组空间
- **双指针**：使用 `front` 和 `rear` 分别指示队头和队尾
- **循环复用**：通过模运算实现数组空间的循环使用

#### 关键理解点
- **浪费一个空间**：牺牲一个数组元素位置来区分空队和满队
- **初始状态**：`front = rear = 0` 表示空队列
- **模运算**：`(rear + 1) % MAX_SIZE` 实现指针自动回绕

::: code-tabs
@tab:active C++
```cpp
const int MAX_SIZE = 101;
int A[MAX_SIZE]{};
int front = 0;
int rear = 0;
```

@tab Go
```go
type ArrayQueue struct {
	data     []int
	front    int
	rear     int
	capacity int
}

func NewArrayQueue(capacity int) *ArrayQueue {
	return &ArrayQueue{
		data:     make([]int, capacity),
		front:    0,
		rear:     0,
		capacity: capacity,
	}
}
```
:::

### 判断队列是否为空

#### 函数参数说明
- 无参数（操作全局变量）
- 返回值：`true` 表示队列为空，`false` 表示队列非空

#### 操作原理
- 当 `front == rear` 时，队列为空
- 初始状态和经过完全出队后都会出现该条件

#### 关键理解点
- **唯一条件**：`front == rear` 是判断队列为空的唯一标准
- **循环不变性**：无论指针如何循环移动，该判断条件始终成立

::: code-tabs
@tab:active C++
```cpp
bool IsEmpty() {
    return front == rear;
}
```

@tab Go
```go
func (q *ArrayQueue) IsEmpty() bool {
    return q.front == q.rear
}
```
:::

### 判断队列是否已满

#### 函数参数说明
- 无参数（操作全局变量）
- 返回值：`true` 表示队列已满，`false` 表示队列未满

#### 操作原理
- 通过 `(rear + 1) % MAX_SIZE == front` 判断队列是否已满
- 队尾指针的下一个位置若等于队头指针，说明队列已满

#### 关键理解点
- **空间牺牲**：队列实际最多存储 `MAX_SIZE - 1` 个元素
- **循环边界**：模运算确保了指针在数组边界处的正确回绕

::: code-tabs
@tab:active C++
```cpp
bool IsFull() {
    return (rear + 1) % MAX_SIZE == front;
}
```

@tab Go
```go
func (q *ArrayQueue) IsFull() bool {
    return (q.rear+1)%q.capacity == q.front
}
```
:::

### 入队操作

#### 函数参数说明
- `x`：要入队的元素值
- 无返回值，直接修改全局队列

#### 操作原理
1. **队满检查**：检查队列是否已满
2. **存入元素**：将元素存入 `rear` 指向的位置
3. **移动队尾**：将 `rear` 指针后移一位（循环移动）

#### 关键理解点
- **先存后移**：先将元素存入当前 `rear` 位置，再将 `rear` 指针后移
- **循环后移**：使用 `(rear + 1) % MAX_SIZE` 实现队尾指针的循环移动
- **溢出保护**：必须在入队前检查队满条件

::: code-tabs
@tab:active C++
```cpp
void Enqueue(int x) {
    using std::cout;
    if (IsFull()) {
        cout << "错误，队列已满\n";
        return;
    }
    A[rear] = x;
    rear = (rear + 1) % MAX_SIZE;
}
```

@tab Go
```go
func (q *ArrayQueue) Enqueue(x int) {
    if q.IsFull() {
        fmt.Println("错误，队列已满")
        return
    }
    q.data[q.rear] = x
    q.rear = (q.rear + 1) % q.capacity
}
```
:::

### 出队操作

#### 函数参数说明
- 无参数（操作全局队列）
- 无返回值，直接修改全局队列

#### 操作原理
1. **队空检查**：检查队列是否为空
2. **移动队头**：将 `front` 指针后移一位（循环移动）

#### 关键理解点
- **逻辑删除**：出队只移动指针，数据仍在数组中但逻辑上不可访问
- **下溢保护**：必须在出队前检查队空条件
- **循环移动**：使用 `(front + 1) % MAX_SIZE` 实现队头指针的循环移动

::: code-tabs
@tab:active C++
```cpp
void Dequeue() {
    using std::cout;
    if (IsEmpty()) {
        cout << "错误，队列为空\n";
        return;
    }
    front = (front + 1) % MAX_SIZE;
}
```

@tab Go
```go
func (q *ArrayQueue) Dequeue() {
    if q.IsEmpty() {
        fmt.Println("错误，队列为空")
        return
    }
    q.front = (q.front + 1) % q.capacity
}
```
:::

### 访问队头元素

#### 函数参数说明
- 无参数（操作全局队列）
- 返回值：队头元素的值

#### 操作原理
- 直接通过 `front` 指针索引数组，返回队头元素

#### 关键理解点
- **只读操作**：访问队头不改变队列状态
- **空队处理**：在访问前应确保队列不为空

::: code-tabs
@tab:active C++
```cpp
int Front() {
    return A[front];
}
```

@tab Go
```go
func (q *ArrayQueue) Front() (int, error) {
    if q.IsEmpty() {
        return 0, fmt.Errorf("队列为空")
    }
    return q.data[q.front], nil
}
```
:::

### 打印队列元素

#### 函数参数说明
- 无参数（操作全局队列）

#### 操作原理
1. **临时变量**：使用 `i` 从 `front` 开始遍历
2. **循环遍历**：遍历直到 `i == rear`，逐个输出元素
3. **指针移动**：每次循环 `i = (i + 1) % MAX_SIZE`

#### 关键理解点
- **遍历范围**：从 `front` 到 `rear - 1`（模运算意义下）
- **循环遍历**：遍历过程中需要处理指针回绕
- **调试工具**：主要用于验证队列操作的正确性

::: code-tabs
@tab:active C++
```cpp
void Print() {
    using std::cout;
    cout << "Queue: ";
    for (int i = front; i != rear; i = (i + 1) % MAX_SIZE) {
        cout << A[i] << " ";
    }
    cout << "\n";
}
```

@tab Go
```go
func (q *ArrayQueue) Print() {
    fmt.Print("Queue: ")
    for i := q.front; i != q.rear; i = (i + 1) % q.capacity {
        fmt.Printf("%d ", q.data[i])
    }
    fmt.Println()
}
```
:::

___

## 使用链表实现队列

链表实现队列利用动态内存分配，可以灵活适应队列大小的变化，没有固定容量限制。

### 构造队列节点

#### 设计原理
- **节点结构**：每个节点包含数据域和指向下一节点的指针
- **双指针**：使用 `front` 和 `rear` 分别指向队头和队尾
- **动态内存**：节点在堆中动态分配，无固定大小限制

#### 关键理解点
- **队头出队**：出队操作在链表头部进行
- **队尾入队**：入队操作在链表尾部进行
- **尾指针优化**：通过 `rear` 指针实现 O(1) 的入队操作

::: code-tabs
@tab:active C++
```cpp
struct Node {
    int data;
    Node* next;
};

Node* front = NULL;
Node* rear = NULL;
```

@tab Go
```go
import "container/list"

type LinkedListQueue struct {
    data *list.List
}

func NewLinkedListQueue() *LinkedListQueue {
    return &LinkedListQueue{
        data: list.New(),
    }
}
```
:::

### 判断队列是否为空

#### 函数参数说明
- 无参数（操作全局指针）
- 返回值：`true` 表示队列为空，`false` 表示队列非空

#### 操作原理
- 当 `front == NULL` 时，队列为空
- 队列为空时，`rear` 也应为 `NULL`

#### 关键理解点
- **统一判断**：仅需检查 `front` 即可
- **空队状态**：入队第一个元素时需要同时更新 `front` 和 `rear`

::: code-tabs
@tab:active C++
```cpp
bool IsEmpty() {
    return front == NULL;
}
```

@tab Go
```go
func (q *LinkedListQueue) IsEmpty() bool {
    return q.data.Len() == 0
}
```
:::

### 入队操作（链表尾部插入节点）

#### 函数参数说明
- `x`：要入队的元素值
- 无返回值，直接修改全局队列

#### 操作原理
1. **创建节点**：在堆中为新元素分配节点内存
2. **设置数据**：将元素值存入新节点的数据域
3. **节点连接**：
   - **空队情况**：新节点同时成为 `front` 和 `rear`
   - **非空情况**：当前 `rear` 的 `next` 指向新节点，然后 `rear` 更新为新节点
4. **更新队尾**：`rear` 指针指向新节点

#### 关键理解点
- **尾插法**：在链表尾部插入保证 FIFO 顺序
- **空队特殊处理**：第一个节点需要同时更新队头和队尾指针
- **内存分配**：每次入队都需要动态分配内存

::: code-tabs
@tab:active C++
```cpp
void Enqueue(int x) {
    Node* temp = new Node();
    temp->data = x;
    temp->next = NULL;
    
    if (front == NULL && rear == NULL) {
        // 空队列，新节点同时成为队头和队尾
        front = rear = temp;
        return;
    }
    // 将新节点连接到队尾
    rear->next = temp;
    rear = temp;
}
```

@tab Go
```go
func (q *LinkedListQueue) Enqueue(x int) {
    q.data.PushBack(x)
}
```
:::

### 出队操作（链表头部删除节点）

#### 函数参数说明
- 无参数（操作全局指针）

#### 操作原理
1. **队空检查**：检查队列是否为空
2. **临时保存**：保存当前队头节点指针
3. **更新队头**：`front` 指向原队头的下一个节点
4. **唯一节点处理**：如果出队后队列为空，同时将 `rear` 置为 `NULL`
5. **释放内存**：删除原队头节点

#### 关键理解点
- **头删法**：删除链表头部节点实现 O(1) 的出队操作
- **内存管理**：必须手动释放出队节点的内存
- **特殊边界**：删除最后一个节点时要同时更新 `front` 和 `rear`
- **指针安全**：在释放内存前先更新指针

::: code-tabs
@tab:active C++
```cpp
void Dequeue() {
    using std::cout;
    if (front == NULL) {
        cout << "错误，队列为空\n";
        return;
    }
    Node* temp = front;
    front = front->next;
    // 如果出队后队列为空，同时更新 rear
    if (front == NULL) {
        rear = NULL;
    }
    delete temp;
}
```

@tab Go
```go
func (q *LinkedListQueue) Dequeue() {
    if q.IsEmpty() {
        fmt.Println("错误，队列为空")
        return
    }
    q.data.Remove(q.data.Front())
}
```
:::

### 访问队头元素

#### 函数参数说明
- 无参数（操作全局队列）
- 返回值：队头元素的值

#### 操作原理
- 直接通过 `front` 指针访问队头节点的数据域

#### 关键理解点
- **只读操作**：访问队头不影响队列结构
- **空队处理**：访问前需要确保队列不为空

::: code-tabs
@tab:active C++
```cpp
int Front() {
    if (front != NULL) {
        return front->data;
    }
    return -1; // 返回 -1 表示队列为空
}
```

@tab Go
```go
func (q *LinkedListQueue) Front() (int, error) {
    if q.IsEmpty() {
        return 0, fmt.Errorf("队列为空")
    }
    return q.data.Front().Value.(int), nil
}
```
:::

### 打印队列元素

#### 函数参数说明
- 无参数（操作全局队列）

#### 操作原理
1. **临时指针**：使用临时指针遍历队列，避免修改原指针
2. **顺序遍历**：从队头到队尾遍历所有节点
3. **数据输出**：输出每个节点的数据值

#### 关键理解点
- **遍历保护**：使用临时指针避免破坏队列结构
- **FIFO 顺序**：按先进先出的顺序打印元素

::: code-tabs
@tab:active C++
```cpp
void Print() {
    using std::cout;
    Node* temp = front;
    cout << "Queue: ";
    while (temp != NULL) {
        cout << temp->data << " ";
        temp = temp->next;
    }
    cout << "\n";
}
```

@tab Go
```go
func (q *LinkedListQueue) Print() {
    fmt.Print("Queue: ")
    for e := q.data.Front(); e != nil; e = e.Next() {
        fmt.Printf("%d ", e.Value.(int))
    }
    fmt.Println()
}
```
:::

___

## 完整示例代码

以下为完整的 C++ 和 Go 实现代码，可直接运行测试。

### 数组循环队列完整实现

::: code-tabs
@tab:active C++
```cpp
/* Queues01.cpp - 数组循环队列实现 */
#include <iostream>
using std::cout;

const int MAX_SIZE = 101;
int A[MAX_SIZE]{};
int front = 0;
int rear = 0;

bool IsEmpty() {
    return front == rear;
}

bool IsFull() {
    return (rear + 1) % MAX_SIZE == front;
}

void Enqueue(int x) {
    if (IsFull()) {
        cout << "错误，队列已满\n";
        return;
    }
    A[rear] = x;
    rear = (rear + 1) % MAX_SIZE;
}

void Dequeue() {
    if (IsEmpty()) {
        cout << "错误，队列为空\n";
        return;
    }
    front = (front + 1) % MAX_SIZE;
}

int Front() {
    return A[front];
}

void Print() {
    cout << "Queue: ";
    for (int i = front; i != rear; i = (i + 1) % MAX_SIZE) {
        cout << A[i] << " ";
    }
    cout << "\n";
}

int main() {
    Enqueue(10); Print();
    Enqueue(20); Print();
    Enqueue(30); Print();
    Dequeue();   Print();
    Enqueue(40); Print();
    return 0;
}
```

@tab Go
```go
// array_queue.go - 数组循环队列实现
package main

import "fmt"

type ArrayQueue struct {
	data     []int
	front    int
	rear     int
	capacity int
}

func NewArrayQueue(capacity int) *ArrayQueue {
	return &ArrayQueue{
		data:     make([]int, capacity),
		front:    0,
		rear:     0,
		capacity: capacity,
	}
}

func (q *ArrayQueue) IsEmpty() bool {
	return q.front == q.rear
}

func (q *ArrayQueue) IsFull() bool {
	return (q.rear+1)%q.capacity == q.front
}

func (q *ArrayQueue) Enqueue(x int) {
	if q.IsFull() {
		fmt.Println("错误，队列已满")
		return
	}
	q.data[q.rear] = x
	q.rear = (q.rear + 1) % q.capacity
}

func (q *ArrayQueue) Dequeue() {
	if q.IsEmpty() {
		fmt.Println("错误，队列为空")
		return
	}
	q.front = (q.front + 1) % q.capacity
}

func (q *ArrayQueue) Front() (int, error) {
	if q.IsEmpty() {
		return 0, fmt.Errorf("队列为空")
	}
	return q.data[q.front], nil
}

func (q *ArrayQueue) Print() {
	fmt.Print("Queue: ")
	for i := q.front; i != q.rear; i = (i + 1) % q.capacity {
		fmt.Printf("%d ", q.data[i])
	}
	fmt.Println()
}

func main() {
	q := NewArrayQueue(101)
	q.Enqueue(10); q.Print()
	q.Enqueue(20); q.Print()
	q.Enqueue(30); q.Print()
	q.Dequeue();   q.Print()
	q.Enqueue(40); q.Print()
}
```
:::

### 链表队列完整实现

::: code-tabs
@tab:active C++
```cpp
/* Queues02.cpp - 链表队列实现 */
#include <iostream>
using std::cout;

struct Node {
    int data;
    Node* next;
};

Node* front = NULL;
Node* rear = NULL;

bool IsEmpty() {
    return front == NULL;
}

void Enqueue(int x) {
    Node* temp = new Node();
    temp->data = x;
    temp->next = NULL;

    if (front == NULL && rear == NULL) {
        front = rear = temp;
        return;
    }
    rear->next = temp;
    rear = temp;
}

void Dequeue() {
    if (front == NULL) {
        cout << "错误，队列为空\n";
        return;
    }
    Node* temp = front;
    front = front->next;
    if (front == NULL) {
        rear = NULL;
    }
    delete temp;
}

int Front() {
    if (front != NULL) {
        return front->data;
    }
    return -1;
}

void Print() {
    Node* temp = front;
    cout << "Queue: ";
    while (temp != NULL) {
        cout << temp->data << " ";
        temp = temp->next;
    }
    cout << "\n";
}

int main() {
    Enqueue(10); Print();
    Enqueue(20); Print();
    Enqueue(30); Print();
    Dequeue();   Print();
    Enqueue(40); Print();
    return 0;
}
```

@tab Go
```go
// linked_queue.go - 链表队列实现
package main

import (
	"container/list"
	"fmt"
)

type LinkedListQueue struct {
	data *list.List
}

func NewLinkedListQueue() *LinkedListQueue {
	return &LinkedListQueue{
		data: list.New(),
	}
}

func (q *LinkedListQueue) IsEmpty() bool {
	return q.data.Len() == 0
}

func (q *LinkedListQueue) Enqueue(x int) {
	q.data.PushBack(x)
}

func (q *LinkedListQueue) Dequeue() {
	if q.IsEmpty() {
		fmt.Println("错误，队列为空")
		return
	}
	q.data.Remove(q.data.Front())
}

func (q *LinkedListQueue) Front() (int, error) {
	if q.IsEmpty() {
		return 0, fmt.Errorf("队列为空")
	}
	return q.data.Front().Value.(int), nil
}

func (q *LinkedListQueue) Print() {
	fmt.Print("Queue: ")
	for e := q.data.Front(); e != nil; e = e.Next() {
		fmt.Printf("%d ", e.Value.(int))
	}
	fmt.Println()
}

func main() {
	q := NewLinkedListQueue()
	q.Enqueue(10); q.Print()
	q.Enqueue(20); q.Print()
	q.Enqueue(30); q.Print()
	q.Dequeue();   q.Print()
	q.Enqueue(40); q.Print()
}
```
:::

___

## 两种实现方式对比

| 特性 | 数组实现（循环队列） | 链表实现 |
|------|---------------------|----------|
| 内存分配 | 静态预分配 | 动态分配 |
| 最大容量 | 固定（最多 `MAX_SIZE - 1`） | 理论上无限 |
| 内存开销 | 较小，浪费一个元素空间 | 每个节点有额外指针开销 |
| 访问速度 | 较快（缓存友好） | 相对较慢（内存跳跃） |
| 入队/出队时间复杂度 | O(1) | O(1) |
| 实现复杂度 | 中等（需处理循环逻辑） | 简单 |
| 内存碎片 | 无 | 可能产生碎片 |
| 适用场景 | 大小可预估的场景 | 大小变化大的场景 |

### 基本操作复杂度

| 操作 | 数组实现 | 链表实现 |
|------|----------|----------|
| 入队（Enqueue） | O(1) | O(1) |
| 出队（Dequeue） | O(1) | O(1) |
| 访问队头（Front） | O(1) | O(1) |
| 判空（IsEmpty） | O(1) | O(1) |
| 判满（IsFull） | O(1) | O(1)（链表无满） |
| 搜索 | O(n) | O(n) |

## 应用场景

### 1. 任务调度与消息队列
操作系统中的进程调度、消息中间件（如 RabbitMQ、Kafka）都使用队列来缓冲和按序处理任务。

### 2. 广度优先搜索（BFS）
在图论和树的层序遍历中，使用队列存储待访问节点，保证按"先发现先访问"的顺序进行遍历。

### 3. 打印任务管理
操作系统中的打印任务队列，按提交顺序依次处理打印请求。

### 4. 网络数据包缓冲
网络设备中使用队列缓冲到达的数据包，以平滑突发流量并保证按序处理。

### 5. 生产者-消费者模型
队列作为生产者线程和消费者线程之间的缓冲区，解决速度不匹配问题，解耦生产与消费。

### 6. 键盘缓冲区
计算机键盘输入采用队列结构，按键事件依次入队，应用程序按序读取。

### 7. IO 请求队列
磁盘 IO 请求按到达顺序排队，确保请求公平处理。
