---
layout: post
title:  "[leetcode] reverse-vowels-of-a-string"
author: "lastgleam"
---
URL : [https://leetcode.com/problems/reverse-vowels-of-a-string/](https://leetcode.com/problems/reverse-vowels-of-a-string/)

양쪽 끝에서 동시에 탐색하는 투 포인터 문제였다.
일단 처음 작성한 답안은 다음과 같다.

> Runtime: 3260 ms, faster than 5.13% of JavaScript online submissions for Reverse Vowels of a String.
Memory Usage: 43.9 MB, less than 31.03% of JavaScript online submissions for Reverse Vowels of a String.

```javascript
/**
 * @param {string} s
 * @return {string}
 */
var reverseVowels = function(s) {
    const indexes = [];
    let mid = 0;
    for(let i = 0; i < s.length; i++) {
        if (s.charAt(i).match(/(a|e|i|o|u)/i)) {
            indexes.push(i);
        } 
    }
    if (indexes % 2 !== 0) {
        mid = (s.length - 1) / 2;
    } else {
        mid = (s.length /2 - 1);
    }
    let result = "";
    let count = 0;
    for(i = 0; i < s.length; i++) {
        if (indexes.includes(i)) {
            if (count <= mid) {
                result += s.charAt(indexes[indexes.length - 1 - count]);
            } else {
                result += s.charAt(indexes[count - 1 - indexes.length]);
            }
            count++;
        } else {
            result += s.charAt(i);
        }
    }
    return result;
};
```