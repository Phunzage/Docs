---
title: 数组
createTime: 2025/07/30 00:00:00
permalink: /data-structure/array/
---

# 数组（Array）

## 一、什么是数组

**数组（Array）** 是最基础、最常用的数据结构之一。它是由**相同类型**的元素组成的集合，这些元素在内存中**连续存储**，并通过**索引（下标）** 来访问。

### 核心特性

| 特性 | 说明 |
|------|------|
| **连续内存** | 所有元素在内存中占据一段连续的地址空间 |
| **同质元素** | 所有元素必须是同一数据类型 |
| **随机访问** | 通过索引可在 O(1) 时间内访问任意元素 |
| **固定大小** | 传统数组一旦创建，容量无法动态改变 |

### 内存布局

假设有一个 `[5]int` 数组，每个 `int` 占 8 字节，起始地址为 `0x1000`：

```
地址：   0x1000  0x1008  0x1010  0x1018  0x1020
        ┌──────┬──────┬──────┬──────┬──────┐
索引：  │  0   │  1   │  2   │  3   │  4   │
        └──────┴──────┴──────┴──────┴──────┘
```

访问 `arr[2]` 时，CPU 通过公式 `base_addr + index × element_size` 直接计算出地址 `0x1000 + 2×8 = 0x1010`，这就是 **O(1) 随机访问** 的原理。

### 数组与切片

在 Go 语言中，数组是**值类型**（赋值或传参会拷贝整个数组），而**切片（Slice）** 是对数组的一个动态视图，提供了动态数组的能力。本文以 Go 切片为主进行示例，因为它更贴近实际开发中的使用场景。

---

## 二、数组操作（Go 语言实现）

### 1. 初始化

Go 中数组和切片的多种初始化方式：

```go
package main

import "fmt"

func main() {
    // ── 固定长度数组 ──

    // 方式一：声明时初始化
    var arr1 [5]int = [5]int{1, 2, 3, 4, 5}
    fmt.Println("arr1:", arr1)

    // 方式二：简短声明
    arr2 := [5]int{10, 20, 30, 40, 50}
    fmt.Println("arr2:", arr2)

    // 方式三：部分初始化（未指定的元素为零值）
    arr3 := [5]int{1, 2} // [1, 2, 0, 0, 0]
    fmt.Println("arr3:", arr3)

    // 方式四：指定索引初始化
    arr4 := [5]int{0: 100, 3: 200} // [100, 0, 0, 200, 0]
    fmt.Println("arr4:", arr4)

    // ── 切片（动态数组） ──

    // 方式五：make 创建
    slice1 := make([]int, 5)     // [0, 0, 0, 0, 0]
    fmt.Println("slice1:", slice1)

    // 方式六：字面量创建
    slice2 := []int{1, 2, 3, 4, 5}
    fmt.Println("slice2:", slice2)

    // 方式七：从数组切片
    slice3 := arr1[1:4] // [2, 3, 4]
    fmt.Println("slice3:", slice3)
}
```

### 2. 随机访问（Random Access）

通过索引直接访问元素，时间复杂度 **O(1)**：

```go
func randomAccess(arr []int, index int) (int, error) {
    if index < 0 || index >= len(arr) {
        return 0, fmt.Errorf("索引越界: %d", index)
    }
    return arr[index], nil // O(1) — 直接计算内存地址
}

// 使用示例
func main() {
    nums := []int{10, 20, 30, 40, 50}
    val, _ := randomAccess(nums, 2)
    fmt.Println("arr[2] =", val) // 输出: arr[2] = 30
}
```

### 3. 插入元素（Insert）

在数组中间插入元素时，需要将插入位置及其之后的所有元素**向右平移一位**，腾出空位后再写入新元素。时间复杂度 **O(n)**。

```go
// Insert 在切片 index 位置插入元素 value
func Insert(slice *[]int, index int, value int) error {
    if index < 0 || index > len(*slice) {
        return fmt.Errorf("索引越界: %d", index)
    }

    // 在末尾追加一个零值元素，扩大容量
    *slice = append(*slice, 0)

    // 从后往前，将元素依次右移
    for i := len(*slice) - 1; i > index; i-- {
        (*slice)[i] = (*slice)[i-1]
    }

    // 在目标位置写入新值
    (*slice)[index] = value
    return nil
}

// 使用示例
func main() {
    arr := []int{1, 2, 3, 4, 5}
    fmt.Println("插入前:", arr) // [1, 2, 3, 4, 5]

    Insert(&arr, 2, 99)
    fmt.Println("插入后:", arr) // [1, 2, 99, 3, 4, 5]
}
```

