---
layout: post
title: "Binary Search"
date:   2024-03-23
tags: [algorithm]
comments: true
author: Gnonymous
---

#### 有一个序列是某个有序系列围绕着下标为K的元素（0 <= k <length）旋转得到的序列，使数组下标变为 [k], [k+1], …, [n-1], [0], [1], …, [k-1]，如 123456围绕着下标为3的元素旋转得到456123，请为此序列编写元素查找算法，并分析你的算法性能。

> 基础的二分查找（两种）：
>
> 1. <span id = "first">先排序，按照有序性二分查找</span>
> 2. （乱序）分治的思想，不断二分查找
>
> 这题看似有序，也不是完全有序，利用特殊的有序性，进行类似第一种“[排序后的二分查找](#first)”

#### 思路：

为了解决这个问题，我们可以设计一个二分查找算法来查找给定元素在旋转数组中的位置。二分查找算法的基本思想是在有序的区间内通过比较中间元素与目标值的关系来缩小搜索范围。对于旋转数组，我们可以利用数组的有序性质来确定如何进行二分查找。

以下是查找算法的步骤：

1. 初始化两个指针 `left` 和 `right`，分别指向数组的起始位置和结束位置（即 `left = 0` 和 `right = length - 1`）。

2. 当 `left` 小于等于 `right` 时，执行以下步骤：
   a. 计算中间位置 `mid = (left + right) / 2`。
   b. 检查 `mid` 位置的元素是否是我们要找的元素，如果是，则返回 `mid`。
   c. 判断 `mid` 元素与 `left` 端点元素的大小关系：
      - 如果 `array[mid] >= array[left]`，说明 `array[left]` 到 `array[mid]` 这一段是有序的。
         - 如果目标值 `target` 位于 `array[left]` 和 `array[mid]` 之间，那么 `target` 一定在 `left` 和 `mid` 之间，更新 `right = mid - 1`。
         - 否则，`target` 在 `mid` 和 `right` 之间，更新 `left = mid + 1`。
      - 否则，`mid` 右侧是有序的。
         - 如果目标值 `target` 位于 `array[mid]` 和 `array[right]` 之间，那么 `target` 一定在 `mid` 和 `right` 之间，更新 `left = mid + 1`。
         - 否则，`target` 在 `left` 和 `mid` 之间，更新 `right = mid - 1`。

3. 如果在数组中没有找到目标值，返回 `-1` 表示元素不存在。

算法性能分析：
- 时间复杂度：由于每次迭代都会将搜索区间减半，所以这个算法的时间复杂度是 O(log n)，其中 n 是数组的长度。
- 空间复杂度：这个算法是原地进行的，不需要额外的存储空间，因此空间复杂度是 O(1)。

这个算法比线性搜索更高效，尤其是在大型数组中查找元素时。通过二分查找，我们可以快速定位到元素的大致位置，从而减少不必要的比较次数。

#### 代码：

~~~c++
#include <iostream>
using namespace std;
 
int search(int nums[],int left,int right,int target) {
	while(left<=right){
		int mid = left+(right-left)/2;
		if(nums[mid]==target){
			return mid;
		}
		//左边有序 
		if(nums[left]<nums[mid]){
			if(nums[left]<=target&&nums[mid]>=target){
				right = mid - 1;
			}else{
				left = mid + 1;
			}
		}else{
			if(nums[mid]<target&&target<=nums[right]){
				left = mid +1;
			}else{
				right = mid -1;
			}
		}
	}
	return -1;
}

int main() {
    int nums[] = {4, 5, 6, 7, 8, 0, 1, 2};
    int target = 0; // 假设我们要查找的元素是数组中的一个
    int n = sizeof(nums) / sizeof(nums[0]);
    int index = search(nums, 0, n - 1, target);
    if (index != -1) {
        cout << "Element found at index: " << index << endl;
    } else {
        cout << "Element not found in the array." <<endl;
    }
    return 0;
}
~~~

### 我们学习的二分查找算法是针对一维有序序列的，现假设有一个矩阵，其每一行每一列分别是从左到右、从上到下有序的，请为此矩阵编写元素查找算法，并分析你的算法性能。

#### 思路：

> 二分查找的精髓在于找到**mid**,对于这个特殊的矩阵，其实每个数都可以作为mid，主要看其mid的趋势如何：每一个数左边都小于，下边都大于。那么**二分**的雏形就出来了，通过与mid比较大小，逐步缩小逼近target。
>
> 根据mid的特性（左边小于，下边大于），自然而然，右上角便是起始位置，从此开始二分。

对于这样一个按行和列有序排列的矩阵，可以采用一种称为"行列逼近法"的查找算法。其基本思想是从矩阵的右上角（或左下角）开始查找，通过比较目标值与当前位置元素的大小关系，逐步逼近目标值，直至找到目标值或者确定不存在。

以下是该算法的实现步骤：

1. 从矩阵的右上角开始，设定初始行号为0，列号为列数减1。
2. 比较目标值与当前位置的元素大小关系：
   - 如果目标值等于当前位置元素，则返回找到的位置；
   - 如果目标值小于当前位置元素，则说明目标值应该在当前位置的左侧，列号减1；
   - 如果目标值大于当前位置元素，则说明目标值应该在当前位置的下方，行号加1。
3. 重复步骤2，直至找到目标值或者行号超出矩阵范围为止。

#### 代码：

~~~c++
#include <iostream>
#include <vector>

using namespace std;

bool searchInMatrix(vector<vector<int>>& matrix, int target) {
    if(matrix.empty() || matrix[0].empty()) {
        return false;
    }
    
    int rows = matrix.size();
    int cols = matrix[0].size();
    
    // 从矩阵的右上角开始
    int row = 0;
    int col = cols - 1;
    
    while(row < rows && col >= 0) {
        if(matrix[row][col] == target) {
            return true;
        } else if(matrix[row][col] > target) {
            col--;
        } else {
            row++;
        }
    }
    
    return false;
}

int main() {
    vector<vector<int>> matrix = {
        {1, 4, 7, 11},
        {2, 5, 8, 12},
        {3, 6, 9, 16},
        {10, 13, 14, 17}
    };
    int target = 5;
    
    if(searchInMatrix(matrix, target)) {
        cout << "Target found in the matrix." << endl;
    } else {
        cout << "Target not found in the matrix." << endl;
    }
    
    return 0;
}

~~~

