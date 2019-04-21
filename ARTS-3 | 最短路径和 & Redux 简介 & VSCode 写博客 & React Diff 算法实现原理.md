# ARTS-3 | 最短路径和 / Redux 简介 / VSCode 写博客 / React Diff 算法实现原理

## Algorithm

LeetCode 64. Minimum Path Sum

>Given a m x n grid filled with non-negative numbers, find a path from top left to bottom right which minimizes the sum of all numbers along its path.
>
>Example:
>
>Input:
>
>[
>
> [1,3,1],
>
> [1,5,1],
>
> [4,2,1]
>
>]
>
>Output: 7
>
>Explanation: Because the path 1→3→1→1→1 minimizes the sum.

从矩形的左上角走到右下角，只能向右或者向下走，寻找走过的格子和最小的值。

这道题最直接的方法是使用回溯来解决，向右或者向下一直走，直到最右下角，然后回来继续走另一种情况。代码如下：

```javascript
let min = Number.MAX_VALUE;
const recall = (grid, i, j, dist) => {
    const m = grid.length;
    const n = grid[0].length;

    // 累加走到的格子
    dist += grid[i][j];

    // 走到右下角终点，判断是否是最小值
    if (i == m-1 && j == n-1) {
        if (dist < min) {
            min = dist;
        }
        return;
    }

    // 向下走
    if (i < m-1) {
        recall(grid, i+1, j, dist);
    }

    // 向右走
    if (j < n-1) {
        recall(grid, i, j+1, dist);
    }
}

var minPathSum = function(grid) {
    if (!grid.length || !grid[0].length) {
        return 0;
    }

    recall(grid, 0, 0, 0);
    return min;
};

```

使用 dist 记录所有走到右下角格子的情况，将最小值存在 min 变量中。

然而使用回溯会存在重复计算的中间状态。这道题适合使用动态规划，到达当前格子的路径和，可以有左边和上边格子的路径和取最小值得到。

使用同样大小的二维数组 states，存储到达格子的路径和，递推公式为：

```javascript
(states[i-1][j], state[i][j-1]) -> states[i][j]
```

 动态规划的代码如下：

```javascript
var minPath = (grid) => {
    let states = []
    const m = grid.length;
    const n = grid[0].length;

    // 初始化第一行的数据
    let row = []
    let sum = 0
    for (let j = 0; j < n; ++j) {
        sum += grid[0][j];
        row[j] = sum;
    }
    states.push(row);

    // 初始化第一列的数据
    sum = grid[0][0];
    for (let i = 1; i < m; ++i) {
        let row = [];
        sum += grid[i][0];
        row[0] = sum;
        states.push(row);
    }

    for (let i = 1; i < m; ++i) {
        for (let j = 1; j < n; ++j) {
           states[i][j] = grid[i][j] + Math.min(states[i-1][j], states[i][j-1]);
        }
    }

    return states[m-1][n-1];
}
```

我们将二维数组的第一行和第一列填好，然后使用递推公式，从左边和上边格子取最小值，加上当前格子的值，填入表格。

循环中每次决策的结果都是到达该格子最小的路劲和，所以 states[m-1][n-1] 的值为到达二维数组右下角的最小值。

## Review

本周阅读的文章比较基础，介绍前端状态管理框架 Redux，它的设计初衷，以及工作原理。

<https://blog.logrocket.com/why-use-redux-reasons-with-clear-examples-d21bffd5835>

前端和客户端虽然都会负责展示页面，但是在数据层的维护和管理还是存在较大的差异。因为在客户端上屏幕比较小，每个页面的设计会比较单一，格式不同的数据不会太多。

在电脑端展示的网页，一般信息量都比较大，视图层级也会比较复杂。很多视图之间需要共享数据，这就考虑考虑如何维护和复用这些数据。功能页面作为独立的 Component 数据需要由父视图传递，如果视图层级比较深，就需要层层传递。

比如层级比较深的两个兄弟组件需要共享数据，需要将数据源通过它们的祖先视图一层一层传递，最终由它们共有的父视图提供数据。如果只是展示可能还好，但如果这两个兄弟节点存在交互，需要将视图 A 的数据传给视图 B 展示，这就需要通过一层层向上传递回调，然后修改数据源，再将数据源层层传递到 B。

