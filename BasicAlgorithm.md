---
layout: default
---

# Basic Algorithms

## Sorting

### Merge Sorting

Merge Sorting is an efficient, comparison-based sorting algorithm with O(nlogn) time complexity in either Best, Worst or Average cases. It will take O(2n) = O(n) space: O(n) for the original list and O(n) for extra space. The core idea is dividing and merging. If there are two sorted arrays whose total number of items is n then it will takes O(n) time to merge these two array into a new sorted array. To the original array, we divide it by two until every subset left a single item. In that case, every subset was sorted(just one item). And from bottom to top, each time we merge two nearby arrays. Each time there would be a change in every items' order in original 'big' array. Thus each time it will cost O(n). And since each time we dividing the array by two, we need merge O(logn) times, so totally will cost O(nlogn)

The recurrence would be T(n) = 2T(n/2) + O(n)

Here is an example

![](assets/images/BA_MergeSort.png)

### Pseudocode

```pseudocode
Let A[i] i = 1, 2, ..., n; be the original array

defun MergeSort(A, start, end)
{
	while start <= end do:
		middle = start + (end - start) / 2 
		//Recursive division
		MergeSort(A, start, middle, end)
		MergeSort(A, middle + 1, end)
		//Merge
		Merge(A, start, middle, end)
}

defun Merge(A, start, middle, end)
{
	Let subA1[0,...,P] be a new array that store A[start : middle]
	Let subA2[0,...,Q] be a new array that store A[start + 1 : end]
	Let j = 0; k = 0;

	//Sort the array from start to end position
	for (i = start; i <= end; i++)
	{
		if (subA1[j] < subA2[k])
		{
			A[i] = subA1[j];
			j++;
		}
		else if (subA1[j] >= subA2[k])
		{
			A[i] = subA2[k];
			k++;
		}
	}
}

```

[back](./)