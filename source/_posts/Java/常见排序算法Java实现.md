---
title: 常见排序算法Java实现
cover: false
top: false
date: 2019-10-24 14:42:49
group: java
permalink: java-sorting
categories: Java后端
tags:
	- Java
	- 排序算法
keywords:
	- Java实现常见排序算法
summary: 用Java语言实现常见排序算法
---



本文总结常见排序算法，并用Java语言实现这些算法。

### 1\. 选择排序

#### 1.1 直接选择排序

```java
package longyg.algorithm.sorting.select;
 
import java.util.Arrays;
 
/**
 * 选择排序之：直接选择排序
 */
public class SelectSort {
 
    public static int[] selectSort(int[] data) {
        for (int i = 0; i < data.length - 1; i++) {
            for (int j = i+1; j < data.length; j++) {
                if (data[i] > data[j]) {
                    int tmp = data[i];
                    data[i] = data[j];
                    data[j] = tmp;
                }
            }
        }
        return data;
    }
 
    // 优化后的算法, 减少了交换次数
    public static int[] selectSort2(int[] data) {
        for (int i = 0; i < data.length - 1; i++) {
            int minIndex = i;
            for (int j = i + 1; j < data.length; j++) {
                if (data[minIndex] > data[j]) {
                    minIndex = j;
                }
            }
            if (minIndex != i) {
                int tmp = data[i];
                data[i] = data[minIndex];
                data[minIndex] = tmp;
            }
        }
        return data;
    }
 
    public static void main(String[] args) {
        System.out.println(Arrays.toString(selectSort(new int[] {23, 4, 34, 5, 10, 50, 29})));
        System.out.println(Arrays.toString(selectSort2(new int[] {23, 4, 34, 5, 10, 50, 29})));
    }
}
```

#### 1.2 堆排序
```java
package longyg.algorithm.sorting.select;
 
import java.util.Arrays;
 
/**
 * 选择排序之：堆排序
 */
public class HeapSort {
 
    // 从小到大排序
    public static int[] heapSort(int[] data) {
 
        for (int i = 0; i < data.length - 1; i++) {
            // 先建大顶堆
            buildMaxHeap(data, data.length - 1 - i);
            // 再把堆的root节点与数组的最后一个元素交换
            swap(data, 0, data.length - 1 - i);
        }
 
        return data;
    }
 
    // 从大到小排序
    public static int[] heapSort2(int[] data) {
 
        for (int i = 0; i < data.length - 1; i++) {
            // 先建小顶堆
            buildMinHeap(data, data.length - 1 - i);
            // 再把堆的root节点与数组的最后一个元素交换
            swap(data, 0, data.length - 1 - i);
        }
 
        return data;
    }
 
    // 建大顶堆
    private static void buildMaxHeap(int[] data, int lastIndex) {
        for (int i = (lastIndex - 1) / 2; i >= 0; i--) {
 
            int k = i;
 
            // 如果当前k节点存在子节点
            while (2*k + 1 <= lastIndex) {
                // 初始化biggerIndex为左子节点的索引
                int biggerIndex = 2*k + 1;
 
                // 如果当前k节点存在右子节点
                if (biggerIndex < lastIndex) {
                    // 比较左右子节点大小
                    if (data[biggerIndex] < data[biggerIndex + 1]) {
                        // 如果右子节点大，把biggerIndex设为右子节点的索引
                        biggerIndex++;
                    }
                }
                // 比较k节点与最大子节点
                if (data[k] < data[biggerIndex]) {
                    // 如果k比子节点小，则交换
                    swap(data, k, biggerIndex);
 
                    // 交换后需要循环子树，重新调整子树
                    k = biggerIndex;
                } else {
                    // 避免死循环
                    break;
                }
            }
 
        }
    }
 
    // 建小顶堆
    private static void buildMinHeap(int[] data, int lastIndex) {
        for (int i = (lastIndex - 1) / 2; i >= 0; i--) {
 
            int k = i;
 
            // 如果当前k节点存在子节点
            while (2*k + 1 <= lastIndex) {
                // 初始化lowerIndex为左子节点的索引
                int lowerIndex = 2*k + 1;
 
                // 如果当前k节点存在右子节点
                if (lowerIndex < lastIndex) {
                    // 比较左右子节点大小
                    if (data[lowerIndex] > data[lowerIndex + 1]) {
                        // 如果右子节点小，把lowerIndex设为右子节点的索引
                        lowerIndex++;
                    }
                }
                // 比较k节点与最大子节点
                if (data[k] > data[lowerIndex]) {
                    // 如果k比子节点大，则交换
                    swap(data, k, lowerIndex);
 
                    // 交换后需要循环子树，重新调整子树
                    k = lowerIndex;
                } else {
                    // 避免死循环
                    break;
                }
            }
        }
    }
 
    private static void swap(int[] data, int index1, int index2) {
        int tmp = data[index1];
        data[index1] = data[index2];
        data[index2] = tmp;
    }
 
    public static void main(String[] args) {
        System.out.println(Arrays.toString(heapSort(new int[] {23, 4, 34, 5, 10, 50, 29})));
        System.out.println(Arrays.toString(heapSort2(new int[] {23, 4, 34, 5, 10, 50, 29})));
    }
}
```

