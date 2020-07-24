---
layout: post
title: Finding Max Of Sub Rectangle in The Matrix
subtitle: Finding Max Of Sub Rectangle in The Matrix
comments: true
---

# Problem 
Given the 2D Matrix, find the max sum of sub-matrixs 

# Run manually
0   -2  7   0
9   2   -6  2
-4  1   -4  1
-1  8   0   -2

Iterator 1 (left = 0)
right = 0, row total matrix = [0,9,-4,-1]   => max sum = 9, max_left = 0, max_right=0,max_top=1,max_bottom = 1
right = 1, row total matrix = [-2,11,-3,7] =>max sum = 15  ,max_left =0, max_right = 1, max_top = 1,max_bottom = 3
right = 2, row total matrix = [5,5,-7,7] => max  sum = 10 < 15 -> skip
right = 3, row total matrix = [5,7,-6,5]    => max sum = 11 < 15 -> skip

Iterator 2 (left = 1) 
right = 1, row total matrix = [-2,2,1,8] => max sum 11 < 15 -> skip
right = 2, row total matrix = [5,-4,-3,8]  => max sum 8 < 15 -> skip
right -=3, row total matrix = [5,-2,-2,6] => max sum 6 < 15 -> skip

Iterator 2 (left = 2) 
right = 2, row total matrix = [7,-6,-4,0] => max sum 7 < 15 -> skip
right = 3, row total matrix = [7,-4,-3,-2] => max sum 7 < 15 -> skip

Iterator 3 (left = 3)
right = 3 , row total matrix [0,2,1,-2] => max sum 2 < 15 -> skip

# Pseudo code
```text
function kanade(array A) {
    //two variable final max, max iterator
    max_so_far = 0;
    max_ending =0;
    allNegativeElements = true;
    for(i:= 0 ; i < A.length; i++) {
        max_ending = max_ending + A[i];
        if (max_ending < 0) 
            max_ending = 0;
        if (max_ending > max_so_far) {
            max_so_far = max_ending;
        }
        if (A[i] > 0) {
            allNegativeElements = false;
        }
    }
    if (!allNegativeElements) 
        return max_so_far;
    else    
        return max_in_array
}
function findSubMatrix (array) {
    var maxSum  = 0;
    left:0 , left -> num of columns
        sumTempRowArray = [0,0,0,0];
        right: left ; right -> num of columns
            sumTempRowArray = A[row iterator ][right] + sumTempRowArray[row iterator];
        
        sum = kanade(sumTempRowArray);
        if (maxSum < sum) {
            maxSum = sum;
        }
}
```
# Javascript Code
```javascript
function max_in_array(temp, numRows) {
    var max = temp[0];
    for(var i =0; i < numRows; i++) {
        if (temp[i] > max) {
            max = temp[i];
        }
    }
    return max;
}
function kadane(temp, numRows) {
    var max_so_far = 0;
    var max_ending_here = 0;
    var allNegativeF = true;
    for (var i = 0; i < numRows; i++) {
        max_ending_here = max_ending_here + temp[i];
        if (max_ending_here < 0) {
            max_ending_here = 0;
        }
        if (max_so_far < max_ending_here) {
            max_so_far = max_ending_here;
        }
        if (temp[i] > 0) {
            allNegativeF = false;
        }
    }
    if (allNegativeF) {
            return max_in_array(temp,numRows);
    } else {
        return max_so_far;
    }

}
function findMaxSum(matrix, numRows, numCol) {
    var maxSum = 0;
    var maxLeft = 0;
    var maxRight = 0;
    var maxTop = 0;
    var maxBottom = 0;
    
    for(var left=0; left < numCol; left++) {
        var temp = [0,0,0,0];
        for(var right = left; right < numCol;right++) {
            // Find sum of every mini-row between left and right columns and save it into temp[]
            for (var i = 0; i < numRows; ++i)
                temp[i] += matrix[i][right];
 
            // Find the maximum sum subarray in temp[].
            var sum = kadane(temp, numRows);
 
            if (sum > maxSum)
                maxSum = sum;
        }
    }
    return maxSum;
}
function main() {
    var a = [
        [0 ,-2 ,-7 ,0],
        [9 ,2 ,-6 ,2],
        [-4 ,1 ,-4 ,1],
        [-1 ,8 ,0 ,-2]
    ]
    var maxSum = findMaxSum(a,4,4);
    console.log(maxSum);
}
main();
```

