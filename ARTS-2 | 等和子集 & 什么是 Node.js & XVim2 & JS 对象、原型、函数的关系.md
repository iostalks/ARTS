# ARTS-2 | 等和子集 / 什么是 Node.js / XVim2 / JS 对象、原型、函数的关系

## Algorithm

LeetCode 416. Partition Equal Subset Sum
>Given a non-empty array containing only positive integers, find if the array can be partitioned into two subsets such that the sum of elements in both subsets is equal.

>Note:
>
>Each of the array element will not exceed 100.
>The array size will not exceed 200.
>
>Example 1:
>```
>Input: [1, 5, 11, 5]
>
>Output: true
>
>Explanation: The array can be partitioned as [1, 5, 5] and [11].
>```
>
>Example 2:
>```
>Input: [1, 2, 3, 5]
>
>Output: false
>
>Explanation: The array cannot be partitioned into equal sum subsets.
>```

将一个数组分为两个子集，判断是否存在两个子集的和相等的情况。如果两个子集和相等，即原数组和的一半。

题目就变为判断数据是否存在一个子集，它的和等于数组所有元素和一半，我们可以先求出数组的总和，这个即为已知量。

这就转化为典型的 0/1 背包问题。 遍历数组，每次可以选择或者不选择这个元素，直到选择的元素和等于目标值。

第 i 次抉择后的和依赖于前一次选择后的结果。我们使用一个二维数组 states，第一维度代表元素的个数，用于记录对第 i 个元素是否选择，第二个维度代表当前选择的所有元素的总和。

如果第 i 次抉择的元素和为 j, 就将 states[i][j] 设置为 true, 要获得的 states[i][j] 的结果有两种情况:

- 没有选择第 i 个元素，那么 states[i][j] 与 states[i-1][j] 的值保持一致；

- 选择了第 i 个元素，那么 state[i][j] 与 states[i-1][j-nums[i]] 的值保持一致；

递推公式为：(states[i-1][j], states[i-1][j-nums[i]]) -> states[i][j]

将 nums = {1, 5, 5, 11 } 递推情况画成表格如下：

![image]()

第 0 个元素选择需要特殊处理，不选择时和为 0，选择和为 nums[0]，即 1。
接下来的选择，都是基于前一次，选择或不选择的结果。红色代表不选择本次元素的结果（即与上一次相同），绿色表示选择了本次的元素。

```javascript
var canPartition = function(nums) {
    // 数组求和
    let sum = nums.reduce((c, s) => c + s, 0)
    // 如果和为奇数，说明不存在两个和相等的整数子集
    if (sum & 1 == 1) return false;
    // 取一半，作为背包大小
    sum = sum >> 1;

    const n = nums.length;
    // 二维数组容器
    const states = [];

    // 选择第一个元素两个情况
    let row0 = [];
    row0[0] = true;
    row0[nums[0]] = true;
    states.push(row0);

    // 构造二维数组的 1..<n 子数组组
    for (let i = 1; i < n; ++i) {
        let row = [];
        states.push(row);
    }

    // 从第二个元素开始考虑
    for (let i = 1; i < n; ++i) {
        for (let s = 0; s <= sum; ++s) {
            // 不选择第 i 个元素
            states[i][s] = states[i-1][s];
            // 选择第 i 个元素
            if (states[i-1][s-nums[i]]) {
                states[i][s] = true;
            }
        }
    }
    // 最后一行的 sum 是否被填充
    return states[n-1][sum] == true;
}
```
提交了下，发现效率极低。上述代码的时间复杂度和空间复杂度都是 `O(m*n)`;

其实空间复杂度和时间复杂度都能够再优化，通过上图其实能发现，在纵向上只要有一个为 true，一列都会是 true，所以可以将二维数组纵向压缩成一维的。

时间上我们遍历到了整个数组的长度才终止，其实在找到目标子集和为 sum 时，就可以提前终止循环。

```javascript
var canPartition = function(nums) {
    let sum = nums.reduce((c, s) => c + s, 0)
    if (sum & 1 == 1) return false;
    sum = sum >> 1;

    const n = nums.length;
    const states = [];

    states[0] = true;
    states[nums[0]] = true;
    for (let i = 1; i < n; ++i) {
        for (let j = sum - nums[i]; j >= 0; --j) {
            if (state[j]) {
                states[j+nums[i]] = true;
            }
        }
        if (states[sum]) return true;
    }
    return false;
}
```

转成一维的后，从后往前遍历可以减少重复计算。

## Review
经常会听到前端的同学讲写 node，但 node 也是用 JS 写的，它能不能算作是一门独立语言呢？

这篇文章详细的介绍了什么是 Node.js：