### 2\. 交换排序

#### 2.1 冒泡排序

```java
package longyg.algorithm.sorting.exchange;
 
import java.util.Arrays;
 
/**
 * 交换排序之：冒泡排序
 */
public class BubbleSort {
    public static int[] bubbleSort(int[] data) {
        for (int i = 0; i < data.length - 1; i++) {
            boolean flag = false; // 标记是否进行了交换
            for (int j = 0; j < data.length - 1 - i; j++) {
                if (data[j] > data[j + 1]) {
                    int tmp = data[j + 1];
                    data[j + 1] = data[j];
                    data[j] = tmp;
                    flag = true;
                }
            }
            // 如果没有进行过交换，说明数组已经是有序的了，即可提前结束
            if (!flag) {
                break;
            }
        }
        return data;
    }
 
    public static void main(String[] args) {
        System.out.println(Arrays.toString(bubbleSort(new int[] {23, 4, 34, 5, 10, 50, 29})));
    }
}
```

#### 2.2 快速排序
```java
package longyg.algorithm.sorting.exchange;
 
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;
 
/**
 * 交换排序之：快速排序
 */
public class QuickSort {
    public static int[] quickSort(int[] data) {
        subSort(data, 0, data.length - 1);
        return data;
    }
 
    private static void subSort(int[] data, int start, int end) {
        if (start >= end) return;
        int middle = divide(data, start, end);
        subSort(data, start, middle - 1);
        subSort(data, middle + 1, end);
    }
 
    private static int divide(int[] data, int start, int end) {
        int base = data[end];
 
        while (start < end) {
            while (start < end  && data[start] <= base) {
                start++;
            }
            if (start < end) {
                swap(data, start, end);
                end--;
            }
            while (start < end && data[end] >= base) {
                end--;
            }
            if (start < end) {
                swap(data, start, end);
                start++;
            }
        }
        return end;
    }
 
    private static void swap(int[] data, int index1, int index2) {
        int tmp = data[index1];
        data[index1] = data[index2];
        data[index2] = tmp;
    }
 
    public static void main(String[] args) {
        System.out.println(Arrays.toString(quickSort(new int[] {23, 4, 34, 5, 10, 50, 29})));
    }
}
```

### 3\. 插入排序

#### 3.1 直接插入排序
```java
package longyg.algorithm.sorting.insert;
 
import java.util.Arrays;
 
/**
 * 插入排序之：直接插入排序
 */
public class DirectInsertSort {
    public static int[] directInsertSort(int[] data) {
 
        for (int i = 1; i < data.length; i++) {
 
            int tmp = data[i];
 
            // 如果比前一个数小，说明需要插入前面的有序序列
            if (data[i] < data[i - 1]) {
                int j = i - 1;
                for (; j >= 0 && data[j] > tmp; j--) {
                    data[j+1] = data[j];
                }
                data[j+1] = tmp;
            }
        }
        return data;
    }
 
    public static void main(String[] args) {
        System.out.println(Arrays.toString(directInsertSort(new int[] {23, 4, 34, 5, 10, 50, 29})));
    }
 
}
```

#### 3.2 二分插入排序

```java
package longyg.algorithm.sorting.insert;
 
import java.util.Arrays;
 
/**
 * 插入排序之：二分插入排序
 */
public class BinaryInsertSort {
    public static int[] binaryInsertSort(int[] data) {
 
        for (int i = 1; i < data.length; i++) {
            int tmp = data[i];
 
            int low = 0;
            int high = i - 1;
 
            while (low <= high) {
                // mid 为low和high的中间索引
                int mid = (low + high) / 2;
                // 如果tmp值大于中间元素的值
                if (data[mid] < tmp) {
                    // 下一躺将在索引大于mid那一半中搜索
                    low = mid + 1;
                } else {
                    // 下一躺将在索引小于mid那一半中搜索
                    high = mid - 1;
                }
            }
            // 将low到i处的所有元素向后整体移一位
            for (int j = i; j > low; j--) {
                data[j] = data[j - 1];
            }
            // 将tmp插入合适位置
            data[low] = tmp;
        }
 
        return data;
    }
 
    public static void main(String[] args) {
        System.out.println(Arrays.toString(binaryInsertSort(new int[] {23, 4, 34, 5, 10, 50, 29})));
    }
 
}
```

#### 3.3 Shell排序

```java
package longyg.algorithm.sorting.insert;
 
import java.util.Arrays;
 
/**
 * 插入排序之：Shell排序
 */
public class ShellSort {
 
    public static int[] shellSort(int[] data) {
        int h = 1;
        while (h <= data.length/3) {
            h = h * 3 + 1;
        }
 
        while (h > 0) {
            System.out.println("h = " + h);
            for (int i = h; i < data.length; i++) {
                int tmp = data[i];
 
                if (data[i] < data[i - h]) {
                    int j = i - h;
                    // 整体后移h格
                    for (; j >=0 && data[j] > tmp; j -= h) {
                        data[j + h] = data[j];
                    }
                    data[j + h] = tmp;
                }
                System.out.println(Arrays.toString(data));
            }
            h = (h - 1) / 3;
        }
        return data;
    }
 
    public static void main(String[] args) {
        System.out.println(Arrays.toString(shellSort(new int[] {23, 4, 34, 5, 10, 50, 29})));
    }
}
```

