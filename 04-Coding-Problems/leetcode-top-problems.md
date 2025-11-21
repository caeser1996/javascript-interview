# LeetCode Top JavaScript Problems

## Table of Contents
- [Arrays & Strings](#arrays--strings)
- [Two Pointers](#two-pointers)
- [Sliding Window](#sliding-window)
- [Hash Tables](#hash-tables)
- [Recursion & Backtracking](#recursion--backtracking)
- [Dynamic Programming](#dynamic-programming)
- [Trees & Graphs](#trees--graphs)
- [Common Patterns Summary](#common-patterns-summary)

## Arrays & Strings

### 1. Two Sum

**Problem**: Given an array of integers, return indices of two numbers that add up to target.

```javascript
/**
 * @param {number[]} nums
 * @param {number} target
 * @return {number[]}
 */
function twoSum(nums, target) {
  const map = new Map();

  for (let i = 0; i < nums.length; i++) {
    const complement = target - nums[i];

    if (map.has(complement)) {
      return [map.get(complement), i];
    }

    map.set(nums[i], i);
  }

  return [];
}

// Time: O(n), Space: O(n)

console.log(twoSum([2, 7, 11, 15], 9)); // [0, 1]
console.log(twoSum([3, 2, 4], 6));      // [1, 2]
```

### 2. Valid Anagram

**Problem**: Check if two strings are anagrams.

```javascript
/**
 * @param {string} s
 * @param {string} t
 * @return {boolean}
 */
function isAnagram(s, t) {
  if (s.length !== t.length) return false;

  const count = {};

  for (const char of s) {
    count[char] = (count[char] || 0) + 1;
  }

  for (const char of t) {
    if (!count[char]) return false;
    count[char]--;
  }

  return true;
}

// Alternative: Sort and compare
function isAnagram2(s, t) {
  return s.split('').sort().join('') === t.split('').sort().join('');
}

console.log(isAnagram("anagram", "nagaram")); // true
console.log(isAnagram("rat", "car"));         // false
```

### 3. Contains Duplicate

**Problem**: Check if array contains duplicates.

```javascript
/**
 * @param {number[]} nums
 * @return {boolean}
 */
function containsDuplicate(nums) {
  return new Set(nums).size !== nums.length;
}

// Alternative: Using hash set
function containsDuplicate2(nums) {
  const seen = new Set();

  for (const num of nums) {
    if (seen.has(num)) return true;
    seen.add(num);
  }

  return false;
}

console.log(containsDuplicate([1, 2, 3, 1]));    // true
console.log(containsDuplicate([1, 2, 3, 4]));    // false
```

### 4. Product of Array Except Self

**Problem**: Return array where each element is product of all elements except itself.

```javascript
/**
 * @param {number[]} nums
 * @return {number[]}
 */
function productExceptSelf(nums) {
  const n = nums.length;
  const result = new Array(n).fill(1);

  // Calculate left products
  let left = 1;
  for (let i = 0; i < n; i++) {
    result[i] = left;
    left *= nums[i];
  }

  // Calculate right products and multiply
  let right = 1;
  for (let i = n - 1; i >= 0; i--) {
    result[i] *= right;
    right *= nums[i];
  }

  return result;
}

// Time: O(n), Space: O(1) (output array doesn't count)

console.log(productExceptSelf([1, 2, 3, 4]));
// [24, 12, 8, 6]
```

### 5. Maximum Subarray (Kadane's Algorithm)

**Problem**: Find contiguous subarray with largest sum.

```javascript
/**
 * @param {number[]} nums
 * @return {number}
 */
function maxSubArray(nums) {
  let maxSum = nums[0];
  let currentSum = nums[0];

  for (let i = 1; i < nums.length; i++) {
    currentSum = Math.max(nums[i], currentSum + nums[i]);
    maxSum = Math.max(maxSum, currentSum);
  }

  return maxSum;
}

// Time: O(n), Space: O(1)

console.log(maxSubArray([-2, 1, -3, 4, -1, 2, 1, -5, 4])); // 6
// Subarray: [4, -1, 2, 1]
```

## Two Pointers

### 6. Valid Palindrome

**Problem**: Check if string is palindrome.

```javascript
/**
 * @param {string} s
 * @return {boolean}
 */
function isPalindrome(s) {
  // Clean string: lowercase, remove non-alphanumeric
  const cleaned = s.toLowerCase().replace(/[^a-z0-9]/g, '');

  let left = 0;
  let right = cleaned.length - 1;

  while (left < right) {
    if (cleaned[left] !== cleaned[right]) {
      return false;
    }
    left++;
    right--;
  }

  return true;
}

console.log(isPalindrome("A man, a plan, a canal: Panama")); // true
console.log(isPalindrome("race a car"));                      // false
```

### 7. Container With Most Water

**Problem**: Find two lines that form container with maximum water.

```javascript
/**
 * @param {number[]} height
 * @return {number}
 */
function maxArea(height) {
  let left = 0;
  let right = height.length - 1;
  let maxArea = 0;

  while (left < right) {
    const width = right - left;
    const minHeight = Math.min(height[left], height[right]);
    const area = width * minHeight;

    maxArea = Math.max(maxArea, area);

    // Move pointer with smaller height
    if (height[left] < height[right]) {
      left++;
    } else {
      right--;
    }
  }

  return maxArea;
}

// Time: O(n), Space: O(1)

console.log(maxArea([1, 8, 6, 2, 5, 4, 8, 3, 7])); // 49
```

### 8. 3Sum

**Problem**: Find all unique triplets that sum to zero.

```javascript
/**
 * @param {number[]} nums
 * @return {number[][]}
 */
function threeSum(nums) {
  nums.sort((a, b) => a - b);
  const result = [];

  for (let i = 0; i < nums.length - 2; i++) {
    // Skip duplicates
    if (i > 0 && nums[i] === nums[i - 1]) continue;

    let left = i + 1;
    let right = nums.length - 1;

    while (left < right) {
      const sum = nums[i] + nums[left] + nums[right];

      if (sum === 0) {
        result.push([nums[i], nums[left], nums[right]]);

        // Skip duplicates
        while (left < right && nums[left] === nums[left + 1]) left++;
        while (left < right && nums[right] === nums[right - 1]) right--;

        left++;
        right--;
      } else if (sum < 0) {
        left++;
      } else {
        right--;
      }
    }
  }

  return result;
}

console.log(threeSum([-1, 0, 1, 2, -1, -4]));
// [[-1, -1, 2], [-1, 0, 1]]
```

## Sliding Window

### 9. Longest Substring Without Repeating Characters

**Problem**: Find length of longest substring without repeating characters.

```javascript
/**
 * @param {string} s
 * @return {number}
 */
function lengthOfLongestSubstring(s) {
  const charSet = new Set();
  let left = 0;
  let maxLength = 0;

  for (let right = 0; right < s.length; right++) {
    // Shrink window while duplicate exists
    while (charSet.has(s[right])) {
      charSet.delete(s[left]);
      left++;
    }

    charSet.add(s[right]);
    maxLength = Math.max(maxLength, right - left + 1);
  }

  return maxLength;
}

// Time: O(n), Space: O(min(n, m)) where m is charset size

console.log(lengthOfLongestSubstring("abcabcbb")); // 3 ("abc")
console.log(lengthOfLongestSubstring("bbbbb"));    // 1 ("b")
console.log(lengthOfLongestSubstring("pwwkew"));   // 3 ("wke")
```

### 10. Minimum Window Substring

**Problem**: Find minimum window in S that contains all characters of T.

```javascript
/**
 * @param {string} s
 * @param {string} t
 * @return {string}
 */
function minWindow(s, t) {
  if (s.length < t.length) return "";

  const need = new Map();
  const window = new Map();

  // Count characters in t
  for (const char of t) {
    need.set(char, (need.get(char) || 0) + 1);
  }

  let left = 0;
  let right = 0;
  let valid = 0;
  let minLen = Infinity;
  let start = 0;

  while (right < s.length) {
    const c = s[right];
    right++;

    // Update window
    if (need.has(c)) {
      window.set(c, (window.get(c) || 0) + 1);
      if (window.get(c) === need.get(c)) {
        valid++;
      }
    }

    // Shrink window
    while (valid === need.size) {
      // Update result
      if (right - left < minLen) {
        minLen = right - left;
        start = left;
      }

      const d = s[left];
      left++;

      if (need.has(d)) {
        if (window.get(d) === need.get(d)) {
          valid--;
        }
        window.set(d, window.get(d) - 1);
      }
    }
  }

  return minLen === Infinity ? "" : s.substring(start, start + minLen);
}

console.log(minWindow("ADOBECODEBANC", "ABC")); // "BANC"
```

## Hash Tables

### 11. Group Anagrams

**Problem**: Group strings that are anagrams together.

```javascript
/**
 * @param {string[]} strs
 * @return {string[][]}
 */
function groupAnagrams(strs) {
  const map = new Map();

  for (const str of strs) {
    // Use sorted string as key
    const key = str.split('').sort().join('');

    if (!map.has(key)) {
      map.set(key, []);
    }

    map.get(key).push(str);
  }

  return Array.from(map.values());
}

console.log(groupAnagrams(["eat", "tea", "tan", "ate", "nat", "bat"]));
// [["eat", "tea", "ate"], ["tan", "nat"], ["bat"]]
```

### 12. Top K Frequent Elements

**Problem**: Find k most frequent elements.

```javascript
/**
 * @param {number[]} nums
 * @param {number} k
 * @return {number[]}
 */
function topKFrequent(nums, k) {
  // Count frequencies
  const freq = new Map();
  for (const num of nums) {
    freq.set(num, (freq.get(num) || 0) + 1);
  }

  // Bucket sort by frequency
  const buckets = Array(nums.length + 1).fill(null).map(() => []);

  for (const [num, count] of freq) {
    buckets[count].push(num);
  }

  // Collect top k
  const result = [];
  for (let i = buckets.length - 1; i >= 0 && result.length < k; i--) {
    result.push(...buckets[i]);
  }

  return result.slice(0, k);
}

// Time: O(n), Space: O(n)

console.log(topKFrequent([1, 1, 1, 2, 2, 3], 2)); // [1, 2]
```

## Recursion & Backtracking

### 13. Generate Parentheses

**Problem**: Generate all combinations of well-formed parentheses.

```javascript
/**
 * @param {number} n
 * @return {string[]}
 */
function generateParenthesis(n) {
  const result = [];

  function backtrack(current, open, close) {
    if (current.length === 2 * n) {
      result.push(current);
      return;
    }

    if (open < n) {
      backtrack(current + '(', open + 1, close);
    }

    if (close < open) {
      backtrack(current + ')', open, close + 1);
    }
  }

  backtrack('', 0, 0);
  return result;
}

console.log(generateParenthesis(3));
// ["((()))", "(()())", "(())()", "()(())", "()()()"]
```

### 14. Permutations

**Problem**: Generate all permutations of an array.

```javascript
/**
 * @param {number[]} nums
 * @return {number[][]}
 */
function permute(nums) {
  const result = [];

  function backtrack(current, remaining) {
    if (remaining.length === 0) {
      result.push([...current]);
      return;
    }

    for (let i = 0; i < remaining.length; i++) {
      current.push(remaining[i]);
      const newRemaining = [...remaining.slice(0, i), ...remaining.slice(i + 1)];
      backtrack(current, newRemaining);
      current.pop();
    }
  }

  backtrack([], nums);
  return result;
}

console.log(permute([1, 2, 3]));
// [[1,2,3], [1,3,2], [2,1,3], [2,3,1], [3,1,2], [3,2,1]]
```

## Dynamic Programming

### 15. Climbing Stairs

**Problem**: How many distinct ways to climb n stairs (1 or 2 steps at a time)?

```javascript
/**
 * @param {number} n
 * @return {number}
 */
function climbStairs(n) {
  if (n <= 2) return n;

  let prev2 = 1;
  let prev1 = 2;

  for (let i = 3; i <= n; i++) {
    const current = prev1 + prev2;
    prev2 = prev1;
    prev1 = current;
  }

  return prev1;
}

// Time: O(n), Space: O(1)

console.log(climbStairs(5)); // 8
```

### 16. Coin Change

**Problem**: Minimum coins needed to make amount.

```javascript
/**
 * @param {number[]} coins
 * @param {number} amount
 * @return {number}
 */
function coinChange(coins, amount) {
  const dp = new Array(amount + 1).fill(Infinity);
  dp[0] = 0;

  for (let i = 1; i <= amount; i++) {
    for (const coin of coins) {
      if (coin <= i) {
        dp[i] = Math.min(dp[i], dp[i - coin] + 1);
      }
    }
  }

  return dp[amount] === Infinity ? -1 : dp[amount];
}

console.log(coinChange([1, 2, 5], 11)); // 3 (5 + 5 + 1)
console.log(coinChange([2], 3));        // -1
```

### 17. Longest Increasing Subsequence

**Problem**: Length of longest increasing subsequence.

```javascript
/**
 * @param {number[]} nums
 * @return {number}
 */
function lengthOfLIS(nums) {
  const dp = new Array(nums.length).fill(1);

  for (let i = 1; i < nums.length; i++) {
    for (let j = 0; j < i; j++) {
      if (nums[i] > nums[j]) {
        dp[i] = Math.max(dp[i], dp[j] + 1);
      }
    }
  }

  return Math.max(...dp);
}

// Time: O(n²), Space: O(n)

console.log(lengthOfLIS([10, 9, 2, 5, 3, 7, 101, 18])); // 4
```

## Trees & Graphs

### 18. Binary Tree Inorder Traversal

**Problem**: Inorder traversal of binary tree.

```javascript
class TreeNode {
  constructor(val, left = null, right = null) {
    this.val = val;
    this.left = left;
    this.right = right;
  }
}

/**
 * @param {TreeNode} root
 * @return {number[]}
 */
function inorderTraversal(root) {
  const result = [];

  function traverse(node) {
    if (!node) return;

    traverse(node.left);
    result.push(node.val);
    traverse(node.right);
  }

  traverse(root);
  return result;
}

// Iterative version
function inorderTraversalIterative(root) {
  const result = [];
  const stack = [];
  let current = root;

  while (current || stack.length > 0) {
    // Go to leftmost node
    while (current) {
      stack.push(current);
      current = current.left;
    }

    // Process node
    current = stack.pop();
    result.push(current.val);

    // Visit right subtree
    current = current.right;
  }

  return result;
}
```

### 19. Maximum Depth of Binary Tree

**Problem**: Find maximum depth of binary tree.

```javascript
/**
 * @param {TreeNode} root
 * @return {number}
 */
function maxDepth(root) {
  if (!root) return 0;

  return 1 + Math.max(maxDepth(root.left), maxDepth(root.right));
}

// Iterative BFS version
function maxDepthBFS(root) {
  if (!root) return 0;

  const queue = [root];
  let depth = 0;

  while (queue.length > 0) {
    const levelSize = queue.length;

    for (let i = 0; i < levelSize; i++) {
      const node = queue.shift();

      if (node.left) queue.push(node.left);
      if (node.right) queue.push(node.right);
    }

    depth++;
  }

  return depth;
}
```

### 20. Validate Binary Search Tree

**Problem**: Check if tree is valid BST.

```javascript
/**
 * @param {TreeNode} root
 * @return {boolean}
 */
function isValidBST(root, min = -Infinity, max = Infinity) {
  if (!root) return true;

  if (root.val <= min || root.val >= max) {
    return false;
  }

  return isValidBST(root.left, min, root.val) &&
         isValidBST(root.right, root.val, max);
}

// Inorder traversal approach
function isValidBST2(root) {
  const values = [];

  function inorder(node) {
    if (!node) return;

    inorder(node.left);
    values.push(node.val);
    inorder(node.right);
  }

  inorder(root);

  // Check if sorted
  for (let i = 1; i < values.length; i++) {
    if (values[i] <= values[i - 1]) {
      return false;
    }
  }

  return true;
}
```

## Common Patterns Summary

### 1. Two Pointers
- Valid palindrome
- Container with most water
- 3Sum
- Remove duplicates

### 2. Sliding Window
- Longest substring without repeating chars
- Minimum window substring
- Max consecutive ones

### 3. Fast & Slow Pointers
- Linked list cycle
- Find middle of linked list
- Happy number

### 4. Hash Maps
- Two sum
- Group anagrams
- Top K frequent elements

### 5. Recursion & Backtracking
- Generate parentheses
- Permutations
- Subsets

### 6. Dynamic Programming
- Climbing stairs
- Coin change
- Longest increasing subsequence
- House robber

### 7. Binary Search
- Search in rotated sorted array
- Find minimum in rotated sorted array
- Search a 2D matrix

### 8. Trees (DFS/BFS)
- Tree traversals
- Maximum depth
- Validate BST
- Lowest common ancestor

### 9. Graphs
- Number of islands
- Clone graph
- Course schedule (topological sort)

### 10. Greedy
- Jump game
- Gas station
- Meeting rooms

## Practice Strategy

1. **Master patterns first**: Understand the core patterns
2. **Practice daily**: Solve 2-3 problems per day
3. **Time yourself**: Aim for 20-30 minutes per medium problem
4. **Explain out loud**: Practice explaining your approach
5. **Review solutions**: Learn from optimal solutions
6. **Track progress**: Keep a log of solved problems

---

**See also**: [Coding Patterns](./patterns/) for detailed explanations
