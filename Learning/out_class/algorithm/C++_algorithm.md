## 基础概念知识：

### 堆（`heap`）：

- 一种完全二叉树，满足堆序性质

#### 大顶堆（max_heap）：

- 任意节点值 $\geqslant$ 子节点值，堆顶为最大值

- 使用实例：

```c++
#include <queue>
#include <vector>
#include <iostream>

using namespace std;

int main() {
    priority_queue<int> maxq;     // 大顶堆（默认）
    maxq.push(10); 
    maxq.push(5); 
    maxq.push(20);
    cout << maxq.top() << "\n";   // 20
    maxq.pop();                   // 删除堆顶（不返回值）
}
```

#### 小顶堆（min_heap）：

- 任意节点值 $\leqslant$ 子节点值，堆顶为最大值

- 使用实例：

```c++
#include <queue>
#include <vector>
#include <iostream>

using namespace std;

int main(){
	priority_queue<int, vector<int>, greater<int>> minq;
	minq.push(10); 
	minq.push(5);
	minq.push(20);
	cout << minq.top() << "\n";  // 5
}
```




### 二叉树：

- **定义**：二叉树是一种每个节点最多有两个子节点（通常称为左子节点和右子节点）的树状数据结构。
- **基本概念**：
    - **节点（node）**：包含数据和指向子节点的链接（或索引）。
    - **根节点（root）**：树的顶端节点，没有父节点。
    - **叶子节点（leaf）**：没有子节点的节点。
    - **高度/深度**：树的层数（或从某节点到最深叶子的最长路径长度）。	

---
#### 1. 分类

- **满二叉树**
    - 定义：每一层的节点都达到最大数量，所有节点的度要么是 0，要么是 2。
    - 特点：节点总数为 $2^k - 1$（其中 $k$ 为树的层数）。
        
- **完全二叉树**
    - 定义：除了最底层，其余各层的节点数都达到最大值，并且最底层的节点必须从左到右连续排列。
    - 特点：便于用数组进行顺序存储，常用于堆的实现。

- **平衡二叉树 (Balanced Tree)**
	- 定义：对于任意一个节点，其左右子树高度差绝对值不超过 1.
        
- **二叉搜索树（BST, Binary Search Tree）**
    - 定义：对于任意一个节点：
        - 左子树所有节点的值 < 当前节点值
        - 右子树所有节点的值 > 当前节点值 
    - 特点：只对元素的相对顺序有要求，对树的结构本身没有强制要求。
    - 应用：查找、插入、删除等操作的时间复杂度在平均情况下可达 $O(\log n)$。
        
- **平衡二叉搜索树（Balanced BST）**
    - 定义：在二叉搜索树的基础上，保证每个节点左右子树高度差的绝对值不超过 1。
    - 典型实现：AVL 树、红黑树。
    - 应用：C++ STL 中的 `map`、`set`、`multimap`、`multiset` 都是基于红黑树实现的，因此具有 **有序性** 和 **对数级别的查找效率**。

---
#### 2. 存储方式

1. **链式存储**
    - 节点定义通常为结构体（或类），包含数据域和左右子节点指针：

```c++
struct TreeNode {
    int val;
	TreeNode* left;
	TreeNode* right;
	TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};
```

2. **顺序（线性）存储**
    - 适合完全二叉树，用数组按层序存储：
        - 若父节点下标为 `i`，则：
            - 左子节点下标 ： `2 * i + 1`
            - 右子节点下标 ：`2 * i + 2`
    - 常用于堆（大顶堆、小顶堆）的实现。

---
#### 3. 遍历方式

#####  深度优先遍历（DFS）

- **递归实现常见，迭代法也可实现**。
- 三种遍历顺序：
    - **前序遍历（中 -> 左 -> 右）**  
        应用：构建表达式树时可用于输出前缀表达式。
    - **中序遍历（左 -> 中 -> 右）**  
        应用：在二叉搜索树中，中序遍历得到的是 **递增有序序列**。
    - **后序遍历（左 -> 右 -> 中）**  
        应用：常用于释放树节点内存（先删除子节点，再删除父节点）。

> [!NOTE] 递归设计逻辑：
> 1. 确定递归函数参数与返回值
> 2. 确定终止条件
> 3. 确定单层递归逻辑
>    
> 递归的底层逻辑实现：栈
> 递归 ——> 迭代（通过栈进行转换）

#####  广度优先遍历（BFS）
- **层序遍历**：从根节点开始，逐层访问节点。
- 常用队列（`queue`）辅助实现：
    - 入队根节点；
    - 依次出队节点并访问，同时将其左右子节点入队。








## 常用库

### queue：



### unordered_set

- **头文件**：`#include <unordered_set>`
- **定义**：
    - `unordered_set` 是一个基于哈希表实现的 **集合容器**。
    - 元素具有 **唯一性**（不允许重复）。
    - 元素存储 **无序**，不保证插入顺序。
- **底层结构**：哈希表（平均时间复杂度 O(1)）。
- **主要特性**：
    - 不能通过下标访问。
    - 元素值本身即为键值。
    - 查找、插入、删除的平均复杂度均为 **O(1)**。
- **常用操作**：