**插入过程图解：**

```
初始：[1, 2, 3, 4, 5]
                ↓ 插入 99 到索引 2
Step1：先扩容 [1, 2, 3, 4, 5, _]      末尾追加零值
Step2：右移   [1, 2, 3, 4, 4, 5]      arr[5] = arr[4]
Step3：右移   [1, 2, 3, 3, 4, 5]      arr[4] = arr[3]
Step4：右移   [1, 2, 3, 3, 4, 5]      arr[3] = arr[2]
Step5：写入   [1, 2, 99, 3, 4, 5]     arr[2] = 99
```

### 4. 删除元素（Delete）

删除中间元素时，需要将删除位置之后的所有元素**向左平移一位**，然后缩减切片长度。时间复杂度 **O(n)**。

```go
// Delete 删除切片 index 位置的元素
func Delete(slice *[]int, index int) error {
    if index < 0 || index >= len(*slice) {
        return fmt.Errorf("索引越界: %d", index)
    }

    // 从前往后，将后续元素依次左移
    for i := index; i < len(*slice)-1; i++ {
        (*slice)[i] = (*slice)[i+1]
    }

    // 缩减切片长度
    *slice = (*slice)[:len(*slice)-1]
    return nil
}

// 使用示例
func main() {
    arr := []int{1, 2, 99, 3, 4, 5}
    fmt.Println("删除前:", arr) // [1, 2, 99, 3, 4, 5]

    Delete(&arr, 2)
    fmt.Println("删除后:", arr) // [1, 2, 3, 4, 5]
}
```

**删除过程图解：**

```
初始：[1, 2, 99, 3, 4, 5]
                ↓ 删除索引 2
Step1：左移   [1, 2, 3, 3, 4, 5]      arr[2] = arr[3]
Step2：左移   [1, 2, 3, 4, 4, 5]      arr[3] = arr[4]
Step3：左移   [1, 2, 3, 4, 5, 5]      arr[4] = arr[5]
Step4：截断   [1, 2, 3, 4, 5]         截掉末尾
```

### 5. 遍历（Traverse）

遍历数组的所有元素，时间复杂度 **O(n)**：

```go
// 方式一：for 循环 + 索引
func traverseByIndex(arr []int) {
    for i := 0; i < len(arr); i++ {
        fmt.Printf("arr[%d] = %d\n", i, arr[i])
    }
}

// 方式二：for range（推荐）
func traverseByRange(arr []int) {
    for i, v := range arr {
        fmt.Printf("索引 %d: 值 %d\n", i, v)
    }
}

// 方式三：只取值，忽略索引
func traverseValues(arr []int) {
    for _, v := range arr {
        fmt.Printf("值: %d ", v)
    }
    fmt.Println()
}

func main() {
    nums := []int{10, 20, 30, 40, 50}
    fmt.Println("=== for 索引遍历 ===")
    traverseByIndex(nums)
    fmt.Println("=== for range 遍历 ===")
    traverseByRange(nums)
}
```

### 6. 查找元素（Find）

在无序数组中查找指定值，需要**线性扫描**，时间复杂度 **O(n)**：

```go
// Find 在无序数组中查找 value，返回第一个匹配的索引，未找到返回 -1
func Find(arr []int, value int) int {
    for i, v := range arr {
        if v == value {
            return i // 找到，返回索引
        }
    }
    return -1 // 未找到
}

// FindAll 查找所有匹配的索引
func FindAll(arr []int, value int) []int {
    indices := []int{}
    for i, v := range arr {
        if v == value {
            indices = append(indices, i)
        }
    }
    return indices
}

func main() {
    nums := []int{5, 2, 8, 1, 9, 3, 8}
    fmt.Println("查找 8:", Find(nums, 8))       // 输出: 2
    fmt.Println("查找 10:", Find(nums, 10))     // 输出: -1
    fmt.Println("所有 8 的位置:", FindAll(nums, 8)) // 输出: [2, 6]
}
```

### 7. 扩容（Extend）

Go 的切片在 `append` 时自动扩容。当容量不足时，Go 会创建一个新数组，将旧元素拷贝过去，然后返回新的切片。手动扩容的原理如下：

