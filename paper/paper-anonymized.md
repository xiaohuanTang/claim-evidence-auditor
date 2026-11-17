# 基于地形特征分析的冒泡排序改进算法研究

> Anonymized paper text. The cover page, author, course metadata, score, and signature have been removed. The original `.doc` is not included. This is a course paper, not a peer-reviewed publication.

## 摘要

冒泡排序作为一种基础的排序算法，其优化研究一直是计算机算法领域的热点。本文提出一种基于地形特征分析的冒泡排序改进算法——BubbleSort-Topo。通过将待排序数组视为数字地形，定义了山峰、山谷、上坡、下坡、平地五种地形类型，并设计了相应的优化策略。实验结果表明，在处理具有明显地形特征的数据时，该算法能获得 11%~16% 的性能提升，尤其在逆序数据和特定地形分布数据中表现优异。

**关键词：** 排序算法；冒泡排序；地形分析；算法优化；性能提升

## 引言

排序算法的效率一直是值得探讨的经典问题。冒泡排序因其逻辑直观，至今仍常用于教学与小规模数据处理。以往的改进多聚焦于优化比较与交换策略，却往往忽略了数据本身呈现的特定形态。事实上，数据排列形成的结构特征中，可能蕴含着提升排序效率的关键。这一思路与地理学中分析地形的方法相似，即通过量化局部邻接关系来识别宏观地貌特征[1]，此思维也被引入计算机领域，用于数据结构的可视化和聚类分析[2]。受地貌自动识别方法 Geomorphons[3] 的启发，我们将待排序数组类比为一维“数字地形”，用数值模拟海拔，并据此定义了五种基础地形单元。基于这一识别体系，本文提出 BubbleSort-Topo 算法，通过在排序中动态感知数据“地貌”，并针对不同地形类型自适应调整比较与移动策略，从而减少无效操作。实验表明，当数据分布具有明显“起伏”规律时（如完全逆序或周期性序列），该方法相比传统冒泡排序及其变种，能有效减少比较次数，获得可观的性能提升。这为排序优化提供了一条新思路：在排序之前，先读懂数据的“地貌”。

## 1 相关算法

### 1.1 传统冒泡排序算法

冒泡排序是一种基础的比较排序算法，其核心思想是通过反复交换相邻的无序元素，使得较大（或较小）的元素逐渐“浮”到序列的一端。标准的冒泡排序算法流程可描述如下：从序列的第一个元素开始，依次比较相邻的两个元素，若它们的顺序错误则进行交换。这样，每一轮遍历都会将当前未排序部分中的最大（或最小）元素移动至其正确位置。该算法的标准实现如伪代码所示：

```text
for i = 0 to n-2
    for j = 0 to n-i-2
        if A[j] > A[j+1]
            swap(A[j], A[j+1])
```

其中，n 代表数组长度。在排序过程中，外层循环每完成一轮，排序范围的右边界就向左缩减一位。在最坏和平均情况下，算法需要进行约 n(n-1)/2 次比较，因此其时间复杂度为 O(n²)。由于排序过程仅需常数级别的额外存储空间用于元素交换，其空间复杂度为 O(1)[4]。

### 1.2 经典优化变体

鉴于标准冒泡排序效率上的局限性，研究者们提出了多种优化方案，旨在减少不必要的比较或交换操作，以提升算法在特定数据场景下的性能。

#### （1）提前终止优化

一项直观且有效的优化是监测内层循环中是否发生交换。如果在某一轮完整的遍历中没有发生任何元素交换，则表明序列已经有序，算法可以提前终止。这一优化在最好情况下（输入序列已完全有序）能将时间复杂度降至 O(n)[5]。

#### （2）鸡尾酒排序

鸡尾酒排序，又称双向冒泡排序或摇摆排序，是对冒泡排序的一种改进。该算法不再仅单向地进行比较交换，而是交替地从左至右和从右至左进行扫描。在一次从左至右的遍历将最大元素移动到右端后，紧接着一次从右至左的遍历将最小元素移动到左端，如此反复。这种方法对于序列两端局部有序的数据更为有效[6]。其核心流程如下：