```c++
// 构建：
unordered_set<int> s;                             // 默认
unordered_set<int> s2 = {1,2,3};                  // 列表构造
unordered_set<int> s3(100);                       // 初始桶数
unordered_set<int> s4(100, std::hash<int>{});     // 指定哈希
unordered_set<int> s5(s2.begin(), s2.end());      // 区间

// 插入/原地构造：
s.insert(10);                     				  // 返回 pair<it,bool>，注意重复插入无效
s.insert({30, 40, 50});           				  // 初始化列表
s.emplace(20);                    				  // 原地构造，避免临时对象

// 查找：
auto it = s.find(20);            				  // 找不到则 == s.end()
size_t n = s.count(20);           				  // 只有 0 或 1

// 删除/清空/交换：
s.erase(10);                      // 按键删除，返回删除数量(0或1)
s.erase(s.begin());               // 按迭代器
s.erase(s.begin(), s.end());      // 按区间
s.clear();
s.swap(s2);

// 遍历：
for (const auto& v : s) { /* ... */ }
```

- **常用场景**：
	- 判断元素是否存在（快速查找）
	- 数据去重
	- 集合操作（交集、并集）

---
- 力扣 349 题：两数交集

```c++
#include<iostream>
#include<unordered_set>
#include<vector>
  
using namespace std;
  
class Solution {
public:
    // 算法思路：通过 unordered_set 来存储第一个数组的元素，然后遍历第二个数组，查找交集
    // unordered_set 特性：哈希表，查找速度快，适合用于查找交集, 且不允许重复元素
    vector<int> intersection(vector<int>& nums1, vector<int>& nums2)
    {
        unordered_set<int> set(nums1.begin(), nums1.end());
        unordered_set<int> res;
  
        for(int num = 0; num < nums2.size(); num++)
        {
           if(set.find(nums2[num]) != set.end())
           {
               res.insert(nums2[num]);
           }
        }
  
        return vector<int>(res.begin(), res.end());
    }
};
```

### unordered_map

- **头文件**：`#include <unordered_map>`
- **定义**：
	- `unordered_map` 是一个基于哈希表实现的 **键值对容器**
	- 元素由 **key-value** 组成
	- `key` 唯一，但可以修改 `value`
	- 存储 **无序**
- **底层结构**：哈希表
- **主要特性**：
	- 按键 `key` 访问对应的值 `value`
	- 插入、查找、删除的平均复杂度为O(1)
	- 支持 `[]` 运算符访问，若 `key` 不存在则自动创建一个默认值
- **常见操作**：

```c++
// 构建：
unordered_map<string,int> m;                             // 默认
unordered_map<string,int> m2 = { {"a",1}, {"b",2} };     // 列表
unordered_map<string,int> m3(1024);                      // 初始桶数
unordered_map<string,int> m4(m2.begin(), m2.end());      // 区间

// 元素访问：
m["apple"] = 2;              // 若不存在，插入 {"apple",0} 再赋值为 2
int n = m["apple"];          // 若不存在，会插入默认值（0）！小心副作用
int x = m.at("apple");       // 不存在会抛出 std::out_of_range

// 插入/原地构造：
m.insert({"banana", 5});               // 返回 pair<it,bool>
m.emplace("orange", 3);                // 原地构造

// 查找/计数/判断存在：
auto it = m.find("banana");            // 找不到 == m.end()
size_t n = m.count("banana");          // 0 或 1
auto [first,last] = m.equal_range("banana"); // 范围（对唯一键等同于 find）

// 删除/清空/交换：
m.erase("apple");                      // 按键
m.erase(it);                           // 按迭代器，返回下一个迭代器
m.erase(m.begin(), m.end());           // 区间
m.clear();
m.swap(m2);

// 遍历：
for (auto& [k, v] : m) { /*  */ }
```

- **适用场景**：
	- 快速的键值对存储与查找（如单词计数、索引映射）。
	- 替代传统的 `map`（红黑树实现，O(logn)）。
	- 哈希表存储比有序 map 更高效，但元素无序。

---
- 力扣 1 题：两数之和

```C++
#include <iostream>
#include <vector>
#include <unordered_map>
  
using namespace std;
  
class Solution{
public:
    // 算法思路：查找互补数
    //          使用哈希表存储已遍历的数字及其索引
    //          每次跳转时进行哈希表互补数查找
    vector<int> twoSum(vector<int>& nums, int target)
    {
        unordered_map<int, int> numMap;
        for(int i = 0; i < nums.size(); i++)
        {
            int complement = target - nums[i];
            if(numMap.find(complement) != numMap.end())
            {
                return {numMap[complement], i};
            }
  
            numMap[nums[i]] = i;
        }
        return {};
    }
};
```


## 算法：

### 排序算法（`Sorting Algorithms`）

#### 1. 概述

- **定义**：排序算法是将一组数据按照特定顺序（升序或降序）重新排列的算法过程。
- **分类方式**：
    - 按**时间复杂度**：$O(n^2)$、$O(n\log n)$、$O(n)$。
    - 按**是否稳定**：稳定排序（保持相同元素的原有相对位置）与不稳定排序。
    - 按**是否原地排序**：是否使用额外空间（如原地排序 vs 非原地排序）。
- **应用场景**：数据检索、去重、二分查找、统计分析、动态规划预处理等。

---
#### 2. 冒泡排序（`Bubble Sort`）

##### 思想：

- 相邻元素两两比较，若顺序错误则交换，使较大的元素“逐步冒泡”到末尾。

##### 步骤：

1. 外层循环控制趟数，每一趟确定一个最大元素位置；
2. 内层循环进行相邻比较与交换；
3. 若一趟内未发生交换，则表示排序已完成。

##### 时间复杂度：

- 最好：$O(n)$（已排序）
- 最坏：$O(n^2)$
- 空间复杂度：$O(1)$
- 稳定性：稳定

##### 代码示例：

