# ARTS4 | 最长子序列 / Swift 编译和标准库 / 闭包捕获瞬时值 / lazy 的作用

## Algorithm

LeetCode 300. Longest Increasing Subsequence

>Given an unsorted array of integers, find the length of longest increasing subsequence.
>
>Example:
>
>Input: [10,9,2,5,3,7,101,18]
>Output: 4
>Explanation: The longest increasing subsequence is [2,3,7,101], therefore the length is 4. 
>Note:
>
>There may be more than one LIS combination, it is only necessary for you to return the length.
>Your algorithm should run in O(n2) complexity.
>Follow up: Could you improve it to O(n log n) time complexity?

在一个整数序列中查找最长的递增子序列，子序列不需要连续。

假设我们的输入数组是 a[0...n-1]，以 a[i] 为结尾的最长递增子序列的长度为 L(i)，那么：

L(i) = 1 + max(L(j)); 0 < j < i && a[j] < a[i]

L(i) = 1; 如果 [0,i) 不存在比 a[i] 小的元素

根据表达式，我们需要使用递归和循环来求解。使用 maxEnding 表示以 a[i] 结尾的最大子序列长度，用 maxValue 保存以所有元素为结尾的最长子序列长度的最大值。

代码如下：

```javascript
let maxValue = 1;
var lengthOfLIS = function(nums) {
    if (!nums.length) return 0;

    lis(nums, nums.length);

    return maxValue;
}

const lis = (arr, n) => {
    if (n === 1) return 1;

    // 存放以 a[0] ..< a[n-1] 作为结尾的最长子序列的长度
    let maxEnding = 1;

    for (let i = 1; i < n; ++i) {
        // 获取以 a[i] 结尾的的最长子序列的长度
        const valI = lis(arr, i);
        // 更新以 a[n-1] 为结尾的序列长度 maxEnding
        if (arr[i] < arr[n-1] && valI + 1 > maxEnding) {
            maxEnding = valI + 1;
        }
    }

    // 更新 a[0..<n] 结尾的子数组的的最长子序列
    if (maxValue < maxEnding) {
        maxValue = maxEnding;
    }

    return maxEnding;
}
```

使用递归会存在重复计算最大子序列的问题，画出递归树可以看出，该算法的时间复杂度为 O($2^n$)。

我们可以使用一个数组，缓存计算过的最长子序列长度。

代码如下：

```javascript
var lengthOfLIS = function(nums) {
    if (!nums.length) return 0;
    return lis2(nums, nums.length);
}

const lis2 = (arr, n) => {
    if (n === 1) {
        return 1;
    }

    // 保存以a[i]结尾的最大子序列长度
    // 第一个元素长度为 1 先填好
    let maxState = [1];

    for (let i = 1; i < n; ++i) {
        maxState[i] = 1;
        // 查找以a[i]结尾的最大子序列长度
        for (j = 0; j < i; ++j) {
            if (arr[i] > arr[j] && maxState[j] + 1 > maxState[i]) {
                maxState[i] = maxState[j] + 1;
            }
        }
    }

    let maxValue = 0;
    for (let i = 0; i < n; ++i) {
        if (maxState[i] > maxValue) {
            maxValue = maxState[i];
        }
    }
    return maxValue;
}

```

引入数组 maxState 缓存后，时间复杂度降到 O($n^2$)

不过这还不是最优的解法，最快的解法时间复杂度能够降到 O(nlog(n))，思路跟前面的完全不一样。

先不考虑最长子序列，同一长度的递增子序列可能有多个，比如说 {2, 7, 5}，长度为 2 的子序列有 {2,5}, {2,7}，如果下一个元素是 6，那么 {2,7} 对我们来说没有意义，同一长度的子序列，我们只关心最小的尾数。

使用一个数组 tail 来保存每一种长度子序列最小的末尾元素, tail 将会是个递增数组。从数组第一个元素开始，逐个考察，如果元素比 tail 数组的末尾元素大，我们就将该元素添加进入 tail。

如果比 tail 小，就查找该元素所在的区间，用它替换 tail 元素中，比它大的最小的元素。因为 tail 是有序的，所以可以使用二分查找。

```javascript
var lengthOfLIS2 = function(nums) {
    const n = nums.length;
    let tail = Array(n).fill(0);
    let s = 0; // tail 的长度
    for (let i = 0; i < n; ++i) {
        let l = 0;
        let r = s;
        // 二分查找
        while (l != r) {
            const m = ((l + r) / 2) | 0; // m 取整
            if (nums[i] > tail[m]) {
                l = m+1;
            } else {
                r = m;
            }
        }
        // 二分查找结束 l 对应目标放置位置
        tail[l] = nums[i];
        // 如果目标位置是 tail 的长度，说明是添加到尾部的
        if (l === s) ++s;
    }
    return s;
};
```