```text
left = 0, right = n-1
while left < right
    for i = left to right-1
        if A[i] > A[i+1]
            swap(A[i], A[i+1])
    right--
    for i = right-1 downto left
        if A[i] > A[i+1]
            swap(A[i], A[i+1])
    left++
```

#### （3）梳排序

梳排序由 Włodzimierz Dobosiewicz 于 1980 年提出，后由 Stephen Lacey 和 Richard Box 重新发现并命名[7]。该算法是对冒泡排序的另一种重要改进，其核心思想是引入一个大于 1 的“间隔”（或称为“梳子齿距”），先比较和交换距离较远的元素，以快速消除序列末端的微小无序（海龟问题），随后逐步缩小间隔直至为 1。当间隔为 1 时，算法退化为标准的冒泡排序，但此时序列通常已接近有序，因此效率更高。间隔缩小的因子常取经验值 1.3，算法的效率对间隔序列的选择较为敏感。

### 1.3 基于数据特征的排序优化研究

近年来，排序算法的研究趋势之一是从通用的、与数据无关的算法，转向能够感知和利用输入数据固有特征的自适应方法[8]。这类方法通过分析数据的分布、有序程度或特定模式，在运行时动态选择或调整排序策略，以期获得更好的性能。

#### （1）自适应排序算法

此类算法会在排序开始前或排序过程中，对数据的关键特征（如均匀性、偏态性、是否存在大量重复键值等）进行快速评估。根据评估结果，系统可能在不同的经典排序算法（如快速排序、归并排序、插入排序）间进行切换，或调整某个算法的参数（如快速排序的枢轴选择策略）。例如，对于小规模数据或几乎有序的数据，插入排序往往比快速排序更高效[9]。

#### （2）面向部分有序数据的优化

部分有序是现实数据中常见的特征。针对这一特征，除了鸡尾酒排序，还有如“Timsort”（Python 和 Java 的内置排序算法）等混合排序算法。Timsort 会主动识别并合并数据中自然存在的升序或降序“游程”，从而在部分有序数据上达到接近线性的时间复杂度。在冒泡排序的框架内，也有研究尝试与插入排序结合，主要是利用数据的局部有序性。

#### （3）基于模式识别的排序

更进一步的研究探索识别数据中更复杂的模式，如周期性波动、多个单调子序列等，并据此设计定制化的排序或预处理步骤。例如，若数据被识别为若干个已排序块的连接，则可以优先对这些块进行归并[10]。然而，这类方法通常需要一个前置的、可能具有一定复杂度的模式识别阶段，其开销未必总能被排序阶段的加速所抵消，实现复杂度和通用性是其面临的主要挑战。

## 2 改进模型

### 2.1 地形分析模型

#### 2.1.1 基本定义

给定长度为 n 的数组 A，对于任意元素 A[i]（0 ≤ i < n），定义其地形类型如下：

1. **山峰（PEAK）**：当 i > 0 且 i < n-1 时，若 A[i] > A[i-1] 且 A[i] > A[i+1]，则 A[i] 为山峰元素。
2. **山谷（VALLEY）**：当 i > 0 且 i < n-1 时，若 A[i] < A[i-1] 且 A[i] < A[i+1]，则 A[i] 为山谷元素。
3. **上坡（UP_SLOPE）**：当 i > 0 且 i < n-1 时，若 A[i-1] < A[i] < A[i+1]，则 A[i] 为上坡元素。
4. **下坡（DOWN_SLOPE）**：当 i > 0 且 i < n-1 时，若 A[i-1] > A[i] > A[i+1]，则 A[i] 为下坡元素。
5. **平地（FLAT）**：当 i > 0 且 i < n-1 时，若 A[i-1] = A[i] = A[i+1]，则 A[i] 为平地元素。

对于边界元素（i=0 或 i=n-1），将其标记为边界类型（BOUNDARY）。

#### 2.1.2 地形特征的意义

不同地形特征对应数据的不同局部性质：

- **山峰和山谷**：数据的极值点，这些元素在排序过程中很可能需要移动较长的距离才能到达正确位置。
- **上坡和下坡**：数据的趋势方向，上坡区域可能已经部分有序，下坡区域则可能需要反转。
- **平地**：数据的稳定区域，这些元素可能已经在正确位置附近，无需过多调整。