```cpp
void bubbleSort(vector<int>& arr) {
	for (int i = 0; i < arr.size() - 1; ++i) {
		bool flag = false;
		for (int j = 0; j < arr.size() - i - 1; ++j) {
			if (arr[j] > arr[j + 1]) {
				swap(arr[j], arr[j + 1]);
				flag = true;
			}
		}
		if (!flag) break;
	}
}
```

---
#### 3. 选择排序（`Selection Sort`）

##### 思想：

- 每一趟从未排序区中选择最小（或最大）元素，放到已排序区末尾。

##### 步骤：

1. 遍历未排序区，找到最小值；
2. 将其与未排序区首元素交换；
3. 重复上述步骤直到所有元素有序。

##### 时间复杂度：

- 最好/最坏：$O(n^2)$
- 空间复杂度：$O(1)$
- 稳定性： 不稳定

##### 代码示例：

```cpp
void selectionSort(vector<int>& arr) {
	for (int i = 0; i < arr.size() - 1; ++i) {
		int minIndex = i;
		for (int j = i + 1; j < arr.size(); ++j) if (arr[j] < arr[minIndex]) minIndex = j;
		swap(arr[i], arr[minIndex]);
	}
}
```

---

#### 4. 插入排序（`Insertion Sort`）

##### 思想：

- 将序列分为已排序与未排序两部分；
- 逐个取出未排序区元素，插入到前面已排序区的正确位置。

##### 步骤：

1. 从第二个元素开始；
2. 将当前元素与已排序区元素从后向前比较；
3. 找到合适位置插入。

##### 时间复杂度：

- 最好：$O(n)$（已排序）
- 最坏：$O(n^2)$
- 空间复杂度：$O(1)$
- 稳定性：稳定

##### 代码示例：

```cpp
void insertionSort(vector<int>& arr) {
	for (int i = 1; i < arr.size(); ++i) {
		int temp = arr[i];
		int j = i - 1;
		while (j >= 0 && arr[j] > temp) {             
			arr[j + 1] = arr[j];             
			j--;         
		}         
		arr[j + 1] = temp;     
	} 
}
```

---
#### 5. 快速排序（`Quick Sort`）

##### 思想：

- 采用分治思想；
- 通过一次划分将数组分为两部分：左边都比基准小，右边都比基准大；
- 对左右子序列递归排序。

##### 步骤：

1. 选取基准（通常为首或中间元素）；
2. 左右指针交换，完成一次分区；
3. 对左右部分递归调用快速排序。

##### 时间复杂度：

- 最好/平均：$O(n\log n)$
- 最坏：$O(n^2)$
- 空间复杂度：$O(\log n)$（递归栈）
- 稳定性：不稳定

##### 代码示例：

```c
int partition(vector<int>& arr, int low, int high) {     
	int pivot = arr[low];     
	while (low < high) {         
		while (low < high && arr[high] >= pivot) high--;         
		arr[low] = arr[high];         
		while (low < high && arr[low] <= pivot) low++;         
		arr[high] = arr[low];     
	}     
	arr[low] = pivot;     
	return low; 
}  
	
void quickSort(vector<int>& arr, int low, int high) {     
	if (low < high) {         
		int pivotIndex = partition(arr, low, high);         
		quickSort(arr, low, pivotIndex - 1);         
		quickSort(arr, pivotIndex + 1, high);     
	} 
}
```

---

#### 6. 归并排序（`Merge Sort`）

##### 思想：

- 使用分治思想，将数组不断二分，直到每个部分只有一个元素；
- 然后逐步合并成有序序列。

##### 时间复杂度：

- 最好/最坏/平均：$O(n\log n)$
- 空间复杂度：$O(n)$（合并过程需要临时数组）
- 稳定性：稳定

##### 代码示例：

```cpp
void merge(vector<int>& arr, int left, int mid, int right) {     
	vector<int> temp;     
	int i = left, j = mid + 1;     
	while (i <= mid && j <= right) temp.push_back(arr[i] <= arr[j] ? arr[i++] : arr[j++]);     
	while (i <= mid) temp.push_back(arr[i++]);     
	while (j <= right) temp.push_back(arr[j++]);     
	for (int k = 0; k < temp.size(); ++k)  arr[left + k] = temp[k]; 
}  

void mergeSort(vector<int>& arr, int left, int right) {     
	if (left >= right) return;     
	int mid = (left + right) / 2;     
	mergeSort(arr, left, mid);     
	mergeSort(arr, mid + 1, right);     
	merge(arr, left, mid, right); 
}
```

---

#### 7. 堆排序（`Heap Sort`）

##### 思想：

- 构建一个大顶堆；
- 每次取堆顶元素（最大值），与末尾元素交换；
- 然后重新调整堆。

##### 时间复杂度：

- 最好/平均/最坏：$O(n\log n)$
- 空间复杂度：$O(1)$
- 稳定性： 不稳定

##### 代码示例：

```cpp
void heapify(vector<int>& arr, int n, int i) {     
	int largest = i, l = 2 * i + 1, r = 2 * i + 2;     
	if (l < n && arr[l] > arr[largest]) largest = l;     
	if (r < n && arr[r] > arr[largest]) largest = r;     
	if (largest != i) {         
		swap(arr[i], arr[largest]);         
		heapify(arr, n, largest);     
	} 
}  

void heapSort(vector<int>& arr) {     
for (int i = arr.size() / 2 - 1; i >= 0; --i)
	heapify(arr, arr.size(), i);     
	for (int i = arr.size() - 1; i > 0; --i) {         
		swap(arr[0], arr[i]);         
		heapify(arr, i, 0);     
	} 
}
```

---
#### 8. 各排序算法性能对比

