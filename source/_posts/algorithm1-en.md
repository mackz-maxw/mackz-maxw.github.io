---
title: algorithm1-en
date: 2025-01-21 20:36:03
mathjax: true
categories: leetcode
lang: en
---

I've been working on LeetCode problems for almost a week now, and I feel like my progress with algorithms is pretty slow—managing just one or two problems a day seems like my limit.

I'm focusing on problems from the "Beginner Algorithms" section on LeetCode. Sometimes, I feel my memory is terrible; I realized I had encountered some of these problems before in an introductory algorithms book but couldn't recall the solutions. I ended up solving them with the most inefficient methods...

I can foresee that learning techniques like two-pointers will take some getting used to. It's tough! But I'll take it one step at a time.

(All problems below are from LeetCode's official website.)

## 2/25-3/1

### LC26. Remove Duplicates from Sorted Array

You are given an array `nums` sorted in non-decreasing order. Remove the duplicates in-place such that each unique element appears only once. The relative order of the elements should be kept the same. After removing the duplicates, return the new length of the array.

Since some languages cannot change the size of the array, the final result must be placed in the first part of the array `nums`. More formally, if there are `k` elements after removing the duplicates, then the first `k` elements of `nums` should hold the final result.

Do not allocate extra space for another array. You must do this by modifying the input array in-place with $O(1)$ extra memory.

$$
\begin{aligned}
&\text{class Solution \{} \\
&\quad \text{public:} \\
&\quad \quad \text{int removeDuplicates(vector<int>\& nums) \{} \\
&\quad \quad \quad \text{int a = 0;} \\
&\quad \quad \quad \text{int b = nums.size();} \\
&\quad \quad \quad \text{if (b <= 1) return b;} \\
&\quad \quad \quad \text{for (int i = 0; i < (b - 1); i++) \{} \\
&\quad \quad \quad \quad \text{if (nums[i] != nums[i + 1]) \{} \\
&\quad \quad \quad \quad \quad \text{nums[a] = nums[i];} \\
&\quad \quad \quad \quad \quad \text{a++;} \\
&\quad \quad \quad \quad \text{\}} \\
&\quad \quad \quad \text{\}} \\
&\quad \quad \quad \text{nums[a] = nums[b - 1];} \\
&\quad \quad \quad \text{return a + 1;} \\
&\quad \quad \text{\}} \\
&\text{\};}
\end{aligned}
$$

Here, I learned (again) about the two-pointer technique and refreshed my long-unused C++ knowledge. I chose C++ mainly because many of the algorithm and OpenCV books I previously studied use C++. Although I've heard C++ interviews focus heavily on language features, it feels more suitable than Python for understanding internal mechanisms. Python seems to hide a lot of details, which might not be ideal for my coding style later.

This problem was manageable, but for problems involving in-place modifications, it's essential to use `vector<int>& nums` to pass by reference.

While solving this, I realized I had forgotten basic functions like `.size()`, `memset()`, and `.length()`. Sigh.

---

### LC122. Best Time to Buy and Sell Stock II

You are given an array `prices` where `prices[i]` is the price of a given stock on the `i`th day.

On each day, you may decide to buy and/or sell the stock. You can hold at most one share of the stock at any time. However, you can buy it and then sell it on the same day. Return the maximum profit you can achieve.

$$
\begin{aligned}
&\text{class Solution \{} \\
&\quad \text{public:} \\
&\quad \quad \text{int maxProfit(vector<int>\& prices) \{} \\
&\quad \quad \quad \text{int day = 0;} \\
&\quad \quad \quad \text{int count = 0;} \\
&\quad \quad \quad \text{bool st = false;} \\
&\quad \quad \quad \text{if (prices.size() == 1) return 0;} \\
&\quad \quad \quad \text{for (int i = 0; i < prices.size() - 1; i++) \{} \\
&\quad \quad \quad \quad \text{if (prices[i] < prices[i + 1] && st == false) \{} \\
&\quad \quad \quad \quad \quad \text{day = i; st = true;} \\
&\quad \quad \quad \quad \text{\}} \\
&\quad \quad \quad \quad \text{if (prices[i] > prices[i + 1] && st == true) \{} \\
&\quad \quad \quad \quad \quad \text{count += prices[i] - prices[day]; st = false;} \\
&\quad \quad \quad \quad \text{\}} \\
&\quad \quad \quad \text{\}} \\
&\quad \quad \quad \text{return count;} \\
&\quad \quad \text{\}} \\
&\text{\};}
\end{aligned}
$$

The above solution attempts to track local minima and maxima but led to overly complex `if` conditions. A simpler approach is summing positive differences between consecutive days:

$$
\begin{aligned}
&\text{class Solution \{} \\
&\quad \text{public:} \\
&\quad \quad \text{int maxProfit(vector<int>\& prices) \{} \\
&\quad \quad \quad \text{int pf = 0;} \\
&\quad \quad \quad \text{for (int i = 0; i < prices.size() - 1; i++) \{} \\
&\quad \quad \quad \quad \text{if (prices[i] < prices[i + 1]) \{} \\
&\quad \quad \quad \quad \quad \text{pf += prices[i + 1] - prices[i];} \\
&\quad \quad \quad \quad \text{\}} \\
&\quad \quad \quad \text{\}} \\
&\quad \quad \quad \text{return pf;} \\
&\quad \quad \text{\}} \\
&\text{\};}
\end{aligned}
$$

This solution is concise and easier to understand, summing up profits whenever prices go up.

---

### LC48. Rotate Image

You are given an $n \times n$ 2D matrix representing an image. Rotate the image by 90 degrees clockwise.

$$
\begin{aligned}
&\text{class Solution \{} \\
&\quad \text{public:} \\
&\quad \quad \text{void rotate(vector<vector<int>>\& matrix) \{} \\
&\quad \quad \quad \text{int n = matrix.size();} \\
&\quad \quad \quad \text{for (int i = 0; i < n; i++) \{} \\
&\quad \quad \quad \quad \text{for (int j = i; j < n; j++) \{} \\
&\quad \quad \quad \quad \quad \text{swap(matrix[i][j], matrix[j][i]);} \\
&\quad \quad \quad \quad \text{\}} \\
&\quad \quad \quad \text{\}} \\
&\quad \quad \quad \text{for (auto\& row : matrix) \{ reverse(row.begin(), row.end()); \}} \\
&\quad \quad \text{\}} \\
&\text{\};}
\end{aligned}
$$

First, transpose the matrix, then reverse each row to achieve the 90-degree rotation.

After solving the problem using the approach of transposing the matrix followed by reversing the rows, I thought of an alternative way: reversing the rows first and then transposing the matrix. 

This approach also works effectively and provides the same result. Here's how it looks:

$$
\begin{aligned}
&\text{class Solution \{} \\
&\quad \text{public:} \\
&\quad \quad \text{void rotate(vector<vector<int>>\& matrix) \{} \\
&\quad \quad \quad \text{reverse(matrix.begin(), matrix.end());} \\
&\quad \quad \quad \text{for (int i = 0; i < matrix.size(); i++) \{} \\
&\quad \quad \quad \quad \text{for (int j = i; j < matrix[i].size(); j++) \{} \\
&\quad \quad \quad \quad \quad \text{swap(matrix[i][j], matrix[j][i]);} \\
&\quad \quad \quad \quad \text{\}} \\
&\quad \quad \quad \text{\}} \\
&\quad \quad \text{\}} \\
&\text{\};}
\end{aligned}
$$

This version simplifies the triangle iteration by starting the inner loop at `j = i`. It reduces the code's complexity while maintaining clarity. However, I noticed that the memory usage was slightly higher compared to my earlier implementation where I explicitly managed the triangular region with an additional variable. To investigate further, I experimented with both versions on LeetCode's submission platform. Interestingly, I found that memory usage sometimes fluctuated slightly for reasons I couldn't fully understand, possibly due to internal optimizations in the runtime environment.

Ultimately, both methods provide the correct result, and the choice depends on your preference for readability versus strict memory management.