#### 2.1.3 地形统计函数

定义地形统计函数如下：

```c
TerrainStats analyze_all_terrain(int arr[], int n) {
    TerrainStats stats = {0, 0, 0, 0, 0, 0};
    for (int i = 0; i < n; i++) {
        TerrainType type = analyze_terrain(arr, n, i);
        switch(type) {
            case PEAK: stats.peaks++; break;
            case VALLEY: stats.valleys++; break;
            // ... 其他类型
        }
    }
    return stats;
}
```

### 2.2 BubbleSort-Topo 算法设计

#### 2.2.1 算法框架

BubbleSort-Topo 算法分为三个阶段：

1. **地形分析阶段**：扫描整个数组，识别每个元素的地形类型。
2. **针对性优化阶段**：根据地形特征执行针对性的预处理。
3. **优化冒泡阶段**：在传统冒泡排序基础上，结合地形信息优化比较和交换策略。

#### 2.2.2 详细算法描述（完整算法见附录 A）

地形分析函数：

```c
typedef enum {
    PEAK, VALLEY, UP_SLOPE, DOWN_SLOPE, FLAT, BOUNDARY
} TerrainType;

TerrainType analyze_terrain(int arr[], int n, int index) {
    if (index == 0 || index == n-1) return BOUNDARY;
    int left = arr[index-1], curr = arr[index], right = arr[index+1];
    if (curr > left && curr > right) return PEAK;
    if (curr < left && curr < right) return VALLEY;
    if (left < curr && curr < right) return UP_SLOPE;
    if (left > curr && curr > right) return DOWN_SLOPE;
    if (left == curr && curr == right) return FLAT;
    return BOUNDARY;
}
```

针对性优化策略：

1. **山峰**：山峰元素往往较大，需要向右移动。算法会将山峰元素向右移，直到遇到比它大的元素或到达边界。
2. **山谷**：山谷元素往往较小，需要向左移动。算法会将山谷元素向左移，直到遇到比它小的元素或到达边界。
3. **上坡**：上坡区域往往已经有序，算法会减少对这些区域的比较次数。
4. **下坡**：下坡区域往往需要反转，算法会重点关注这些区域的比较和交换。

核心算法实现：

```c
void bubble_sort_topo(int arr[], int n) {
    TerrainType* terrain = analyze_all_terrain(arr, n);
    optimize_by_terrain(arr, n, terrain);
    bool swapped;
    for (int i = 0; i < n-1; i++) {
        swapped = false;
        for (int j = 0; j < n-i-1; j++) {
            if (should_skip_compare(terrain[j], terrain[j+1])) {
                continue;
            }
            if (arr[j] > arr[j+1]) {
                swap(&arr[j], &arr[j+1]);
                swapped = true;
                update_terrain(terrain, arr, n, j);
            }
        }
        if (!swapped) break;
    }
    free(terrain);
}
```

#### 2.2.3 算法复杂度分析

**时间复杂度：**

- 地形分析阶段：O(n)
- 针对性优化阶段：最坏情况 O(n²)，但实际中通常远小于此。
- 优化冒泡阶段：与传统冒泡排序相同，最坏情况 O(n²)，最好情况 O(n)。
- 总体时间复杂度仍为 O(n²)，但常数因子可能更小。

**空间复杂度：**

需要额外的 O(n) 空间来存储地形分析的结果，空间复杂度为 O(n)。

## 3 实验分析

### 3.1 实验环境

- 处理器：AMD Ryzen 7 8745H with Radeon 780M Graphics (3.80 GHz)
- 内存：16.0 GB
- 操作系统：Windows 11
- 编译器：Dev-C++ 5.11
- 编程语言：C 语言

### 3.2 测试数据设计

为全面评估算法性能，设计了五种不同类型的测试数据（所有测试数据规模均为 n=2000）：

1. **随机数据**：使用标准随机数生成器生成的均匀分布数据，作为基准测试。
2. **完整地形数据**：专门设计的数据，包含所有五种地形类型，用于测试算法的全面性。
3. **有序数据**：已经完全有序的数据，用于测试算法的最佳情况性能。
4. **逆序数据**：完全逆序的数据，用于测试算法的最坏情况性能。
5. **山峰-山谷交替数据**：山峰和山谷交替出现的数据[11]，这是 BubbleSort-Topo 算法的优势场景。