| 算法名称 | 平均时间复杂度      | 空间复杂度       | 稳定性 | 适用场景   |
| ---- | ------------ | ----------- | --- | ------ |
| 冒泡排序 | $O(n^2)$     | $O(1)$      | 稳定  | 小规模数据  |
| 选择排序 | $O(n^2)$     | $O(1)$      | 不稳定 | 内存受限   |
| 插入排序 | $O(n^2)$     | $O(1)$      | 稳定  | 部分有序   |
| 快速排序 | $O(n\log n)$ | $O(\log n)$ | 不稳定 | 通用最优   |
| 归并排序 | $O(n\log n)$ | $O(n)$      | 稳定  | 链表、大数据 |
| 堆排序  | $O(n\log n)$ | $O(1)$      | 不稳定 | 优先队列实现 |







### $KMP$ 算法：

#### 一、概述 :

- **解决问题**：字符串模式匹配（`Pattern Matching`）
- **目标**：在文本串 `text` 中找到模式串 `pattern` 的所有匹配位置

#### 二、暴力匹配（Brute Force）

- 思路：
	- 从文本串的第一个字符开始，与模式串逐一比较
	- 匹配失败时，模式串整体向右移动一位，再继续比较
- 时间复杂度：
	- **最坏情况**：$O(m \times n)$
	    - $m$ ：文本串长度
	    - $n$ ：模式串长度

#### 三、KMP 算法（Knuth-Morris-Pratt）

- **思想**：
	- **避免重复比较**：当发生失配时，不再回退文本串的指针，而是利用 **模式串本身的规律**，快速确定下一次匹配位置。
		- （即当模式串匹配不等时，不依据暴力匹配中的逐步移位，转为跳转至当前字符位置后缀字段与前缀字段相等处进行迭代匹配）
	- 核心：**最长相等前后缀匹配**
- **关键概念**：
	- **前缀**：只包含首字母不包含尾字母的所有子串
	- **后缀**：只包含尾字母不包含首字母的所有子串
	- **最长相等前后缀（LPS: Longest Prefix Suffix）**：前缀与后缀相等的最长长度

> [!NOTE]
> 例子：模式串 `"abab"`
> - 前缀集合：`a`, `ab`, `aba`
> - 后缀集合：`b`, `ab`, `bab`
> - 最长相等前后缀：`"ab"`，长度为 2

- **时间复杂度**：
	- **Next 数组计算**：$O(n)$
	- **匹配过程**：$O(m)$
	- **总时间复杂度**：$O(m + n)$

#### 四、算法流程 

- **核心数据结构**：
	- **构建 next 数组**（模式串预处理）
    - 记录每个位置最长相等前后缀长度
- **进行匹配**
    - 遍历文本串
    - 如果失配，利用 `next` 数组调整模式串位置，而不是回退文本指针

---
C++ 代码实现：

```c++
#include <iostream>
#include <vector>
#include <string>
using namespace std;

// 构建 Next 数组
vector<int> buildNext(const string &pattern) {
    // 初始化
    vector<int> next(n, 0);
    int j = 0; // 前缀末尾
    
    for (int i = 1; i < pattern.size(); i++) {
        while (j > 0 && pattern[i] != pattern[j]) {
            j = next[j - 1]; // 回退到之前的最长相等前后缀
        }
        if (pattern[i] == pattern[j]) {
            j++;
        }
        next[i] = j;
    }
    return next;
}

// KMP 匹配
vector<int> KMP(const string &text, const string &pattern) {
    vector<int> next = buildNext(pattern);
    vector<int> res;
    int j = 0; // 模式串指针

    for (int i = 0; i < (int)text.size(); i++) {
        while (j > 0 && text[i] != pattern[j]) {
            j = next[j - 1];
        }
        if (text[i] == pattern[j]) {
            j++;
        }
        if (j == (int)pattern.size()) {
            res.push_back(i - j + 1); // 记录匹配起始位置
            j = next[j - 1];          // 继续寻找下一个匹配
        }
    }
    return res;
}

int main() {
    string text = "ababcabcacbab";
    string pattern = "abcac";
    
    vector<int> positions = KMP(text, pattern);
    if (!positions.empty()) {
        cout << "Pattern found at positions: ";
        for (int pos : positions) cout << pos << " ";
        cout << endl;
    } else {
        cout << "Pattern not found." << endl;
    }
    return 0;
}

```









### 回溯算法（`Backtracking`）

#### 1. 基本概念

- **核心思想**：回溯是一种 **穷举搜索** 的思想，通过 **递归 + 状态回退** 的方式尝试所有可能解。
- **应用场景**：
    - **组合问题**：从 n 个数中选 k 个，不强调顺序（如组合数、求和问题）。
    - **切割问题**：将字符串按照某种规则切割（如分割回文子串）。
    - **子集问题**：集合的所有子集（如子集枚举、幂集）。
    - **排列问题**：强调顺序，要求输出所有可能排列（如全排列）。
    - **棋盘问题**：典型回溯问题，如 $N$ 皇后、解数独、八皇后、迷宫路径搜索。

> [!NOTE] Title
> 思维模型：通常将问题抽象为一棵 **树形结构**，每一次递归就是向树的一层推进，每一条路径代表一种解答的尝试。

---
#### 2. 基本思路

1. **定义递归函数**（`backtracking`），表示在某一层搜索的状态。
2. **递归终止条件**：达到题目要求时收集结果（如组合长度到达 `k`）。
3. **递归处理逻辑**：在当前层做出选择。
4. **回溯操作**：撤销选择，恢复现场，继续尝试其他可能。

---
#### 3. 算法模板（C++ 伪代码）