原本只是临近的两个视图，通信变得异常复杂。如果一个页面存在大量这种情况，数据管理将变得不可控，处于中间层的视图组件，持有数据但自身没有用于展示，而是传给子组件，也显得不太合理。

Redux 就会为了解决这个问题发展而来的。

Redux 使用中心化的思想，整个程序的数据用一颗状态树来维护，每一个视图组件都可以独立的访问数据源，并使用纯函数更新状态。

不过 Redux 也不是万能的，如果应用的数据复杂度还在可控范围内，也没有必要上 Redux。Redux 将 store 中的数据做到高度复用，也有一定的弊端，某个组件在使用过程中更新的数据源，销毁时需要对数据源进行清理，以供其他组件使用。

## Tip

这周介绍一个用 VS Code 写博客的小 Tip。

我使用过很多写博客相关的软件 MWeb、Bear、Ulysses 等等，它们虽然都有优点，但都存在各式各样的小缺点，一直希望能有个集它们所长的工具。

MWeb 能够一边写一遍预览，然而个人并不喜欢它预览的样式。Ulysses 是我每次发公众号必用的工具，它支持自定义 css 样式，能够直接预览的效果拷贝到微信公众号的编辑器里。不过，它对代码的支持非常差，每次都需要重新调整代码样式……

Bear 是我经常用的一个工具，使用 tag 自动分文件夹，并支持多端同步，然而它自带的 html 导出样式令人无法接受……

后来发现 VSCode 有 MarkDown 预览插件，支持自定义 CSS 样式，以及第三方插件提供的样式，比如说受程序员爱戴的 Github 样式。为了便于考虑在手机上的显示效果，我一般把预览窗口拖成手机屏幕的宽度，方便预览段落的高度。

最吸引我的是，在 VS Code 里面能够使用 Vim，习惯了用 Vim 写代码，在普通文本编辑器里，看到插入光标闪烁就会很没有安全感。不过使用 Vim 输入中文有一个体验不好的地方，切换到 Normal 模式，需要将输入法切到英文，再次进入 Insert 模式又要切回中文。

不过只要是程序员自己的需求，就没有实现不了的，如果使用的是 VSCodeVim，只要安装了 im-select 并做简单的配置，就能智能的实现中英文的自动切换了。

此外，VSCode 的 PicGo 插件支持自动上传图片到图床，非常方便。

![arts_3_vscode_preview.png](https://raw.githubusercontent.com/iostalks/images/master/blog/arts_3_vscode_preview.png)

## Share

写 React Native 也挺长时间的，一直没有对 React 的实现做深入了解。这周阅读了几篇 React 实现原理相关的文章，对它的核心功能有了大致的了解。

React 将每一个节点视图作为一个 Component，每个 Compoent 对应一个 Visual-DOM，引入 Visual-DOM 是为了将 DOM 对象化，在更新页面时，通过 Visual-DOM 对比前后的差异，只渲染变化的部分，实现  Actual-DOM 的最小化更新。

React 能够做到高性能的关键是，快速的计算出页面的变化情况，依赖于 Diff 算法的实现。传统的树形结构对比，时间复杂度高达 O(n^3)。

React 的 Diff 算法实现 O(n) 的复杂度，之所有能将复杂度降到最低，是因为它基于以下两种假设：

1. 不同类型的元素节点，就认为是不同的树，直接砍掉；
2. 使用 key 作为节点的唯一标识；

从根节点开始对比，如果根节点的节点类型不相同，就直接重新渲染整棵，不会去对比是否有相同的子树能够复用。

如果节点类型相同，继续对比节点的属性，如果属性发生变化，会在原属性上进行新增或者删除操作。

递归调用子节点，同样根据上面的两种情况做更新，这就是 Diff 算法的实现原理，是不是很简单。。

然而，这种算法并不是完美的，如果父节点不同，就不会再去对比子节点的情况了，如果只是修改父节点的类型，子节点完全一样，整棵树都会重新渲染。如果存在大量这种情况，性能会变得非常差。

好在实践中其实这种情况少之又少，知道具体的实现后，也会有意识避开这种情况。

所谓优秀的算法，其实是一种取舍，能够符合我们使用场景的算法才是好算法。