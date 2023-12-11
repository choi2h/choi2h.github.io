---
layout: post
title: "[코딩테스트] leetcode88.Merge Sorted Array"
author: hwa
categories: CodingTest
tags: Java, CodingTest
---

### 문제
You are given two integer arrays `nums1` and `nums2`, sorted in **non-decreasing order**, and two integers `m` and `n`, representing the number of elements in `nums1` and `nums2` respectively.

**Merge** `nums1` and `nums2` into a single array sorted in **non-decreasing order**.

The final sorted array should not be returned by the function, but instead be _stored inside the array_ `nums1`. To accommodate this, `nums1` has a length of `m + n`, where the first `m` elements denote the elements that should be merged, and the last `n` elements are set to `0` and should be ignored. `nums2` has a length of `n`.  
link : https://leetcode.com/problems/merge-sorted-array/description/  


### 문제 설명
#### 제공되는 값
- 오름차순으로 정렬 된 두 배열`nums1`과 `nums2`
- 각 배열에 값이 들어있는 개수 `m` 과 `n`
- `nums1` 배열의 크기는 m+n으로 생성되어 있다.

#### 요구사항
- 두 배열을 오름차순으로 정렬된 하나의 배열로 만들어라.
- 정렬된 배열은 `nums1` 배열에 저장되어야 한다.


### 문제 풀이
`nums1` 배열의 크기가 m+n으로 설정되어 있는 이유는 새로운 원소(`nums2`의 원소)를 추가할 수 있는 공간을 의미한다. 

`nums1`의 비어있는 공간부터 채워넣기 위해 가장 큰 값부터 정렬하고자 한다.  
각 배열의 가장 큰 값이 있는 인덱스부터 순회하며 비교한다.  
비교한 값 중 큰 값을 `nums1` 의 마지막 인덱스(`m+n-1`)부터 채워 넣는다.

`nums1`은 이미 배열 내에 오름차순 정렬이 되어 있으므로 `nums2`의 원소가 모두 소진될 때까지만 순회하면 된다.

``` java
class Solution {

	public void merge(int[] nums1, int m, int[] nums2, int n) {
		int index1 = m-1; // nums1배열의 가장 큰 값이 존재하는 인덱스
		int index2 = n-1; // nums2배열의 가장 큰 값이 존재하는 인덱스
		
		int index = m+n-1; // nums1배열의 가장 마지막 인덱스
		
		while (index2 >= 0) { //nums2배열의 원소가 존재할 때까지
			// nums1배열은 순회가 완료되고 nums2배열의 원소만 남았을 경우
			if(index1 < 0) { 
				// nums2의 원소들을 이전해준다.
				nums1[index--] = nums2[index2--];
				continue;
			}
			
			// nums2의 원소값이 값과 nums1의 원소값보다 클 경우
			if(nums1[index1] < nums2[index2]) {
				// nums2의 원소값을 nums1의 정렬인덱스에 삽입한다.
				nums1[index--] = nums2[index2--];
			} 
			// nums1의 원소값이 값과 nums2의 원소값보다 클 경우
			else {
				// nums1의 원소값을 nums1의 정렬인덱스에 삽입한다.
				nums1[index--] = nums1[index1--];
			}
		}
	}
}
```

위 방법으로 풀이를 하면 아래와 같은 시간복잡도가 나온다.
- 최선일 때 : `O(m)`
- 최악일 때 : `O(n+m)`
