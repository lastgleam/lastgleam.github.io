[---
layout: post
title:  "[leetcode] Add To Array Form Of Integer"
author: "lastgleam"
---
URL : [https://leetcode.com/problems/add-to-array-form-of-integer](https://leetcode.com/problems/add-to-array-form-of-integer)

자리수를 맞추어 덧셈을 유도하는 문제다.
주어지는 매개변수가 배열과 int형이기 때문에
int를 배열로 바꾼 후 길이가 짧은 배열에 0을 unshift하여 넣어주어 길이를 맞출 필요가 있다.
예를 들어, [1,2,0,0]과 34가 주어진 경우, 34를 [3,4]로 변환 후 `unshift()`로 [0,0,3,4]로 만들어준다.

> Runtime: 268 ms, faster than 5.45% of JavaScript online submissions for Add to Array-Form of Integer.
>
> Memory Usage: 42 MB, less than 29.31% of JavaScript online submissions for Add to Array-Form of Integer.


```javascript
/**
 * @param {number[]} A
 * @param {number} K
 * @return {number[]}
 */
var addToArrayForm = function(A, K) {
    K = K.toString().split("");
    let result = [];
    let carry = 0;
    let differ = A.length - K.length;
    if(differ > 0) {
        let count = 0;
        while (count < differ) {
            K.unshift(0);
            count++;
        }
    } else {
        let count = 0;
        while (count < Math.abs(differ)) {
            A.unshift(0);
            count++;
        }
    }
    for (let i = A.length - 1; i >= 0; i--) {
        let a = A[i];
        let k = Number.parseInt(K[i]);
        const now = a + k + carry;
        result.unshift(now % 10);
        carry = Math.floor(now / 10);
    }
    if (carry > 0) {
        result.unshift(carry);
    }
    return result;
};
```
](---
layout: post
title:  "[leetcode] Max Consecutive Ones"
author: "lastgleam"
---
URL : [https://leetcode.com/problems/max-consecutive-ones](https://leetcode.com/problems/max-consecutive-ones)

주어진 배열에서 가장 길게 1이 연속된 값을 리턴하는 문제다.
O(n)으로 풀 수 있는 문제로, 1이 나올때마다 카운트하고 0이 나올때마다 max값과 비교하여 더 크면 max에 카운트된 값을 할당하는 방식으로 해결한다. 

> Runtime: 76 ms, faster than 54.71% of JavaScript online submissions for Max Consecutive Ones.
>
> Memory Usage: 37 MB, less than 56.06% of JavaScript online submissions for Max Consecutive Ones.


```javascript
/**
 * @param {number[]} nums
 * @return {number}
 */
var findMaxConsecutiveOnes = function(nums) {
    let max = 0;
    let temp = 0;
    for(let i = 0; i < nums.length; i++) {
        if(nums[i] === 1) {
            temp += 1;
        } else if (nums[i] === 0) {
            if(temp > max) {
                max = temp;
            }
            temp = 0;
        }
    }
    if(temp > max) {
        max = temp;
    }
    return max;
};
```
)