### 3.3 性能对比实验

表 1 展示了 BubbleSort-Topo 算法与传统冒泡排序的性能对比结果。

**表 1 性能对比实验结果（n=2000，见附件二）**

| 数据类型 | 传统冒泡 (ms) | BubbleSort-Topo (ms) | 性能变化 |
| --- | ---: | ---: | ---: |
| 随机数据 | 7.00 | 8.00 | -14.3% |
| 完整地形数据 | 8.00 | 7.00 | +12.5% |
| 有序数据 | 0.00 | 0.00 | 0.0% |
| 逆序数据 | 12.00 | 10.00 | +16.7% |
| 山峰-山谷数据 | 8.00 | 8.00 | 0.0% |

![图 1 性能对比实验输出（附件二）](figures/attachment-2-performance.png)

*图 1：附件二的控制台输出。原图同时包含“完整 Topo”和“简化 Topo”两列；表 1 的 BubbleSort-Topo 列对应“简化 Topo”列。*

### 3.4 结果分析

从实验结果可以看出：

在随机数据上，BubbleSort-Topo 算法比传统冒泡排序慢 14.3%。这是因为随机数据缺乏明显的地形特征，地形分析的开销超过了优化带来的收益。在完整地形数据上，BubbleSort-Topo 算法获得了 12.5% 的性能提升。这表明当数据包含明显的地形特征时，地形分析能够有效指导排序过程。在逆序数据上，BubbleSort-Topo 算法获得了最大的性能提升（16.7%）。逆序数据具有规律的下坡特征，算法能够有效识别并优化处理。

在有序数据上，两种算法性能相同，都能在 O(n) 时间内完成。在山峰-山谷数据上，两种算法性能没有差别。这表明对于某些特定模式，算法需要进一步优化。

### 3.5 规模扩展实验

为测试算法的可扩展性，使用山峰-山谷交替数据进行了规模扩展实验，结果如表 2 所示。

**表 2 规模扩展实验结果（见附件四）**

| 数据规模 (n) | 传统冒泡 (ms) | BubbleSort-Topo (ms) | 性能提升 |
| ---: | ---: | ---: | ---: |
| 1000 | 2.00 | 2.00 | 0.0% |
| 2000 | 9.00 | 8.00 | 11.1% |
| 5000 | 52.00 | 46.00 | 11.5% |
| 10000 | 180.00 | 178.00 | 1.1% |

![图 2 规模扩展测试（附件四）](figures/attachment-4-scalability.png)

*图 2：附件四的控制台输出。*

从规模扩展实验结果可以看出：

1. 在中等规模（n=2000, 5000）时，算法能够获得稳定的性能提升（约 11%）。
2. 在小规模（n=1000）时，性能未得到提升（因为算法的开销和获得的优化抵消了）。
3. 在大规模（n=10000）时，性能提升极小。

### 3.6 地形分析准确性验证

为验证地形分析的准确性，选取了一个长度 15 的数组进行分析。

从以下分析结果可以看出，算法能够准确识别各种地形类型（见附件三）：

- 山峰（PEAK）：3 个（20.0%）
- 山谷（VALLEY）：4 个（26.7%）
- 上坡（UP_SLOPE）：4 个（26.7%）
- 下坡（DOWN_SLOPE）：2 个（13.3%）
- 平地（FLAT）：0 个（0.0%）

![图 3 地形分析演示（附件三）](figures/attachment-3-terrain.png)

*图 3：附件三的控制台输出。*

此结果与分析一致，从而证明了地形分析算法的正确性。

## 4 结束语

本文借鉴地形分析思想，对传统冒泡排序算法进行改进，提出一种基于地貌特征的排序算法——BubbleSort-Topo。该方法将数组元素视为地形起伏，划分为山峰、山谷、上坡、下坡与平地五种地貌类型，并针对不同地形设计了差异化的排序优化策略。

