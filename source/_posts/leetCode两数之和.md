---
title: leetCode两数之和
date: 2019-04-09 17:33:44
tags:
---
```javascript
/**
 * @param {number[]} nums
 * @param {number} target
 * @return {number[]}
 */
var twoSum = function(nums, target) {
    let result = []
    for (let i=0; i<nums.length; i++) {
        let tmp = target - nums[i];
        if (nums.indexOf(tmp) != -1) {
            let tmp2 = nums.indexOf(tmp)
            if (tmp2 === i) continue 
            result.push(i,tmp2)
            return result
        }
    }
};
var nums = [3,2,4]
var target = 6
console.log(twoSum(nums, target))



```
```javascript
/**
 * @param {number[]} nums
 * @param {number} target
 * @return {number[]}
 */
var twoSum = function(nums, target) {
    var result = [];
     for (var i = 0; i < nums.length; i++){
         for (var j = i+1; j < nums.length; j++){
             if (nums[i] + nums[j] == target) {
                 result.push(i)
                 result.push(j)
                 return result
             }
         }
     }
};
var nums = [2, 7, 11, 15]
var target = 26
console.log(twoSum(nums, target))

```