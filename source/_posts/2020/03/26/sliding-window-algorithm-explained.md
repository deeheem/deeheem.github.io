---
title: Sliding Window Algorithm explained
date: 2020-03-26 12:55:22
category:
- Algorithms
tags:
- "#sliding_window"
---

<div style="clear: both;">
  <div style="float: left; margin-right: 1em;">
    <img src="image.gif" width="250"/>
  </div>
  <div>
    In this blog, I am going to consolidate all the problems related to Sliding Window Algorithm which I have come across, from beginner level to advanced level. These are some notes which I found from my placement days, I hope they will help in advancing your knowledge from a basic to an advanced level. For an introductory article on this algorithm, you can go through <a href="https://www.geeksforgeeks.org/window-sliding-technique/">this</a>.
  </div>
</div>

<p>

- [Longest Substring Without Repeating Characters](#Longest-Substring-Without-Repeating-Characters)
- [Longest Substring with At Most Two Distinct Characters](#Longest-Substring-with-At-Most-Two-Distinct-Characters)
- [Longest Substring with At Most K Distinct Characters](#Longest-Substring-with-At-Most-K-Distinct-Characters)
- [Longest Substring with exact K distinct Characters](#Longest-Substring-with-exact-K-distinct-Characters)


## Longest Substring Without Repeating Characters

### Problem

Given a string, find the length of the longest substring without repeating characters.

### Example

Input: “abcabcbb”
Output: 3 
Explanation: The answer is “abc”, with the length of 3.

### Approach

Use HashSet to store the characters in the current window [i, j). Then we slide the index j to the right. If it is not in the HashSet, we slide j further. Doing so until s[j] is already in the HashSet. At this point, we found the maximum size of substrings without duplicate characters start with index i. If we do this for all i, we get our answer.

{% codeblock "Dry Run : Longest Substring Without Repeating Characters" lang:text %}
abcabcbb
i
j 
set = [a], ans = max(0, 0-0+1) = 1
abcabcbb
ij 
set = [a, b], ans = max(1, 1-0+1) = 2
abcabcbb
i j 
set = [a, b, c], ans = max(2, 2-0+1) = 3
abcabcbb
i  j 
repeating character found at j, so i++ and remove char at i from set
set = [b, c]
abcabcbb
 i j 
set = [b, c, a], ans = max(3, 3-1+1) = 3
abcabcbb
 i  j 
repeating character found at j, so i++ and remove char at i from set
set = [c, a]
abcabcbb
  i j 
set = [c, a, b], ans = max(3, 4-2+1) = 3
abcabcbb
  i  j
repeating character found at j, so i++ and remove char at i from set
set = [a, b]
abcabcbb
   i j 
set = [a, b, c], ans = max(3, 5-3+1) = 3
abcabcbb
   i  j 
repeating character found at j, so i++ and remove char at i from set
set = [b, c]
abcabcbb
     ij
repeating character found at j, so i++ and remove char at i from set
set = [c]
abcabcbb
     ij
set = [c, b], ans = max(3, 6-5+1) = 3
abcabcbb
     i j
repeating character found at j, so i++ and remove char at i from set
set = [b]
abcabcbb
      ij
repeating character found at j, so i++ and remove char at i from set
set = []
{% endcodeblock %}

{% codeblock "Code : Longest Substring Without Repeating Characters" lang:java https://gist.github.com/deeheem/776ab95a6f7aca5a641dac019f555f0a gist.java %}
  public int lengthOfLongestSubString(String s) {
    int n = s.length();
    Set<Character> set = new HashSet<>();
    int ans = 0, i = 0, j = 0;
    while (i < n && j < n) {
      if (!set.contains(s.charAt(j))) {
        set.add(s.charAt(j));
        j++;
        ans = Math.max(ans, j - i);
      } else {
        set.remove(s.charAt(i));
        i++;
      }
    }
    return ans;
  }
{% endcodeblock %}

## Longest Substring with At Most Two Distinct Characters

### Problem

Given a string S, find the length of the longest substring T that contains at most two distinct characters.

### Example

Input: “aabcd”
Output: 3 
Explanation: The answer is “aab”, with the length of 3.

### Approach

We use a sliding window that always satisfies the condition where there are always at most two distinct characters in it. When we add a new character that breaks this condition, we move the starting pointer of our string.

{% codeblock "Dry Run : Longest Substring with At Most Two Distinct Characters" lang:text %}
aabcd
i
j
k = 0 (character 'a')
ans = max(0, 0-0+1) = 1
aabcd
ij
k = 1 (characte 'a')
ans = max(1, 1-0+1) = 2
aabcd
i j
k = 2 (character 'b')
ans = max(2, 2-0+1) = 3
aabcd
i  j
more than two distict characters found, i++
aabcd
 i j
more than two distict characters found, i++
aabcd
  ij
ans = max(3, 3-2+1) = 3
aabcd
  i j
more than two distict characters found, i++
aabcd
   ij
ans = max(3, 4-3+1) = 3
{% endcodeblock %}

{% codeblock "Code : Longest Substring with At Most Two Distinct Characters" lang:java https://gist.github.com/deeheem/098651e2f4bcee6da355b2b28e498be8 gist.java %}
    public int lengthOfLongestSubStringTwoDistinct(String s) {
        int n = s.length();
        int i = 0, j = -1, ans = 0;

        for (int k = 1; k < s.length(); k++) {
            if (s.charAt(k) == s.charAt(k - 1)) {
                continue;
            }
            if (j >= 0 && s.charAt(j) != s.charAt(k)) {
                ans = Math.max(ans, k - i);
                i = j + 1;
                j = k - 1;
            }
        }
        return Math.max(ans, n - i);
    }
{% endcodeblock %}

## Longest Substring with At Most K Distinct Characters

### Problem

An extension to the previous problem, but instead of 2 now you need to have k distinct characters in the substring.

### Example

Input: s = “eceba”, k = 2
Output: 3 
Explanation: The answer is “ece”, with the length of 3.

### Approach

This is similar to solving the previous problem with at most 2 distinct characters, but the only difference now is that we need to track the number of distinct characters as well, for which we use the help of a map.

{% codeblock "Code : Longest Substring with At Most K Distinct Characters" lang:java https://gist.github.com/deeheem/b7d83b141c85bc3c9a3111df9faed98d gist.java %}
    public int lengthOfLongestSubStringKDistinct(String s) {
        int n = s.length();
        int[] count = new int[256];
        int i = 0, numDistinct = 0, ans = 0;

        for (int j = 1; j < s.length(); j++) {
            if (count[s.charAt(j)] == 0) {
                numDistinct++;
            }
            count[s.charAt(j)]++;
            while (numDistinct > 2) {
                count[s.charAt(i)]--;
                if (count[s.charAt(i)] == 0) {
                    numDistinct--;
                }
                i++;
            }
            ans = Math.max(ans, j - i + 1);
        }
        return ans;
    }
{% endcodeblock %}

## Longest Substring with exact K distinct Characters

### Problem

Given an array A of positive integers, find the number of subarrays with exactly K number of distinct characters.

### Example

Input: A = [1,2,1,2,3], K = 2
Output: 7
Explanation: Subarrays formed with exactly 2 different integers: [1,2], [2,1], [1,2], [2,3], [1,2,1], [2,1,2], [1,2,1,2].

### Approach 1 (Smart Work)

If we are aware of how to find subarrays with “at most k different characters”, then we can extend the above algorithm to find the number of subarrays with “exactly k different characters” using the equation: 

<div align="center">
  <code>exactly(K) = atMost(K) — atMost(K-1)</code>
</div>

{% codeblock "Code : Longest Substring with exact K distinct Characters" lang:java https://gist.github.com/deeheem/0e8491c55cb91207e5d478e4a8000c68 gist.java %}
  public int lengthOfLongestSubStringAtMostKDistinct(String s, int k) {
    int n = s.length();
    int[] count = new int[256];
    int i = 0, numDistinct = 0, ans = 0;

    for (int j = 1; j < s.length(); j++) {
      if (count[s.charAt(j)] == 0) {
        numDistinct++;
      }
      count[s.charAt(j)]++;
      while (numDistinct > k) {
        count[s.charAt(i)]--;
        if (count[s.charAt(i)] == 0) {
          numDistinct--;
        }
        i++;
      }
      ans = Math.max(ans, j - i + 1);
    }
    return ans;
  }

  public int lengthOfLongestSubStringKDistinctIntegers(String s, int k) {
    return lengthOfLongestSubStringAtMostKDistinct(s, k)
        - lengthOfLongestSubStringAtMostKDistinct(s, k - 1);
{% endcodeblock %}

### Approach 2 (Hard Work)

[Visit this link to learn more about this approach.](https://leetcode.com/problems/subarrays-with-k-different-integers/discuss/235235/C%2B%2BJava-with-picture-prefixed-sliding-window)
   