https://medium.freecodecamp.org/what-exactly-is-node-js-ae36e97449f5

node 是 Node.js 的简称，在谈论 node 的时候，讲的就是 Node.js。

早期 JavaScript 只能运行在浏览器中，浏览器内置了 JavaScript 引擎，例如 Chrome 浏览器使用的是 V8 引擎。Node.js 使得 JS 能够脱离浏览器直接在机器上运行。

从语法上看 Node.js 就是 JS 的语言，它也是使用 V8 引擎解析的， 此外 Node.js 为 JS 运行时提供与系统交互的能力，比如非阻塞的文件、网络 I/O 等功能，就像浏览器为 JS 提供操作 DOM 的接口。

这篇文章还介绍了 JavaScript 作为单线程语言，是如何实现伪多线程的。

我们应该如何定义一门语言，语言应该由哪些部分组成的，仅仅是语法么？比如说 Objective-C，如果没有 runtime 它就无法工作，所以我认为运行时也是语言的一部分。

Node.js 为 JavaScript 提供不同于浏览器的运行时的能力，所以可以认为是一门独立语言。

## Tip

每次 Xcode 更新都需要重新安装一下 Vim，这次 Xcode 10.2 安装后发现 Vim 无法生效，就完全卸载掉重新安装了一下。

以下是安装的基本介绍：

在 Xcode 8 之后，就不支持第三方插件了，官方推出了 Extensions，支持扩展 Xcode 的功能。然而 Vim 插件接入 extensions 功能会受到极大的限制，所以 XVim2 插件没有接入，要安装 XVim2 需要对 Xcode 进行重签名。

Xcode 重签名其实很简单，打开“钥匙串”应用程序，选中“登录”，然后按如下如创建证书。

![image]()

为证书取一个名字，这里使用 XcodeSiger，并选择自签名类型。

接着在终端中执行如下重签名命令（最后一个参数是 Xcode 的路径）：

```
sudo codesign -f -s XcodeSigner /Applications/Xcode.app
```

重签名后，下载 Vim 插件代码到本地:

```
$ git clone https://github.com/XVimProject/XVim2.git
```

使用 xcode-select 确认希望安装 Vim 的 Xcode 版本的路径：
```
$ xcode-select -p
/Applications/Xcode.app/Contents/Developer
```
如果输出的路径不是目标 Xcode 的路径使用`xcode-select -s`手动设置。

进入到 XVim 的目录，编译插件：
```
cd XVim2
make
```
重启 Xcode 会提示加载插件，如果没有提示，使用如下命令激活：
```
defaults delete  com.apple.dt.Xcode DVTPlugInManagerNonApplePlugIns-Xcode-X.X   (X.X 是 Xcode 版本)
```

Github 地址：https://github.com/XVimProject/XVim2

## Share
理解 JavaScript 对象、原型、函数之间的关系。

JavaScript 中是没有类的概念的，虽然在 ES6 中引入了类，其实是个语法糖，还是会被编译成函数。

如果没有类，JavaScript 是如何实现数据或者方法在对象中复用的呢？它是基于原型模式设计的。

当我们声明了一个函数 Person，它默认会拥有`prototype`属性，该属性的值指向称 P。P 会默认具有`constructor`属性指向 Person 函数。

```
Person.prototype.constructor == Person // true
```

我们再通过这个函数创建一个对象：

```
const person = new Person();
```

person 对象默认又会具有`[[prototype]]`属性，这个属性指向 P。我们称 P 是 person 的原型。

```
person.[[prototype]] == P // true
```

它们的关系图如下：

[image]()

它们三者的关系看起来非常复杂，但实际目的只有一个，使我们创建的 person 能够“继承”P，从而复用 P 的方法和属性。

person.toString() // "[object Object]"

在创建 person 时 并没有 toString 属性，也没有在 Person 函数中声明过，而是使用了原型中定义的方法。person 与原型的关系是通过函数隐式构建的。

严格意义上讲，上图是不对的，person 对象的 [[prototype]] 和 constructor 属性其实是通过原型链间接获得的，但如果从访问者角度看，它们是成立的。

其实，函数和原型也都是对象，只是在不同的时候需要扮演的角色不同，如果将他们混为一谈的去看，可能会觉得很凌乱。

如果当把函数当函数看，它就是用来调用，或者创建对象，并链接到原型的。如果把函数当对象看，它也是由函数创建，并具有 [[prototype]] 属性的指向它的原型 Function 对象。

对象都会具有 [[prototype]] 和 constructor 属性，前者指向它的原型，后者指向它的构造函数。函数都会具有 prototype 属性，所有由该函数构建的对象都会指向它。