```c++
// 返回类型可能是 void 或 bool（根据题目需求）
void backtracking(参数...) {
    if (终止条件) {
        收集结果;
        return;
    }

    for (int i = 本层起始位置; i < 集合.size(); i++) {
        // 处理节点（做选择）
        路径.push_back(集合[i]);

        // 递归（进入下一层）
        backtracking(参数...);

        // 回溯（撤销选择）
        路径.pop_back();
    }
}
```

- **路径（`path`）**：记录当前已选择的结果（通常用 `vector<int>`）。
- **选择列表（`candidates`）**：当前可选择的元素。
- **结束条件**：满足题目要求时，收集当前路径。

---
#### 4. 回溯的关键点

1. **树的遍历**：回溯其实就是一棵 N 叉树的深度优先遍历（DFS）。
2. **状态恢复**：必须有“撤销操作”（回溯），否则结果错误。
3. **剪枝优化**：在递归前判断是否有必要继续搜索，减少无效分支。
    - 例如组合问题中：剩余元素数量不足时，可以提前结束循环。












### 动态规划（Dynamic Programming）

---
#### 1. 概述

- 动态规划（Dynamic Programming，简称 **DP**）是一种 **通过拆解问题、保存中间结果** 来避免重复计算，从而提升效率的算法思想。
- 使用场景：
    - 当问题可以分解为 **多个子问题**，且这些子问题之间存在 **重叠**（即相同子问题被多次计算）。
    - 问题需要在 **多种可能解** 中寻找最优解（最值问题）。
- 核心思想：
    - 将大问题拆解成小问题，并记录小问题的解（状态），通过 **状态转移方程** 逐步得到最终解。

-  区别：
	- **贪心算法**：只关注局部最优，不能保证全局最优。
	- **分治算法**：将问题拆分为 **相互独立** 的子问题，递归解决后合并。
	- **动态规划**：适用于子问题有 **重叠** 的情况，需保存并复用结果。

---
#### 2. 常见问题类型

- **基础问题**：斐波那契数列、爬楼梯
- **背包问题**：01 背包、完全背包、多重背包
- **打家劫舍**：房屋选择、环形打家劫舍
- **股票买卖问题**：限制买卖次数、带冷冻期、带手续费
- **子序列问题**：最长上升子序列、最长公共子序列、编辑距离
- **路径规划问题**：不同路径、最小路径和、三角形最小路径和

---
#### 3. 动态规划解题步骤（五要素）

写 DP 时，一般遵循以下五个关键步骤：
1. **确定 `dp` 数组以及下标的含义**
    - 明确 `dp[i]` / `dp[i][j]` 的具体意义（如表示前 i 个元素的最优解）。
2. **确定状态转移公式**
    - 根据问题特性推导出如何从已知状态转移到当前状态。
    - 例如：
        - 斐波那契数列：`dp[i] = dp[i-1] + dp[i-2]`
        - 背包问题：`dp[i][j] = max(dp[i-1][j], dp[i-1][j-w] + v)`
            
3. **初始化 `dp` 数组**
    - 边界条件需要明确（如 `dp[0]`、`dp[1]` 等）。
        
4. **确定遍历顺序**
    - 一维 or 二维遍历，正序 or 倒序，取决于状态转移关系。
    - 特别是背包问题中，不同遍历顺序决定是否是 **01 背包** 或 **完全背包**。
        
5. **打印 / 验证 `dp` 数组**
    - 适当输出 `dp` 数组可帮助调试，确保状态转移公式正确。
        

---
#### 4. 示例解析

##### 4.1 斐波那契数列

- 问题：`f(n) = f(n-1) + f(n-2)`
- 定义：`dp[i]` 表示第 i 个斐波那契数
- 状态转移：`dp[i] = dp[i-1] + dp[i-2]`
- 初始化：`dp[0] = 0, dp[1] = 1`
- 遍历顺序：从 2 到 n

代码：

```C++
int fib(int n) {
    if(n < 2) return n;
    vector<int> dp(n+1, 0);
    dp[0] = 0;
    dp[1] = 1;
    
  	for(int i = 2; i <= n; i++) {
 		dp[i] = dp[i-1] + dp[i-2];
	}
	return dp[n];
}
```

---
##### 4.2 01 背包问题

- 问题：给定 n 件物品和一个容量为 W 的背包，每件物品有重量 w 和价值 v，问如何选择物品使得总价值最大。
- 定义：`dp[i][j]` 表示前 i 件物品在容量 j 下的最大价值
- 状态转移：`dp[i][j] = max(dp[i - 1][j], dp[i - 1][j - w] + v)`
  `(若 j >= w) dp[i][j] = dp[i - 1][j], (若 j < w) dp[i][j] = dp[i - 1][j - w] + v`
- 初始化：`dp[0][j] = 0`
- 遍历顺序：先物品 i，再容量 j（01 背包需倒序遍历容量）。

---
##### 4.3 最长公共子序列 (LCS)

- 问题：求两个字符串的最长公共子序列长度。
- 定义：`dp[i][j]` 表示字符串 A 前 i 个和字符串 B 前 j 个的 LCS 长度。
- 状态转移：
    `if (A[i-1] == B[j-1]) dp[i][j] = dp[i-1][j-1] + 1; else dp[i][j] = max(dp[i-1][j], dp[i][j-1]);`
- 初始化：`dp[0][j] = 0, dp[i][0] = 0`

---
#### 5. 动态规划优化技巧

- **空间优化**：二维数组可压缩为一维数组（如背包问题）。
- **滚动数组**：只保留最近两行数据，减少内存使用。
- **记忆化搜索**：自顶向下递归 + 缓存子问题结果。
- **路径回溯**：通过记录决策路径，输出最优解的方案而不只是数值。