### 4\. 归并排序

```java
package longyg.algorithm.sorting;
 
import java.util.Arrays;
 
/**
 * 归并排序
 */
public class MergeSort {
    public static int[] mergeSort(int[] data) {
 
        sort(data, 0, data.length - 1);
 
        return data;
    }
 
    private static void sort(int[] data, int left, int right) {
        if (left < right) {
            // 取中间索引
            int center = (left + right) / 2;
 
            // 对左边一半数组进行递归排序
            sort(data, left, center);
 
            // 对右边一半数组进行递归排序
            sort(data, center + 1, right);
 
            // 合并左右已排序的数组
            merge(data, left, center, right);
        }
    }
 
    private static void merge(int[] data, int left, int center, int right) {
        int i = left;
        int j = center + 1;
 
        // 临时数组，用于保存merge后的数组
        int[] tmpArr = new int[data.length];
        // 临时数组的索引变量
        int k = left;
 
        while (i <= center && j <= right) {
            if (data[i] <= data[j]) {
                tmpArr[k++] = data[i++];
            } else {
                tmpArr[k++] = data[j++];
            }
        }
 
        // 如果左边有多余元素，依次放入临时数组
        while (i <= center) {
            tmpArr[k++] = data[i++];
        }
        // 如果右边有多余元素，依次放入临时数组
        while (j <= right) {
            tmpArr[k++] = data[j++];
        }
 
        // 将临时数组的内容复制回原数组
        while (left <= right) {
            data[left] = tmpArr[left++];
        }
    }
 
    public static void main(String[] args) {
        System.out.println(Arrays.toString(mergeSort(new int[] {23, 4, 34, 5, 10, 50, 29, 78, 18})));
    }
}
```

### 5\. 桶式排序

```java
package longyg.algorithm.sorting;
 
import java.util.Arrays;
 
/**
 * 桶式排序
 */
public class BucketSort {
 
    public static int[] bucketSort(int[] data) {
        int min = min(data);
        int max = max(data) + 1;
        System.out.println(min + " - " + max);
        int[] tmpArr = new int[data.length];
 
        // 桶数组
        int[] buckets = new int[max - min];
        // 桶数组记录每个元素出现的次数
        for (int i = 0; i < data.length; i++) {
            buckets[data[i] - min]++;
        }
        System.out.println(Arrays.toString(buckets));
 
        for (int i = 1; i < max - min; i++) {
            buckets[i] = buckets[i] + buckets[i - 1];
        }
        System.out.println(Arrays.toString(buckets));
 
        System.arraycopy(data, 0, tmpArr, 0, data.length);
        for (int k = data.length - 1; k >= 0; k--) {
            data[--buckets[tmpArr[k] - min]] = tmpArr[k];
        }
 
        return data;
    }
 
    private static int min(int[] data) {
        int min = data[0];
        for (int i = 1; i < data.length; i++) {
            if (data[i] < min) {
                min = data[i];
            }
        }
        return min;
    }
 
    private static int max(int[] data) {
        int max = data[0];
        for (int i = 1; i < data.length; i++) {
            if (data[i] > max) {
                max = data[i];
            }
        }
        return max;
    }
 
    public static void main(String[] args) {
        System.out.println(Arrays.toString(bucketSort(new int[] {1, 7, 5, 4, 3})));
    }
}
```

### 6\. 基数排序

```java
package longyg.algorithm.sorting;
 
import java.util.Arrays;
 
/**
 * 基数排序
 */
public class MultiKeyRadixSort {
    public static int[] radixSort(int[] data, int radix, int d) {
        int[] tmp = new int[data.length];
        // 基于桶式排序
        int[] buckets = new int[radix];
        for (int i = 0, rate = 1; i < d; i++) {
            Arrays.fill(buckets, 0);
            System.arraycopy(data, 0, tmp, 0, data.length);
            for (int j = 0; j < data.length; j++) {
                int subKey = (tmp[j]/rate) % radix;
                buckets[subKey]++;
            }
 
            for (int j = 1; j < radix; j++) {
                buckets[j] = buckets[j] + buckets[j - 1];
            }
 
            for (int m = data.length - 1; m >= 0; m--) {
                int subKey = (tmp[m] / rate) % radix;
                data[--buckets[subKey]] = tmp[m];
            }
            rate *= radix;
 
            System.out.println(Arrays.toString(data));
        }
        return data;
    }
 
    public static void main(String[] args) {
        System.out.println(Arrays.toString(radixSort(new int[] {23, 43, 134, 5, 1110, 50, 219}, 10, 4)));
    }
}
```