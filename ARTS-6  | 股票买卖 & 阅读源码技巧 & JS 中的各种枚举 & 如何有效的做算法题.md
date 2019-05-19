# ARTS-6 | 股票买卖 / 阅读源码技巧 / JS 中的各种枚举 / 如何有效的做算法题

## Algorithms

LeetCode 121. Best Time to Buy and Sell Stock

>Say you have an array for which the ith element is the price of a given stock on day i.
>
>If you were only permitted to complete at most one transaction (i.e., buy one and sell one share of the stock), design an algorithm to find the maximum profit.
>
>Note that you cannot sell a stock before you buy one.
>
>Example 1:
>
>Input: [7,1,5,3,6,4]
>
>Output: 5
>
>Explanation: Buy on day 2 (price = 1) and sell on day 5 (price = 6), profit = 6-1 = 5.
             >Not 7-1 = 6, as selling price needs to be larger than buying price.
>
>Example 2:
>
>Input: [7,6,4,3,1]
>
>Output: 0
>
>Explanation: In this case, no transaction is done, i.e. max profit = 0.

买卖股票的最佳时机。给定一个数组代表一组股票的价格，按下标从小到大的时间顺序，如何低买高卖获得最大收益？

方法一，遍历一次数组，保存到当前位置为止的最小价格，取与当前价格差较大的值存到 maxProfitSofar。

```javascript
var maxProfit = function(prices) {
    let minSofar = Number.MAX_VALUE;
    let maxProfitSofar = 0;
    for (let i = 0, length = prices.length; i < length; ++i) {
        minSofar = Math.min(minSofar, prices[i]);
        maxProfitSofar = Math.max(maxProfitSofar, prices[i] - minSofar);
    }
    return maxProfitSofar;
};
```

方法二时间复杂度与方法一相同，不过性能会更好一些。这个方法不是很好理解，需要做过[最大子序列和](https://leetcode.com/problems/maximum-subarray/)这道题。原数组相邻两个数据相减，构成的新数组。求新数组的最大子序列的和，即我们的目标结果。

为什么？因为相邻两数相减，前一组的减数，会成为后一组的被减数，所以相加刚好抵消。最后剩下的是原数组头尾两数的差值。

代码：

```javascript
var maxProfit = function(prices) {
    let maxsofar = 0;
    let maxendinghere = 0;
    for (let i = 1, length = prices.length; i < length; ++i) {
        maxsofar = Math.max(0, maxsofar + prices[i] - prices[i-1])
        maxendinghere = Math.max(maxsofar, maxendinghere);
    }
    return maxendinghere;
};
```

## Review

[Code Tells You How, Comments Tell You Why](https://blog.codinghorror.com/code-tells-you-how-comments-tell-you-why/)

代码告诉你怎么做，注释告诉你为什么这么做。是阅读源代码的一种方式，也要时刻以这种思维方式来写代码。

## Tip

关于 JS 的枚举和遍历的方法，恐怕会让很多人一头雾水，要用的时候总是要查一遍确认一下。

在这里整理一下关于 JS 对象属性的枚举，以及对象、数组的遍历方法。

### 对象

- for ( let k in object) {}

> 遍历对象上所有的可枚举属性，不包含原型链。
  
- obj.propertyIsEnumerable(“key”);

> 检查对象上是否存在某个可枚举属性，不包含原型链。

- Object.keys(obj);

> 返回 obj 上的可枚举属性，不包含原型链

- Object.getOwnPropertyNames(obj);

> 返回 obj 上所有的属性，包含不可枚举的，不包含原型链

- "p" in obj

> 检查 obj 是否存在属性 p，包括原型链

- obj.hasOwnProperty("p");

> 检查 obj 是否存在属性 p，不包括原型链

### 数组

- forEach
  
> 接受一个函数，遍历数组，不能中途终止

- every
  
> 接受一个函数，遍历数组，返回 false 时终止

- some

> 接受一个函数，遍历数组，返回 true 时终止

- for..of

> ES6 新增的一种遍历数组的语法，可以被 break 终止。内部其实生成了一个迭代器，可以通过 `Symbol.iterator` 获取。

```javascript
var myArray = [1, 2, 3];
var it = myArray[Symbol.iterator]();
it.next; { value: 1, done: false }
it.next; { value: 2, done: false }
it.next; { value: 3, done: false }
it.next; { done: false }
```

> *代码片段来自《你不知道的 JavaScript》*

## Share

这周在准备一篇比较长的博客，所以没有写分享文，就推荐一篇不错的博客吧。介绍如何做算法题，作者的方法个人很是认同。

做题不要一味的求多，要理解做题的意义所在。做算法是为了锻炼解决问题的能力，以及培养思考问题的方式方法。作者把复盘的时间精细到每个番茄钟，而不是在一道题做完之后。

要注重做题过程中的体验，不要带有不舒服感结束一次刷题，以防在下次开始时产生抗拒心理。

原文：[如何有效地写算法题](https://www.cnblogs.com/sskyy/p/8268976.html)