```go
// Extend 手动扩容：创建一个更大的新数组，拷贝旧元素
func Extend(slice []int, newLength int) []int {
    if newLength <= cap(slice) {
        return slice[:newLength] // 容量充足，直接扩展长度
    }

    // 创建新数组（容量翻倍或指定大小）
    newSlice := make([]int, newLength)

    // 拷贝旧元素
    for i, v := range slice {
        newSlice[i] = v
    }

    // Go 内置的 copy 函数效果相同：
    // copy(newSlice, slice)

    return newSlice
}

func main() {
    arr := []int{1, 2, 3}
    fmt.Println("原数组:", arr, "长度:", len(arr), "容量:", cap(arr))

    // append 触发自动扩容
    arr = append(arr, 4, 5, 6)
    fmt.Println("扩容后:", arr, "长度:", len(arr), "容量:", cap(arr))

    // Go 的扩容策略（以 1.18+ 为例）：
    // - 容量 < 256 时，翻倍扩容
    // - 容量 ≥ 256 时，按 (newcap + 3*256) / 4 的公式增长
    // - 最终容量会根据元素类型的内存对齐进行调整
}
```

**Go 切片扩容策略（Go 1.18+）：**

| 旧容量 | 扩容倍数 | 示例 |
|--------|----------|------|
| < 256  | 2×（翻倍） | cap 64 → 128 |
| ≥ 256  | ~1.25× | cap 1024 → 1280 |

> 扩容的本质就是**创建一个更大的底层数组并拷贝元素**，这正是数组无法动态增长的体现，而切片通过间接层解决了这个问题。

---

## 三、数组 vs 链表

数组和链表是两种最基本的线性数据结构，它们的核心差异在于**内存布局**和**操作效率**。

| 对比维度 | 数组（Array） | 链表（Linked List） |
|----------|--------------|-------------------|
| **内存布局** | 连续内存块 | 分散节点，通过指针连接 |
| **随机访问** | **O(1)** — 通过地址公式直接计算 | **O(n)** — 必须从头遍历 |
| **插入/删除（头部）** | O(n) — 需要平移所有元素 | **O(1)** — 只需修改指针 |
| **插入/删除（尾部）** | **O(1)*** — 摊销常数时间 | O(n) — 需遍历到尾部（无尾指针时） |
| **插入/删除（中间）** | O(n) — 需要平移元素 | O(n) — 查找位置 + O(1) 改指针 |
| **空间开销** | 低，仅有数据本身 | 高，每个节点额外存储指针 |
| **CPU 缓存友好** | ✅ 高度友好（空间局部性） | ❌ 不友好（节点分散） |
| **内存碎片** | 少（整块分配） | 多（频繁分配/释放小对象） |
| **大小调整** | 困难（需创建新数组并拷贝） | 容易（动态增长） |

> \* 尾部插入在 Go 切片的摊销分析下为 O(1)，因为偶尔的扩容成本被分摊到每次插入中。

### 选型建议

- **频繁随机访问** → 选择数组（O(1) > O(n)）
- **频繁头部插入/删除** → 选择链表（O(1) > O(n)）
- **对缓存性能敏感** → 选择数组（顺序访问极快）
- **大小频繁变化** → 选择链表（无需搬运大量数据）
- **内存受限** → 选择数组（无指针开销）

---

## 四、时间复杂度总结

下表演示了数组各项操作的时间复杂度与空间复杂度：

| 操作 | 平均时间复杂度 | 最坏时间复杂度 | 空间复杂度 | 说明 |
|------|--------------|--------------|-----------|------|
| **随机访问** | O(1) | O(1) | O(1) | 通过索引直接计算地址 |
| **查找（无序）** | O(n) | O(n) | O(1) | 线性扫描 |
| **查找（有序、二分）** | O(log n) | O(log n) | O(1) | 前提：数组已排序 |
| **头部插入** | O(n) | O(n) | O(1) | 所有元素右移 |
| **尾部插入** | O(1)* | O(n)** | O(1) | *摊销常数 **触发扩容时 |
| **中间插入** | O(n) | O(n) | O(1) | 半数元素右移 |
| **头部删除** | O(n) | O(n) | O(1) | 所有元素左移 |
| **尾部删除** | O(1) | O(1) | O(1) | 直接缩容 |
| **中间删除** | O(n) | O(n) | O(1) | 半数元素左移 |
| **遍历** | O(n) | O(n) | O(1) | 依次访问每个元素 |

> **关于 O(1) vs O(n) 的直觉**：O(1) 意味着无论数组有多大，操作时间都是常数；O(n) 意味着操作时间和数组大小成正比 — 100 万个元素可能需要 100 万次操作。

---

## 五、数组的常见应用

### 1. 存储固定大小的数据

当数据量固定且需要高效随机访问时，数组是最自然的选择：