### 图论（Graph Theory）

图论是一种用于描述“对象间关系”的数学模型。图由节点（顶点）与边组成，用于表示网络结构，如道路、通信、依赖关系等。  
在编程中，图常用邻接矩阵或邻接表表示。

---
#### 一、并查集（Union-Find / Disjoint Set Union）

##### **适用场景：**

- 判断两个节点是否属于同一集合（如判断两点是否连通）
- 合并两个集合（如 Kruskal 最小生成树算法中用来判断是否成环）

##### **算法思想：**

- 将每个节点看作一棵树的根节点；
- `find` 操作通过递归找到元素所在集合的“祖先节点”（根节点）；
- `union` 操作用于将两个节点所在的集合合并；
- 如果两个节点的祖先节点相同，则它们属于同一集合。

##### **关键优化：**

- **路径压缩（Path Compression）：** 在查找过程中直接将节点连接到根节点；
- **按秩合并（Union by Rank）：** 优先将较小的树挂在较大的树下

##### **算法模板：**

```c++
class checkCollections {
private:
	int elementSize;
	vector<int> father;
	
	int findElement(int u) {
		return u == father[u] ? u : father[u] = findElement(father[u]);
	}
	
public:
	void init(int n) {
		elementSize = n;
		father = vector<int>(n + 1);
		for(int i = 0; i <= n; i ++) father[i] = i;
	}
	
	void joinElement(int u, int v) {
		int u = findElement(u);
		int v = findElement(v);
		
		if(u == v) return;
		else father[v] = u;
	}
	
	bool isSameRoot(int u, int v) {
		int u = findElement(u);
		int v = findElement(v);
		return u == v;
	}
}
```


---
#### 二、Prim 算法（最小生成树之一）

##### **基础概念：**

- **最小生成树（MST）**：  
    在一个无向连通图中，选取若干条边使所有节点连通，且边权和最小。
- 最小生成树有 $n-1$ 条边（n为节点数）。

##### **适用场景：**

- 求解无向有权图的最小生成树问题；
- 常用于网络布线、路径规划、通信成本最优化等问题。

##### **算法思想：**

Prim 算法以“点”为中心，逐步扩展生成树：

1. 从任意一点开始，将它加入生成树；
2. 从当前生成树出发，找到一条**连接生成树与非生成树节点的最短边**；
3. 加入该边与对应的节点；
4. 重复此过程，直到所有节点加入生成树。

##### **复杂度：**

- 邻接矩阵实现：$O(n^2)$
- 邻接表 + 堆优化：$O(E \log V)$

##### **算法模板：**

```c++
int prime(vector<vector>> &graph, vector<bool> &isInTree, vector<int> &minDistances) {
	int minDistance = 0;
	int nodeNums = graph.size() - 1;
	
	// 初始化 prim, 默认最小生成树初始节点为 1
	isInTree[1] = true;
	for(int i = 2; i <= nodeNums; i ++) {
		if(graph[1][i] != 0) minDistances[i] = graph[1][i];
	}
	
	// 循环 nodeNums - 1 次 -> 查找 nodeNums - 1 条边
	for(int i = 1; i < nodeNums; i ++) {
		int curNode = -1;
		int curMinVal = INT_MAX;
		
		// 寻找距离生成树最小非生成树节点
		for(int j = 2; j <= nodeNums; j ++) {
			if(!isInTree[j] && minDistances[j] < curMinVal) {
				curNode = j;
				curMinVal = minDistances[j];
			}
		}
		
		// 加入生成树
		if(curNode == -1) return -1;
		else {
			isInTree[curNode] = true;
			minDistance += minDistances[curNode];
		}
		
		// 更新非生成树节点到生成树节点距离
		for(int j = 2; j <= nodeNums; j ++) {
			if(!isInTree[j] && graph[curNode][j] != 0 && graph[curNode][j] < minDistances[j]) {
				minDistances[j] = graph[curNode][j];
			}
		}
	}
	return miniDistance;
}
```

---
#### 三、Kruskal 算法（最小生成树之一）

##### **算法思想：**

Kruskal 算法以“边”为中心，采用**贪心策略**：

1. 将所有边按权值从小到大排序；
2. 依次选择最小的边；
3. 若该边连接的两个节点属于不同集合，则加入生成树；
4. 若属于同一集合，则跳过（防止成环）；
5. 重复直至生成树包含 $n-1$ 条边

##### **核心：**

使用并查集判断两个节点是否已经连通。

##### **适用场景：**

- 同样用于最小生成树；
- 适合边较少的稀疏图。

##### **时间复杂度：**

$O(E \log E)$ （排序边）

##### **算法模板：**

```c++
struct Edge {
    int left;
    int right;
    int val;
};
  
class checkCollections {
private:
    int elementNums;
    vector<int> father;
  
    int findElement(int u) {
        return u == father[u] ? u : father[u] = findElement(father[u]);
    }
  
public:
    void init(int n) {
        elementNums = n;
        father = vector<int>(n + 1);
        for(int i = 1; i <= n; i ++) father[i] = i;
    }
  
    void joinElement(int u, int v) {
        u = findElement(u);
        v = findElement(v);
  
        if(u == v) return;
        else father[v] = u;
    }
  
    bool isSameRoot(int u, int v) {
        u = findElement(u);
        v = findElement(v);
        return u == v;
    }
};

static bool cmp(const Edge &a, const Edge &b) {
	return a.val < b.val;
}

int kruscal(vector<Edge> &edge, int n) {
	int minDistance = 0;
	int edgeNums = edge.size();

	checkCollections col;
	col.init(n);
	sort(edge.begin(), edge.end(), cmp);

	for(int i = 0; i < edgeNums; i ++) {
		if(!col.isSameRoot(edge[i].left, edge[i].right)) {
			col.joinElement(edge[i].left, edge[i].right);
			minDistance += edge[i].val;
		}
	}
	return minDistance;
}
```