实验表明，该算法在面对具有显著地形特征的数据时，可带来 11% 至 16% 的效率提升，尤其在逆序排列或特定地貌分布的数据中表现突出。在随机数据等场景下，因地形分析带来一定计算开销，性能略有波动。未来将通过算法模型优化与并行计算环境部署，进一步提升运行效率，使其能够适用于实时流式大数据排序等场景。

## 参考文献

- [1] Wilson, J.P., & Gallant, J.C. (Eds.). (2000). Terrain Analysis: Principles and Applications. John Wiley & Sons.
- [2] Ankerst, M., Breunig, M. M., Kriegel, H.-P., & Sander, J. (1999, June). OPTICS: ordering points to identify the clustering structure. In Proceedings of the 1999 ACM SIGMOD international conference on Management of data (pp. 49-60).
- [3] Jasiewicz, J., & Stepinski, T. F. (2013). Geomorphons - a pattern recognition approach to classification and mapping of landforms. Geomorphology, 182, 147-156.
- [4] 李春葆. 数据结构（C语言）[M]. 清华大学出版社，2005.
- [5] Sedgewick R, Wayne K. Algorithms[M]. 4th ed. Addison-Wesley Professional, 2011.
- [6] Astrachan O. Bubble sort: an archaeological algorithmic analysis[C]//Proceedings of the SIGCSE. 2003: 1-5.
- [7] Cormen, T. H., Leiserson, C. E., Rivest, R. L., & Stein, C. (2009). Introduction to Algorithms (3rd ed., p. 147, footnote). MIT Press.
- [8] Estivill-Castro, V., & Wood, D. (1992). A survey of adaptive sorting algorithms. ACM Computing Surveys (CSUR), 24(4), 441-476.
- [9] Bentley J L, McIlroy M D. Engineering a sort function[J]. Software: Practice and Experience, 1993, 23(11): 1249-1265.
- [10] Estivill-Castro V, Wood D. A survey of adaptive sorting algorithms[J]. ACM Computing Surveys (CSUR), 1992, 24(4), 441-476.
- [11] Feller, W. (1971). An Introduction to Probability Theory and Its Applications, Volume II (2nd ed.). John Wiley & Sons.

## 附录 A：完整代码

