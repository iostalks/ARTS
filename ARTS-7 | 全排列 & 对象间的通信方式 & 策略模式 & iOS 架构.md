# ARTS-7 | 全排列 / 对象间的通信方式 / 策略模式 / iOS 架构

## Algorithms

46 Permutations

```c

Given a collection of distinct integers, return all possible permutations.

Example:

Input: [1,2,3]
Output:
[
  [1,2,3],
  [1,3,2],
  [2,1,3],
  [2,3,1],
  [3,1,2],
  [3,2,1]
]
```

求解无重复的一组数列的全排列。

全排列要实现的是每一个元素都出现过在同一个下标位置。

我们先将第 1 位依次与后面的 n - 1 为进行交换，然后第 2 位依次与后面的 n - 2 个元素交换...

最后是第 n - 1 位与第 n 个元素交换，所有的结果构成全排列。

可以发现其实每一位与后面的元素交换所做的事情是一样的，只是数据的规模不同罢了。因此我们可以采用递归来求解。

递归的终止条件是到最后一位没有元素与它交换位置。

这里需要特别注意的是，在进行下一次交换前，要将元素交换回原来的位置。

```javascript
var permute = function(nums) {
    let res = [];

    const permutaions = (list, start) => {
        if (start === nums.length) {
            res.push(list.slice());
        } else {
            for (let i = start, length = nums.length; i < length; ++i) {
                // 交换 i, satrt
                [list[i], list[start]] = [list[start], list[i]];
                // 全排列除 start 的外的剩余数
                permutaions(list, start+1);
                // [i，length] 已经实现了全排列
                // 恢复 start, i 的位置，然后 start 与 i+1 交换
                [list[i], list[start]] = [list[start], list[i]];
            }
        }
    }

    permutaions(nums.slice(), 0);

    return res;
};
```

## Review

[Communication Pattern](https://www.objc.io/issues/7-foundation/communication-patterns/)

这篇文章介绍了在 iOS 中对象间通信的五种方式。分别根据它们的特点，来决定使用场景。

## Tip

设计模式软件开发中占据的地位，并不亚于数据结构和算法。重要的东西一般都不太好理解，之所以重要是因为通用，因为通用所以比较抽象。

如果没有遇到过相关的场景，光了解设计模式的概念其实意义并不大。

本周介绍一下策略模式的使用场景。

它主要解决的问题是，抽象具有类似行为的操作。比如说绘图，我们需要绘制各种各样的形状，三角形，正方形，圆形等等。

我们可以将绘制不同类型的内容用 draw 方法封装在不同的 strategy 中。在画布 context 中持有 strategy 的实例，这个实例由使用者传给画布，画布不用关心绘制的是什么，在需要绘制的时候调用 strategy->draw 方法即可。

策略模式的优点：

1. 解耦了原本属于 context 中的 strategy 行为，便于其他画布复用。
2. 便于扩展，当有新的 strategy 加入时，不需要修改 context。

策略模式的缺点：

1. strategy 和 context 存在通信开销，例如使用 strategy 绘制图形，需要 context 传递画笔颜色，粗细等，此外可能还需要为特定的图形传递参数，比如圆角大小。这只针对于正方形使用，为了保持接口统一，其他图形也不得不接受这个参数，属于不必要的开销。
2. 如果存在很多 strategy，增加了应用中对象的数目。

## Share

周末两天主要沉浸了 [casa 的架构博客](https://casatwy.com/iosying-yong-jia-gou-tan-kai-pian.html)中，这一系列文章开阔了我看待 iOS 开发的视角。

在过去几年开发中遇到一些不爽的设计，自己又想不出很好的解决方案，错失了很多进步的机会。评论中的很多问题，都遇到过，当初解决的时候也没有考虑过方法的利弊，也没有在架构层面上思考过自己的解决方案是否合理。

通过作者的文章，学到不少思考问题，解决问题的方式。

这系列文章已经过去三四年了，现在来看依据很经典，遗憾没有早一点开始学习。

当了解了 iOS 架构需要做哪些事情的时候，基本就把 iOS 开发领域需要掌握的知识框了起来。抽象的来看客户端开发的确没有多少内容需要掌握。无外乎是：

获取外部数据 -> UI 展示

这里外部数据可以包括：网络数据、本地磁盘数据、蓝牙数据等一切计算机外设。

UI 的展示部分其实没有多大的难度，其中要细分的就是布局、渲染和动画。对于渲染如果不是做游戏，当做一个黑盒去使用，问题也不大。

不过，看起来简单的过程，中间还是有不少值得需要设计架构的点。文章中已经介绍的比较全面了，这里就不追溯了，值得每一位 iOS 开发者好好研读。

当然阅读中也不要全盘接受，要有自己的质疑和思考。