---
#### 四、拓扑排序（Topological Sort）

##### **适用场景：**

- 针对**有向无环图（DAG）**；
- 常用于任务调度、依赖分析、编译顺序、项目构建顺序等。

##### **关键概念：**

- **入度（in-degree）**：指向某节点的边的数量；
- **拓扑序列**：每条有向边 $(u, v)$ 中，节点 `u` 必在 `v` 之前。

##### **算法思想：**

1. 统计所有节点的入度；
2. 将所有入度为 0 的节点加入队列；
3. 弹出一个节点并输出；
4. 删除该节点指向的边，使相邻节点入度减 1；
5. 若某节点入度变为 0，则入队；
6. 循环直到队列为空。

##### **判断有无环：**

若最终输出节点数 < 总节点数，则图中存在环。

##### **时间复杂度：**

$O(V + E)$

##### **算法模板：**

```c++ 
vector<int> topologicalSort(unordered_map<int, vector<int>> &umap, vector<int> inDegree) {
	vector<int> result;
	int nodeNums = inDegree.size();
	
	queue<int> que;
	for(int i = 0; i < nodeNums; i++) {
		if(inDegree[i] == 0) que.push(i);
	}
	
	while(!que.empty()) {
		int cur = que.front();
		que.pop();
		result.push_back(cur);
		
		auto it = umap.find(cur);
		if(it != uamp.end()) {
			const vector<int> &files = it.second;
			for(int i = 0; i < files.size(); i++) {
				inDegree[files[i]] --;
				if(inDegree[files[i]] == 0) que.push(files[i]);
			}
		}
	}
	return result;
}
```

---
#### 五、Dijkstra 算法（单源最短路径）

##### **适用场景：**

- 有向/无向图；
- **边权为非负数**；
- 求**单源最短路径**。

##### **算法思想：**

1. 初始化所有点的最短距离为 `INT_MAX`；
2. 起点距离设为 0；
3. 在未访问的节点中，选取当前距离最小的节点 `u`；
4. 标记 `u` 为已访问；
5. 遍历 `u` 的邻接节点 `v`，尝试**松弛（Relaxation）**：
    - 若 `dist[v] > dist[u] + w(u, v)`，则更新 `dist[v]`；
6. 重复直到所有节点访问完毕。

##### **关键概念：**

- **松弛操作（Relaxation）：**
    - 检查当前路径是否比已知的更短；
    - 若是，则更新最短路径值；
    - 是最短路径算法的核心思想。

##### **复杂度：**

- 邻接矩阵：$O(n^2)$
- 邻接表 + 堆优化：$O(E \log V)$
##### **算法模板：**

```c ++
void dijksta(vector<vector<int>> &graph, int n) {
	vector<int> minDist(n + 1, INT_MAX);
	vector<bool> visited(n + 1, false);
	
	minDist[1] = 0;
	visited[1] = true;
	for(int i = 2; i <= n; i ++) {
		if(graph[1][i] != INT_MAX) minDist[i] = graph[1][i];
	}
	
	for(int i = 0; i < n - 1; i ++) {
		int cur = -1;
		int curMinVal = INT_MAX;
		
		for(int j = 2; j <= n; j ++) {
			if(!visited[j] && minDist[i] < curMinVal) {
				cur = j;
				curMinVal = minDist[j];
			}
		}
		visited[cur] = true;
		
		for(int j = 2; i <= n; j ++) {
			for(!visited[j] && graph[cur][j] != INT_MAX) {
				minDist[j] = min(minDist[j], minDist[cur] + graph[cur][j]);
			}
		}
	}
	
	cout << minDist[n] << endl;
}
```

---
#### 六、Bellman-Ford 算法（含负权边的最短路径）

##### **适用场景：**

- 有向图；
- **允许负权边**；
- 但**不能存在负权回路（负环）**。

##### **核心概念：**

- **松弛（Relaxation）**：  
    对所有边执行 $n-1$ 次松弛操作，确保所有路径最短；
- 若第 $n$ 次仍能更新路径，说明存在负环。

##### **算法思想：**

1. 初始化源点距离为 0，其他为 `INT_MAX`；
2. 重复 $n-1$ 次：
    - 对每条边 $(u, v, w)$：
        - 若 `dist[u] + w < dist[v]`，则更新 `dist[v]`；
3. 第 $n$ 次检测是否还能更新，若能则存在负环。

##### **复杂度：**

$O(VE)$，适用于边数较少或需要检测负环的情况。

##### **算法模板：**

```c++
void bellmanFord(vector<vector<int> &sides, int n) {
	vector<int> minDist(n + 1, INT_MAX);
	
	minDist[1] = 0;
	for(int i = 1; i < n; i ++) {
		vector<int> minDistCopy = minDist;
		for(auto &side : sides) {
			if(minDistCopy[side[0]] == INT_MAX) continue;
			minDist[side[1]] = min(minDist[side[1]], minDistCopy[side[0]] + side[2]);
		}
	}
	
	cout << minDist[n] << endl;
}
```

---

#### 七、Floyd-Warshall 算法（多源最短路径）

##### **适用场景：**

- 求任意两点之间的最短路径；
- 图可以有负权边（但不能有负环）。