```go
// 一年的月份天数
var daysInMonth = [12]int{31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31}

// 颜色查找表
var colorTable = [256]RGB{...}

// 字符频率统计（经典计数数组）
func charFrequency(s string) [26]int {
    freq := [26]int{}
    for _, ch := range s {
        if ch >= 'a' && ch <= 'z' {
            freq[ch-'a']++
        }
    }
    return freq
}
```

### 2. 实现其他数据结构

数组是许多高级数据结构的底层基石：

| 数据结构 | 数组的作用 |
|----------|-----------|
| **堆（Heap）** | 用数组存储完全二叉树，父子节点通过索引公式定位：`left = 2i+1`，`right = 2i+2`，`parent = (i-1)/2` |
| **哈希表（Hash Table）** | 用数组作为桶（bucket），通过哈希函数映射到数组索引 |
| **循环队列（Circular Queue）** | 用数组 + 头尾指针实现 FIFO |
| **栈（Stack）** | 用数组 + 栈顶指针实现 LIFO |
| **矩阵（Matrix）** | 二维数组天然表示矩阵 |

### 3. 矩阵运算

```go
// 矩阵加法
func matrixAdd(a, b [][]int) [][]int {
    n, m := len(a), len(a[0])
    result := make([][]int, n)
    for i := range result {
        result[i] = make([]int, m)
        for j := 0; j < m; j++ {
            result[i][j] = a[i][j] + b[i][j]
        }
    }
    return result
}

// 矩阵转置
func matrixTranspose(matrix [][]int) [][]int {
    n, m := len(matrix), len(matrix[0])
    result := make([][]int, m)
    for i := range result {
        result[i] = make([]int, n)
        for j := 0; j < n; j++ {
            result[i][j] = matrix[j][i]
        }
    }
    return result
}
```

### 4. 缓存友好：顺序访问

数组的连续内存布局使得 CPU 可以**预取**后续数据到高速缓存中，大幅提升访问速度：

```go
// ❌ 跳跃访问 — 缓存不友好
func jumpAccess(matrix [][]int) int {
    n := len(matrix)
    m := len(matrix[0])
    sum := 0
    for j := 0; j < m; j++ {     // 外层列
        for i := 0; i < n; i++ { // 内层行
            sum += matrix[i][j]   // 跨行访问，跳跃大
        }
    }
    return sum
}

// ✅ 顺序访问 — 缓存友好
func sequentialAccess(matrix [][]int) int {
    sum := 0
    for i := 0; i < len(matrix); i++ {     // 外层行
        for j := 0; j < len(matrix[0]); j++ { // 内层列
            sum += matrix[i][j] // 逐行顺序访问
        }
    }
    return sum
}
```

### 5. 动态规划

数组是动态规划中最常用的数据结构，用于存储中间状态：

```go
// 斐波那契数列（DP 数组）
func fib(n int) int {
    if n <= 1 {
        return n
    }
    dp := make([]int, n+1)
    dp[0], dp[1] = 0, 1
    for i := 2; i <= n; i++ {
        dp[i] = dp[i-1] + dp[i-2] // 状态转移
    }
    return dp[n]
}
```

---

## 六、数组的局限性

尽管数组是最基础最高效的数据结构之一，它也有明显的局限：

1. **固定容量** — 传统数组大小不可变，扩容需要创建新数组并拷贝（O(n)）
2. **插入/删除慢** — 需要大量移动元素（O(n)）
3. **内存浪费** — 如果预分配过大，会浪费空间；分配过小，则频繁扩容
4. **同质限制** — 所有元素必须是同一类型（Go 中可用 `interface{}` 或泛型绕过，但失去类型安全和性能优势）

---

## 七、总结

数组是最基础、最高效的数据结构之一，其**连续内存布局**和**O(1)随机访问**能力使其成为程序员的瑞士军刀。尽管插入和删除操作较慢（O(n)），但结合 Go 的切片机制，数组在实际开发中仍然是最常用的底层数据结构。

**关键要点回顾：**

- ✅ 数组在内存中**连续存储**，支持 **O(1) 随机访问**
- ✅ 插入和删除操作需要**平移元素**，时间复杂度 **O(n)**
- ✅ Go 的**切片**是对数组的动态封装，是日常开发的首选
- ✅ 数组是**缓存友好**的数据结构，顺序访问性能极佳
- ✅ 作为基础组件，数组支撑着堆、哈希表、矩阵等多种结构的实现

掌握数组，就是掌握数据结构的起点。从数组出发，后面的链表、栈、队列、树、图都将触类旁通。
