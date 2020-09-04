---
title: 前缀和（单维数组）
date: 2020-8-9 8:40:52
tags: [算法基础]
---

## 原理

![alt](https://www.cxyxiaowu.com/wp-content/uploads/2020/07/1594091718-5718cf2307eab26.jpeg)

1. nums，原始数组
2. preSum，前缀和数组，即`preSum[i] = nums[0..i-1]`

## 用途

快速求区间和，nums[i..j] = preSum[j+1] - preSum[i]

> 涉及区间(子数组)的问题，可以优先考虑采用前缀和

## 场景

### 和为K的子数组

[LeetCode 560. Subarray Sum Equals K](https://leetcode.com/problems/subarray-sum-equals-k/)

解法一：暴力双循环遍历求解

解法二：前缀和，（子数组，区间问题，快速求和）

```go
func subarraySum(nums []int, k int) int {
    //多开辟一个占位空间，初始为0
	preSumNums := make([]int, len(nums) +1)
	preSumNums[0] = 0

	//计算前缀和
	for i, v := range nums {
		preSumNums[i+1] = preSumNums[i] + v
	}

	//遍历所有场景，找出满足条件的区间数
	total := 0
	for i := 1; i < len(preSumNums); i++ {
		for j := 0; j < i; j++ {
			if preSumNums[i] - preSumNums[j] == k {
				total++
			}
		}
	}

	return total
}
```

1. 减少暴力双循环过程中对原始数组的多次重复遍历
2. 依然存在对前缀和数组的多次重复遍历

解法三：增加散列表，缓存前缀和特定值出现的个数

```go
func subarraySum(nums []int, k int) int {
	preMap := make(map[int]int)
	preMap[0] = 1

	preSum := 0
	total := 0
	for _, v := range nums {
		preSum = preSum + v

		if v, ok := preMap[preSum-k]; ok {
			total = total + v
		}

		preMap[preSum] += 1
	}

	return total
}
```

1. 由于只要求计数满足条件的个数
2. 缓存满足要求的前缀和元素个数，避免对前缀和数组的多次重复遍历
3. 由于只用到了preSum的最新数据，用临时变量替代临时数组