##### **算法思想（动态规划思想）：**

- 状态定义：
    - `dp[i][j]` 表示节点 i 到 j 的最短路径；
- 状态转移方程：
    `dp[i][j] = min(dp[i][j], dp[i][k] + dp[k][j])`
- 含义：  
    若通过节点 `k` 能让 `i→j` 路径更短，则更新。

> [!NOTE] 理解：
> 为在单源查找的基础上实现多源，引入动态规划思想：通过构建 `k` 层作为中间路径层，探寻每个 `i->j` 路径经过 `k` 点会使得路径更短，短则更新

##### **步骤：**

1. 初始化距离矩阵；
2. 逐步将每个节点 `k` 作为中间点；
3. 更新所有 `i→j` 的最短距离。

##### **复杂度：**

$O(n^3)$，适用于节点数较少的图。

##### **算法模板：**

```c++
class Solution {
public:
	int floyd(vector<vector<int>> &graph, int n) {
		for(int k = 1; k <= n; k ++) {
			for(int i = 1; i <= n; i ++) {
				for(int j = 1; j <= n; j ++) {
					if(graph[i][k] != INT_MAX && graph[k][j] != INT_MAX) {
						graph[i][j] = min(graph[i][j], graph[i][k] + graph[k][j]);
					}
				}
			}
		}
	}
};

int main() {
	Solution sol;

	int n, m;
	cin >> n >> m;
	
	vector<vector<int>> graph(n + 1, vector<int>(n + 1, INT_MAX));
	for(int i = 1; i <= n; i ++) {
		graph[i][j] = 0;
	}
	
	for(int i = 0; i < m; i ++) {
		int s, t, v;
		cin >> s >> t >> v;
		graph[s][t] = v;
		graph[t][s] = v;
	}
	
	sol.floyd(graph, n);
	
	int k;
	cin >> k;
	
	for(int i = 0; i < k; i ++) {
		int start, end;
		cin >> start >> end;
		if(graph[start][end] != INT_MAX) cout << graph[start][end] << endl;
		else cout << -1 << endl;
	}
}
```

---
#### 八、A* 算法（启发式搜索算法）

##### **适用场景：**

- 路径搜索问题（如地图导航、棋盘寻路）；
- 在 BFS 的基础上引入启发式函数（Heuristic Function），对 `BFS` 探索方向进行剪枝

##### **核心思想：**

- A* 算法 = 实际代价 + 预估代价；
    - $f(x) = g(x) + h(x)$；
    - $g(x)$：从起点到当前点的实际代价；
    - $h(x)$：从当前点到目标点的预估代价（启发函数）。

##### **启发函数（Heuristic）：**

- 常见定义：
    - 曼哈顿距离：`abs(x1 - x2) + abs(y1 - y2)`
    - 欧几里得距离：`sqrt((x1-x2)^2 + (y1-y2)^2)`
    - 这里常用平方和简化计算。

##### **算法步骤：**

1. 将起点加入优先队列；
2. 每次从队列中取出 `f(x)` 最小的节点；
3. 若该节点为目标点，则结束；
4. 否则扩展其邻居节点，计算各邻居的 $f = g + h$；
5. 将未访问的邻居加入队列；
6. 重复直到队列为空。

##### **复杂度：**

与启发函数的设计有关，通常优于 BFS。

##### **算法模板：**

```c++
struct knight {
    int x, y;
    int g, h, f;
    bool operator < (const knight &k) const {
        return k.f < f;
    }
};

struct position {
    pair<int, int> start;
    pair<int, int> end;
};
  
class Solution {
private:
    vector<vector<int>> chessBoard;
    position pos;
    int direction[8][2] = {
        {-2, 1}, {-1, 2},
        {1, 2}, {2, 1},
        {2, -1}, {1, -2},
        {-1, -2}, {-2, -1}
    };
  
    int heuristic(const knight &k) {
        return (k.x - pos.end.first)*(k.x - pos.end.first) + (k.y - pos.end.second)*(k.y - pos.end.second);
    }
  
public:
    void knightAttack(const position &pos) {
        chessBoard.assign(1001, vector<int>(1001, 0));
        this->pos = pos;
  
        priority_queue<knight> pq;
        knight start;
        start.x = pos.start.first;
        start.y = pos.start.second;
        start.g = 0;
        start.h = heuristic(start);
        start.f = start.g + start.h;
        chessBoard[start.y][start.x] = 1;
        pq.push(start);
        while(!pq.empty()) {
            knight cur = pq.top();
            pq.pop();
  
            if(cur.x == pos.end.first && cur.y == pos.end.second) break;
            for(int i = 0; i < 8; i ++) {
                knight next;
                next.x = cur.x + direction[i][0];
                next.y = cur.y + direction[i][1];
                if(next.x < 1 || next.x > 1000 || next.y < 1 || next.y > 1000) continue;
                if(!chessBoard[next.y][next.x]) {
                    chessBoard[next.y][next.x] = chessBoard[cur.y][cur.x] + 1;
                    next.g = cur.g + 5;
                    next.h = heuristic(next);
                    next.f = next.g + next.h;
                    pq.push(next);
                }
            }
        }
        cout << chessBoard[pos.end.second][pos.end.first] - 1 << endl;
    }
};
  
int main() {
    Solution sol;
  
    int n;
    cin >> n;

    for(int i = 0; i < n; i ++) {
        int startX, startY, endX, endY;
        cin >> startX >> startY >> endX >> endY;
  
        position pos;
        pos.start = {startX, startY};
        pos.end = {endX, endY};
        sol.knightAttack(pos);
    }

    return 0;
}
```