通过这种解法，其实我们实际上在解决的是，如何向一个有序数组中插入(替换）一个数，如果这个数比数组末尾元素大，就添加这个元素到末尾，如果这个数小于末尾元素，我们就找比它大但是最小的元素替换。

我觉得很多算法难，可能主要是难在，如何将已知问题，转换为另一个可操作性更强的问题。

## Review

Swift 编译和标准库

<https://swift.org/compiler-stdlib/#compiler-architecture>

这是 Swift 官方文档的一篇文章，介绍了 Swift 的编译过程，主要分为：

- 解析（Parsing）：通过词法分析、语法分析生成 token，再根据 token 生成抽象语法树（AST）；
- 语义分析（Semantic analysis）：对上下问进行逻辑审查，类型检测等；
- 导入 Clang 模块（Clang importer）：Clang 会将 OC 的 API 映射为 Swift 的 API；
- 中间语言生成（SIL generation）：生成 Swift 中间语言；
- 中间语言安全转换（SIL guaranteed transformations）：对 SIL 进行数据流诊断，保证程序的正确性；
- 中间语言优化（SIL Optimizations）：对中间语言进行优化，包括 ARC 优化，去虚拟化，泛型优化等；
- 生成 IR（LLVM IR Generation）：将 SIL 转为更底层 LLVM IR，LLVM 进一步优化后生成机器码。

Swift 的编译步骤，比其他 C 系语言复杂，所以速度更慢。

Swift 的标准库主要被分为三部分：

- 核心标准库：主要包括数据类型，协议，函数等；
- 运行时（Runtime）：位于编译器和核心标准库之间，提供动态化的功能，元数据，内存管理等；
- SDK Overlays：为不同的平台提供 OC 和 Swift 的互通性。

## Tip

如何使用 Swift 闭包捕获变量的瞬时值？

我们先来看一个例子：

```swift
var closureArray_1 = [()->()]()

var i = 1;
for _ in 1...5 {
    closureArray_1.append {
        print(i)
    }
    i += 1;
}

closureArray_1[0]() // prints 5
closureArray_1[1]() // prints 5
closureArray_1[2]() // prints 5
closureArray_1[3]() // prints 5
closureArray_1[4]() // prints 5
```

我们向数组中添加 5 个闭包（尾随闭包），每个闭包都会捕获变量`i`，在执行数组中闭包时，打印的值都是 5，是因为闭包捕获的是变量`i`的地址，而不是在闭包加入数组时的瞬时值。

那么该如何捕获变量`i`的瞬时值呢？方法是使用捕获列表（capture list)，把需要保存瞬时值的变量使用方括号包裹。

```swift
var closureArray_1 = [()->()]()

var i = 1;
for _ in 1...5 {
    closureArray_1.append { [i] in
        print(i)
    }
    i += 1;
}

closureArray_1[0]() // prints 1
closureArray_1[1]() // prints 2
closureArray_1[2]() // prints 3
closureArray_1[3]() // prints 4
closureArray_1[4]() // prints 5
```

执行闭包输出的值为变量 i 的瞬时值，每次循环闭包会拷贝一份 i 的临时值，不会再受外部 i 变化的影响。

来源：<https://medium.com/@abhimuralidharan/functional-swift-all-about-closures-310bc8af31dd>

capture list 非常常用的场景是修改捕获变量的内存管理方式，例如：

```swift
lazy var greeting : String = { [weak self] in
    return “Hello \(self?.name)”
}()
```

而且，它还可以使用别名，使局部变量更加清晰：

```swift
// 这里其实没必要修改内存管理方式
lazy var greeting : String = { [weak weakSelf = self] in
    return “Hello \(weakSelf?.name)”
}()
```

## Share

### lazy 的作用

在 ViewController 中声明一个不带 lazy 的属性，该属性值为闭包执行后的返回值：

```swift
var value = { () -> Int in
    print(self)
    return 1;
} ()
```

当使用 self.value 获取属性值时，输出的 self:

```swift
(Function)
```

当在 var 前面加上 lazy 修饰符时，输出变为：

```swift
<JSFamily.ViewController: 0x7f9a9f509de0>
```

为什么用 lazy 声明后，闭包捕获的 self 就不一样呢？

没有 lazy 修饰的属性值，需要在 ViewController 初始化完成前确定，也就是说闭包会在 ViewController 初始化完成前执行，因为 ViewController 没有初始化完成，也就没有 ViewController 实例，self 不可能指向它，而是指向了闭包。

> Swift 在编译时会检测 self 的使用，而不像在 Objective-C 中，能够随意的在 init 方法中使用 self，这存在一定的安全隐患。

而使用 lazy 声明后，闭包在 value 第一次使用时才执行，此时 ViewController 已经初始化完成，self 的指向为 ViewController 实例。

lazy 关键字，是非常实用的一个功能。在 Objective-C 中使用懒加载的方式来实现 getter 方法，需要自己来控制，属性只被初始化一次。而在 Swift 中这些都是隐式实现的。lazy 修饰的立即执行闭包，仅会被执行一次，并将执行结果存储在 value 中。

这么做的好处是，可以加速对象的初始化，减少对象初始化的内存开销。

lazy 修饰的属性，如果关联立即执行的闭包还有一个特点，就是该闭包是非逃逸(noescap)的，不会持有在闭包中使用的 self，也就不会导致循环引用。

不过 lazy 虽然好用，但它是线程不安全的，如果并发访问，可能被初始化多次。
