+++
title = "剑指 Offer 29. 顺时针打印矩阵"
description = ""
date = 2021-10-11T18:04:59+08:00
featured = false

comment = true
toc = true
reward = true
categories = [
  ""
]
tags = [
  ""
]
series = []
images = []

+++



```java
class Solution {
    public int[] spiralOrder(int[][] matrix) {
        ArrayList<Integer> list = new ArrayList<Integer>();
        if (matrix == null || matrix.length == 0) {
            return new int[0];
        }
        int rowStart = 0, rowEnd = matrix.length - 1;
        int columnStart = 0, columnEnd = matrix[0].length - 1;
        while (true) {
            //从左到右
            for (int i = columnStart; i <= columnEnd; i++) {
                list.add(matrix[rowStart][i]);
            }
            if (++rowStart > rowEnd) {
                break;
            }
            //从上到下
            for (int i = rowStart; i <= rowEnd; i++) {
                list.add(matrix[i][columnEnd]);
            }
            if (--columnEnd < columnStart) {
                break;
            }
            //从右到左
            for (int i = columnEnd; i >= columnStart; i--) {
                list.add(matrix[rowEnd][i]);
            }
            if (--rowEnd < rowStart) {
                break;
            }
            //从下到上
            for (int i = rowEnd; i >= rowStart; i--) {
                list.add(matrix[i][columnStart]);
            }
            if (++columnStart > columnEnd) {
                break;
            }
        }
        return list.stream().mapToInt(Integer::intValue).toArray();
    }
}
```



[剑指 Offer 29. 顺时针打印矩阵](https://leetcode-cn.com/problems/shun-shi-zhen-da-yin-ju-zhen-lcof/)

