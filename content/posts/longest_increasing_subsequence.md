+++
title = 'How to find the longest increasing subsequence in an array?'
date = 2025-09-07T12:40:07+08:00
draft = false
+++

## 問題描述

給定一個整數陣列，找到其中最長遞增子序列的長度。子序列是指從原陣列中刪除一些（或不刪除）元素後，剩餘元素保持原有順序的序列。

**例子：**
- 輸入：`[10, 9, 2, 5, 3, 7, 101, 18]`
- 輸出：`4`
- 解釋：最長遞增子序列是 `[2, 3, 7, 18]`，長度為 4

## 方法一：動態規劃

使用動態規劃來解決這個問題。核心思想是：對於每個位置 i，我們需要找到所有在 i 之前且值小於 nums[i] 的元素，然後取其中 dp 值最大的那個，再加上 1。

我們建立一個 dp 陣列，其中 dp[i] 表示以 nums[i] 為結尾的最長遞增子序列的長度。

```c++
int lengthOfLIS(vector<int>& nums) {
    const int n = nums.size();
    vector<int> dp(n);
    // if no nums, length will be zero
    int max_length = 0;

    for (int i = 0; i < n; i++) {
        int prev_length = 0;
        for (int j = 0; j < i; j++) {
            if (nums[j] < nums[i]) {
                prev_length = max(prev_length, dp[j]);
            }
        }
        dp[i] = prev_length + 1;
        max_length = max(max_length, dp[i]);
    }

    return max_length;
}
```

**時間複雜度：** O(n²)  
**空間複雜度：** O(n)

## 方法二：二分搜尋優化

方法一的時間複雜度是 O(n²)，我們可以通過二分搜尋來優化到 O(n log n)。

**核心思想：**
我們維護一個單調遞增的陣列 `tails`，其中 `tails[i]` 表示長度為 i+1 的遞增子序列的最小結尾元素。

**為什麼這樣做是對的？**
對於相同長度的遞增子序列，結尾元素越小，越容易形成更長的子序列。例如：
- 長度為 2 的子序列：`[1, 3]` 比 `[1, 5]` 更好，因為 3 < 5
- 如果後續遇到 4，`[1, 3, 4]` 可以形成長度為 3 的子序列，但 `[1, 5, 4]` 不行

**演算法步驟：**
1. 遍歷每個元素
2. 使用二分搜尋找到第一個大於等於當前元素的位置
3. 如果找到，則替換該位置；如果沒找到，則添加到末尾

```c++
int binary_search(vector<int>& tails, int target) {
    int left = 0, right = tails.size() - 1;
    int result = tails.size(); // 如果沒找到，返回末尾位置

    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (tails[mid] >= target) {
            result = mid;
            right = mid - 1;
        } else {
            left = mid + 1;
        }
    }
    return result;
}

int lengthOfLIS(vector<int>& nums) {
    vector<int> tails;
    
    for (int num : nums) {
        int pos = binary_search(tails, num);
        if (pos == tails.size()) {
            // 當前元素比所有元素都大，添加到末尾
            tails.push_back(num);
        } else {
            // 替換第一個大於等於當前元素的位置
            tails[pos] = num;
        }
    }
    
    return tails.size();
}
```

**時間複雜度：** O(n log n)  
**空間複雜度：** O(n)

### 使用 STL 的簡化版本

我們可以使用 C++ 的 `lower_bound` 來簡化二分搜尋的實作：

```c++
int lengthOfLIS(vector<int>& nums) {
    vector<int> tails;
    
    for (int num : nums) {
        auto it = lower_bound(tails.begin(), tails.end(), num);
        if (it == tails.end()) {
            tails.push_back(num);
        } else {
            *it = num;
        }
    }
    
    return tails.size();
}
```

## 總結

| 方法 | 時間複雜度 | 空間複雜度 | 優點 | 缺點 |
|------|------------|------------|------|------|
| 動態規劃 | O(n²) | O(n) | 想法直觀，容易理解 | 時間複雜度較高 |
| 二分搜尋優化 | O(n log n) | O(n) | 時間複雜度更優 | 理解難度較高 |