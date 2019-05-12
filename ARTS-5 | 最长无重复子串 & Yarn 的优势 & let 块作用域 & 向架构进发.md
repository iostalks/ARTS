# ARTS-5 | 最长无重复子串 / Yarn 的优势 / let 块作用域 / 向架构进发

## Algorithm

LeetCode 3. Longest Substring Without Repeating Characters

>Given a string, find the length of the longest substring without repeating characters.
>
>Example 1:
>
>Input: "abcabcbb"
>
>Output: 3 
>
>Explanation: The answer is "abc", with the length of 3. 
>
>Example 2:
>
>Input: "bbbbb"
>
>Output: 1
>
>Explanation: The answer is "b", with the length of 1.
>
>Example 3:
>
>Input: "pwwkew"
>
>Output: 3
>
>Explanation: The answer is "wke", with the length of 3.
> Note that the answer must be a substring, "pwke" is a subsequence and not a substring.

在一个字符串中，查找无重复字符的最长子串，返回它的长度。

逐个扫描字符，如果遇到前面已经出现的字符，就重新开始计算长度。这里会面临两个问题：

- 如何记录已经出现过的字符
- 相同字符出现多次该如何处理

记录每次字符最近出现过的下标，当遇到相同字符时，获取上次出现的下标，中间间隔的长度，即为无重复子串的长度，对比所有这样的子串的长度，取长的一个。

不过这样会有个 bug，两个重复的字符之间的子串可能存在重复字符。使用 lastStart 记录最近出现的重复字符的隔离前面的重复字符。

```javascript
/**
 * @param {string} s
 * @return {number}
 */
var lengthOfLongestSubstring = function(s) {
    if (!s || s.length === 0) return 0;
    const map = new Map();
    let maxSubLength = 0;
    let lastStart = 0;

    for (let i = 0, n = s.length; i < n; ++i) {
        const c = s.charAt(i)
        const j = map.get(c);
        // for case： ”abba“，”abcabcbb"
        if (j >= lastStart) {
            lastStart = j;
        }
        map.set(c, i);
        if (i - lastStart > maxSubLength) {
            maxSubLength = i - lastStart;
        }
    }
    return maxSubLength;
};
```

## Review

Yarn 是 JavaScript 包管理工具，它在 npm 的基础上做了优化。

<https://code.fb.com/web/yarn-a-new-package-manager-for-javascript/>

npm 安装依赖包的速度不够快，在 5.0 之前拉取的 packages 版本可能不同，更严重的是安装 packages 时，会自动执行依赖的代码，这可能存在安全隐患。

为了解决 npm，FaceBook、Google 等公司联合推出了 Yarn。Yarn 并发拉取 packages，速度得到大幅度提升，并且 Yarn 引入版本锁定文件，确保了不同设备上拉取的版本一致。

## Tip

下面两段代码分别输出什么

```javascript
function foo() {
    for (var i = 1; i <= 5; ++i) {
        setTimeout(()=> {
            console.log(i);
        }, i * 1000)
    }
}
foo()

function foo1() {
    for (let i = 1; i <= 5; ++i) {
        setTimeout(()=> {
            console.log(i);
        }, i * 1000)
    }
}
foo1()
```

第一段代码每个一秒钟输出一个 5，而第二段代码输出的是 1 2 3 4 5。为什么？

var 声明的变量作用域会被提升到 foo 所在的作用域，setTimeout 闭包持有的是 i 的一个引用，timeout 闭包打印发生在五次循环执行结束后的最终值 5。

let 会在每次循环的时候重新声明，因为 let 声明的变量只有块作用域，每次循环会重新创建块作用域，setTimeout 闭包捕获 i 所在的 5 个不同的作用域，所以能依次输出 1-5。

## Share

这周分享一个 ggTalk 的播客：<https://talk.swift.gg/26>