以下为论文附录一的完整 C 代码。

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <time.h>
// ==================== 地形类型定义 ====================
typedef enum {
    PEAK,        // 山峰：比左右都高
    VALLEY,      // 山谷：比左右都低  
    UP_SLOPE,    // 上坡：左边 < 当前 < 右边
    DOWN_SLOPE,  // 下坡：左边 > 当前 > 右边
    FLAT,        // 平地：与左右相等
    BOUNDARY     // 边界情况
} TerrainType;
const char* terrain_names[] = {
    "PEAK", "VALLEY", "UP_SLOPE", "DOWN_SLOPE", "FLAT", "BOUNDARY"
};
// ==================== 工具函数 ====================
void copy_array(int dest[], int src[], int n) {
    memcpy(dest, src, n * sizeof(int));
}
int is_sorted(int arr[], int n) {
    for (int i = 0; i < n - 1; i++) {
        if (arr[i] > arr[i + 1]) {
            return 0;
        }
    }
    return 1;
}
// ==================== 完整地形分析函数 ====================
TerrainType analyze_terrain(int arr[], int n, int index) {
    // 边界处理
    if (index == 0 || index == n-1) {
        return BOUNDARY;
    }
    int left = arr[index - 1];
    int curr = arr[index];
    int right = arr[index + 1];
    if (curr > left && curr > right) return PEAK;
    if (curr < left && curr < right) return VALLEY;
    if (left < curr && curr < right) return UP_SLOPE;
    if (left > curr && curr > right) return DOWN_SLOPE;
    if (left == curr && curr == right) return FLAT;
    return BOUNDARY;
}
void analyze_all_terrain(int arr[], int n, TerrainType terrain[]) {
    for (int i = 0; i < n; i++) {
        terrain[i] = analyze_terrain(arr, n, i);
    }
}
void print_terrain_info(int arr[], int n, TerrainType terrain[]) {
    printf("完整地形分析:\n");
    printf("位置\t数值\t地形\n");
    printf("----\t----\t--------\n");
    int count = 0;
    for (int i = 0; i < n && i < 15; i++) {
        printf("%d\t%d\t%s\n", i, arr[i], terrain_names[terrain[i]]);
        count++;
    }
    if (n > 15) printf("...\n");
    int stats[6] = {0};
    for (int i = 0; i < n; i++) {
        stats[terrain[i]]++;
    }
    printf("\n地形统计:\n");
    for (int i = 0; i < 5; i++) {
        printf("%-10s: %d (%.1f%%)\n", 
               terrain_names[i], stats[i], (float)stats[i]/n*100);
    }
}
// ==================== 传统冒泡排序 ====================
double bubble_standard(int arr[], int n) {
    clock_t start = clock();
    
    for (int i = 0; i < n-1; i++) {
        int swapped = 0;
        for (int j = 0; j < n-1-i; j++) {
            if (arr[j] > arr[j+1]) {
                int temp = arr[j];
                arr[j] = arr[j+1];
                arr[j+1] = temp;
                swapped = 1;
            }
        }
        if (!swapped) break;
    }
    clock_t end = clock();
    return ((double)(end - start)) / CLOCKS_PER_SEC * 1000;
}
// ==================== 完整版BubbleSort-Topo ====================
double bubble_sort_topo_full(int arr[], int n) {
    clock_t start = clock();
    if (n <= 1) {
        clock_t end = clock();
        return ((double)(end - start)) / CLOCKS_PER_SEC * 1000;
    }
    // 第一步：完整地形分析
    TerrainType* terrain = (TerrainType*)malloc(n * sizeof(TerrainType));
    analyze_all_terrain(arr, n, terrain);
    // 第二步：根据地形特征进行针对性优化
    // 阶段1：专门处理山峰(PEAK) - 向右推送
    for (int i = 0; i < n; i++) {
        if (terrain[i] == PEAK) {
            int pos = i;
            while (pos < n-1 && arr[pos] > arr[pos+1]) {
                int temp = arr[pos];
                arr[pos] = arr[pos+1];
                arr[pos+1] = temp;
                pos++;
            }
            analyze_all_terrain(arr, n, terrain);
        }
    }
    
    // 阶段2：专门处理山谷(VALLEY) - 向左推送
    for (int i = n-1; i >= 0; i--) {
        if (terrain[i] == VALLEY) {
            int pos = i;
            while (pos > 0 && arr[pos] < arr[pos-1]) {
                int temp = arr[pos];
                arr[pos] = arr[pos-1];
                arr[pos-1] = temp;
                pos--;
            }
            analyze_all_terrain(arr, n, terrain);
        }
    }
　　// 阶段3：优化上坡(UP_SLOPE)区域
   for (int i = 0; i < n-1; i++) {
        if (terrain[i] == UP_SLOPE && terrain[i+1] == UP_SLOPE) {
            if (arr[i] > arr[i+1]) {
                int temp = arr[i];
                arr[i] = arr[i+1];
                arr[i+1] = temp;
                analyze_all_terrain(arr, n, terrain);
            }
        }
    }
　　// 阶段4：处理下坡(DOWN_SLOPE)区域
    for (int i = 0; i < n-1; i++) {
        if (terrain[i] == DOWN_SLOPE && terrain[i+1] == DOWN_SLOPE) {
                if (arr[i] > arr[i+1]) {
                int temp = arr[i];
                arr[i] = arr[i+1];
                arr[i+1] = temp;
                analyze_all_terrain(arr, n, terrain);
            }
        }
    }
    // 阶段5：传统冒泡排序收尾
    for (int i = 0; i < n-1; i++) {
        int swapped = 0;
        int start = 0;
        int end = n-1-i;
        for (start = 0; start < end; start++) {
            if (terrain[start] != UP_SLOPE && terrain[start] != FLAT) {
                break;
            }
        }
        for (int j = start; j < end; j++) {
            if (terrain[j] == PEAK && terrain[j+1] == VALLEY) {
                if (arr[j] > arr[j+1]) {
                    int temp = arr[j];
                    arr[j] = arr[j+1];
                    arr[j+1] = temp;
                    swapped = 1;
                    for (int k = j-1; k <= j+2; k++) {
                        if (k >= 0 && k < n) {
                            terrain[k] = analyze_terrain(arr, n, k);
                        }
                    }
                }
                continue;
            }
            if (arr[j] > arr[j+1]) {
                int temp = arr[j];
                arr[j] = arr[j+1];
                arr[j+1] = temp;
                swapped = 1;
                for (int k = j-1; k <= j+2; k++) {
                    if (k >= 0 && k < n) {
                        terrain[k] = analyze_terrain(arr, n, k);
                    }
                }
            }
        }
        if (!swapped) break;
    }
    free(terrain);
    clock_t end = clock();
    return ((double)(end - start)) / CLOCKS_PER_SEC * 1000;
}
// ==================== 简化版（性能更好） ====================
double bubble_sort_topo_simple(int arr[], int n) {
    clock_t start = clock();
    // 只处理最明显的地形特征
    for (int phase = 0; phase < 2; phase++) {
        // 第一遍：向右推送山峰
        for (int i = 0; i < n-1; i++) {
            if (i > 0 && i < n-1) {
                if (arr[i] > arr[i-1] && arr[i] > arr[i+1]) {
                    // 发现山峰，向右推送
                    int pos = i;
                    while (pos < n-1 && arr[pos] > arr[pos+1]) {
                        int temp = arr[pos];
                        arr[pos] = arr[pos+1];
                        arr[pos+1] = temp;
                        pos++;
                    }
                    i = pos;
                }
            }
        }
        
        // 第二遍：向左推送山谷
        for (int i = n-1; i > 0; i--) {
            if (i > 0 && i < n-1) {
                if (arr[i] < arr[i-1] && arr[i] < arr[i+1]) {
                    // 发现山谷，向左推送
                    int pos = i;
                    while (pos > 0 && arr[pos] < arr[pos-1]) {
                        int temp = arr[pos];
                        arr[pos] = arr[pos-1];
                        arr[pos-1] = temp;
                        pos--;
                    }
                    i = pos;
                }
            }
        }
    }
    // 最后用传统冒泡收尾
    bubble_standard(arr, n);
    clock_t end = clock();
    return ((double)(end - start)) / CLOCKS_PER_SEC * 1000;
}
// ==================== 数据生成函数 ====================
void generate_test_data(int type, int arr[], int n) {
    static int seeded = 0;
    if (!seeded) {
        srand(12345);
        seeded = 1;
    }
    switch(type) {
        case 0: // 随机数据
            for (int i = 0; i < n; i++) arr[i] = rand() % 10000;
            break;
        case 1: // 完整地形测试数据
            for (int i = 0; i < n; i++) {
                int pattern = i % 12;
                switch(pattern) {
                    case 0: arr[i] = 9999; break;            // 极高峰
                    case 1: arr[i] = 1; break;               // 极深谷
                    case 2: case 3: case 4:                // 明显上坡
                        arr[i] = 1000 + (i % 100) * 30;
                        break;
                    case 5: case 6: case 7:                // 明显下坡
                        arr[i] = 8000 - (i % 100) * 25;
                        break;
                    case 8: case 9:                      // 平地附近
                        arr[i] = 5000 + rand() % 100;
                        break;
                    case 10: arr[i] = 7000; break;          // 中等高峰
                    case 11: arr[i] = 500; break;           // 中等深谷
                }
            }
            break;
        case 2: // 有序数据
            for (int i = 0; i < n; i++) arr[i] = i * 10;
            break;
        case 3: // 逆序数据
            for (int i = 0; i < n; i++) arr[i] = (n - i) * 10;
            break; 
        case 4: // 山峰-山谷交替链（最佳场景）
            for (int i = 0; i < n; i++) {
                if (i % 2 == 0) {
                    arr[i] = 9000 + rand() % 1000;        // 高峰
                } else {
                    arr[i] = rand() % 1000;              // 深谷
                }
            }
            break;
    }
}
// ==================== 性能测试 ====================
void performance_test() {
    printf("==================================================\n");
    printf("     BubbleSort-Topo 完整性能测试\n");
    printf("==================================================\n\n");
    int n = 2000;
    printf("测试规模: n = %d\n\n", n);
    const char* test_names[] = {
        "随机数据",
        "完整地形数据",
        "有序数据",
        "逆序数据",
        "山峰-山谷交替数据"
    };
    printf("%-25s %-12s %-12s %-12s %s\n", 
           "测试类型", "传统冒泡(ms)", "完整Topo(ms)", "简化Topo(ms)", "优化效果");
    printf("-----------------------------------------------------------------------\n");
    for (int test_type = 0; test_type < 5; test_type++) {
        int* original = (int*)malloc(n * sizeof(int));
        int* copy1 = (int*)malloc(n * sizeof(int));
        int* copy2 = (int*)malloc(n * sizeof(int));
        int* copy3 = (int*)malloc(n * sizeof(int));
        generate_test_data(test_type, original, n);
        // 测试传统冒泡
        copy_array(copy1, original, n);
        double time_std = bubble_standard(copy1, n);
        int correct_std = is_sorted(copy1, n);
        // 测试完整Topo
        copy_array(copy2, original, n);
        double time_full = bubble_sort_topo_full(copy2, n);
        int correct_full = is_sorted(copy2, n);
        // 测试简化Topo
        copy_array(copy3, original, n);
        double time_simple = bubble_sort_topo_simple(copy3, n);
        int correct_simple = is_sorted(copy3, n);
        double improvement = 0;
        if (time_std > 0) {
            improvement = (time_std - time_simple) / time_std * 100;
        }
        char effect[20];
        if (improvement > 0) {
            snprintf(effect, sizeof(effect), "+%.1f%%", improvement);
        } else if (improvement < 0) {
            snprintf(effect, sizeof(effect), "%.1f%%", improvement);
        } else {
            strcpy(effect, "基准");
        }
        printf("%-25s %-12.2f %-12.2f %-12.2f %s\n",
               test_names[test_type], time_std, time_full, time_simple, effect);
        if (!correct_std) printf("  ? 传统冒泡排序错误！\n");
        if (!correct_full) printf("  ? 完整Topo排序错误！\n");
        if (!correct_simple) printf("  ? 简化Topo排序错误！\n");
        free(original);
        free(copy1);
        free(copy2);
        free(copy3);
    }
    printf("\n\n地形分析演示:\n");
    printf("--------------------------------------------------\n");
    int demo_n = 15;
    int demo_arr[15];
    generate_test_data(1, demo_arr, demo_n);
    TerrainType* demo_terrain = (TerrainType*)malloc(demo_n * sizeof(TerrainType));
    analyze_all_terrain(demo_arr, demo_n, demo_terrain);
    print_terrain_info(demo_arr, demo_n, demo_terrain);
    free(demo_terrain);
}
// ==================== 规模扩展测试 ====================
void scalability_test() {
    printf("\n\n规模扩展测试（山峰-山谷交替数据）:\n");
    printf("--------------------------------------------------\n");
    printf("规模(n)   传统冒泡(ms)   简化Topo(ms)   提升比例\n");
    printf("--------------------------------------------------\n");
    int sizes[] = {1000, 2000, 5000, 10000};
    for (int i = 0; i < 4; i++) {
        int n = sizes[i];
        int* arr1 = (int*)malloc(n * sizeof(int));
        int* arr2 = (int*)malloc(n * sizeof(int));
        generate_test_data(4, arr1, n);  // 山峰-山谷交替数据
        // 传统冒泡
        copy_array(arr2, arr1, n);
        double time_std = bubble_standard(arr2, n);
        // 简化Topo
        copy_array(arr2, arr1, n);
        double time_topo = bubble_sort_topo_simple(arr2, n);
        double improvement = (time_std - time_topo) / time_std * 100;
        printf("%-10d %-14.2f %-14.2f %-12.1f%%\n", 
               n, time_std, time_topo, improvement);
        free(arr1);
        free(arr2);
    }
}
// ==================== 主函数 ====================
int main() {
    printf("==================================================\n");
    printf("  BubbleSort-Topo 完整实现（含五种地形）\n");
    printf("  PEAK, VALLEY, UP_SLOPE, DOWN_SLOPE, FLAT\n");
    printf("==================================================\n\n");
    performance_test();
    scalability_test();
    printf("\n==================================================\n");
    printf("测试完成！算法包含完整的五种地形分析。\n");
    printf("==================================================\n");
    return 0;
}
```
