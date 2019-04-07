# ARTS-1 | 最短编辑距离 / 工程师进阶 / UIScrollView 事件传递 / 技术的价值

## Algorithm
LeetCode 72 Edit Distance:
> Given two words *word1* and *word2*, find the minimum number of operations required to convert *word1* to *word2*.
> You have the following 3 operations permitted on a word:
> 1. Insert a character
> 2. Delete a character
> 3. Replace a character
> 
> Example 1:
> **Input:** word1 = “horse”, word2 = “ros”
> **Output:** 3
> **Explanation:** 
> horse -\> rorse (replace ‘h’ with ‘r’)
> rorse -\> rose (remove ‘r’)
> rose -\> ros (remove ‘e’)

计算两个单词的最短编辑距离，所谓编辑距离是指从单词 a 到单词 b 需要做的修改次数，插入、删除、替换各算一次。

我们以下标 i 和 j 作为单词 a 和 b 当前关注的目标字符。如果 a[i] 和 b[j] 相等不需要编辑，如果不相等我们用 dist 记录操作步骤，可将需要编辑操作细分为六种情况：

1. 在 a 的 i 下标处插入 b[j] 的值，dist 加一，继续考察 a[i] 和 b[j+1]。
2. 在 b 的 j 下标处插入 a[i] 的值，dist 加一，继续考察 a[i+1] 和 b[j]。
3. 将 b 的 j 下标字符删除，dist 加一，继续考察 a[i] 和 b[j+1]。
4. 将 a 的 i 下标字符删除，dist 加一，继续考察 a[i+1] 和 b[j]。
5. 将 a 下标 i 的字符替换掉 b 下标 j 的字符，dist 加一，继续考察 a[i+1] 和 b[j+1]。
6. 将 b[j] 的字符替换掉 a[i] 的字符，dist 加一，继续考察 a[i+1] 和 b[j+1]。

其中我们能够发现 1 和 3、2 和 4、5 和 6，字符偏移情况是一致的，可以归为三类。

使用回溯的方式，分别考察每一种情况的编辑距离，取最小值即为最短编辑距离。

用代码表达是这样：

	function minDistanceHelper(a, b, i, j, dist) {
	    const m = a.length;
	    const n = b.length
	    if (m === i || n === j) {
	        if (i < m) {
	            // a 单词比 b 单词长，需要补齐 b 单词的距离
	            dist += m - i;
	        }
	        if (j < n) {
	            dist += n - j;
	        }
	        if (dist < minDist) {
	            minDist = dist;
	        }
	        return
	    }
	    if (a[i] == b[j]) {
	        // 不需要编辑
	        minDistanceHelper(a, b, i+1, j+1, dist);
	    }
	    minDistanceHelper(a, b, i+1, j, dist+1); // 2、4
	    minDistanceHelper(a, b, i, j+1, dist+1); // 1、3
	    minDistanceHelper(a, b, i+1, j+1, dist+1); // 5、6
	}
	
	var minDist = Number.MAX_VALUE;
	var minDistance = function(word1, word2) {
	    console.log(word1, word2);
	    const m = word1.length;
	    const n = word2.length;
	    minDistanceHelper(word1, word2, 0, 0, 0);
	    return minDist;
	};

不过这里使用回溯的效率很低，存在很多重复计算的情况，并且还需要全局变量维护结果，甚是丑陋。

我在 leetcode 上测试居然没有通过，失败的 case 是`intention`和`execution`，然而单独测试这两个单词结果却是正确的，还是第一次遇到这种情况。

通过前面的分析可以发现，当前最短编辑距离，是在前一次最短编辑距离的基础上加一，这其实是一个递推的过程，可以用动态规划的思想来做。

当前 (i, j, minDist) 的状态可以由三种子状态推导而来，分别是(i-1, j-i, minDist）、(i-1, j, minDist)、(i, j-1, minDist)。我们只需求出三个子状态中的最小值即为最短编辑距离。

将状态转移以二维状态表的形式组织，刚好表示左边，上面，右上角上个位置，我们取三个位置的最小值填充当前，最后达到二维数据右下角的值即代表最小编辑距离。

如下是单词`abc`和`acb`的填表结果。

file:///.file/id=6571367.8657679112

第一行和第一列代表从空字符到当前字符结尾的单词，需要编辑的距离，由单词长度决定，所以可以先填上。之后根据左、上、左上的值来计算当前格子的最短编辑距离。如果当前对比的两个字符不相等，则由这三个位置上的编辑距离加一，求最小值得到，如果当前对比的两个字符相等，可能有两种情况得来：

一、由前一步的单词增加或删除字符得到相同的字符，(i-1，j)/(i, j-1) -\> (i, j)，即上面和左边递推的情况，此时编辑距离要加一。
二、如果 i 和 j 都由前一步加一得到，(i-1，j-1)-\> (i, j)，并且字符相等，代表这一步没有做任何编辑操作，所以编辑距离不需要变化，也就是左上角的递推情况。

完整的代码如下：

	function min(a, b, c) {
	    let min = a;
	    if (b < min) {
	        min = b;
	    }
	    if (c < min) {
	        min = c
	    }
	    return min;
	}
	
	var minDistance = function(word1, word2) {
	    const m = word1.length + 1, n = word2.length + 1;
	    const minDist = [];
	    const row0 = []
	    // 填充第一行数据
	    for (let j = 0; j < n; ++j) {
	        row0.push(j);
	    }
	    minDist.push(row0);
	    // 创建 1~m 个数组，填充第一个元素，也就是二维数组的第一列
	    for (let i = 1; i < m; ++i) {
	        let row = [];
	        row[0] = i;
	        minDist.push(row);
	    }
	    // 填 minDist 表 [1...m][1...n]
	    for (let i = 1; i < m; ++i) {
	        for (let j = 1; j < n; ++j) {
	            if (word1[i-1] == word2[j-1]) {
	                minDist[i][j] = min(minDist[i-1][j-1], minDist[i][j-1] + 1, minDist[i-1][j] + 1);
	            } else {
	                minDist[i][j] = min(minDist[i-1][j-1] + 1, minDist[i][j-1] + 1, minDist[i-1][j] + 1);
	            }
	        }
	    }
	
	    return minDist[m-1][n-1];
	}


## Review

英文阅读，工程师职业发展所需的 12 种软技能，分为两部分，这里是原文的第一部分。

 [https://www.asme.org/engineering-topics/articles/technology-and-society/12-skills-need-advance-career-part-1][1] 

虽然文中讲述的是工程师，没有说具体的工种，但作为程序员，也属于工程师的一种。这些软技能，也有一定的参考价值

绝大多数工程师有较强的解决问题的能力，但常常忽略了软技能的重要性。能面试进入一个好公司，说明技术能力够硬，之后的天花板可能不在技术上，而在于软技能，比如说如何自我表现，发现问题的能力，与人良好的沟通能力，这都会影响一个人职业发展的方向。

不过不希望只是做一名普通的工程师，需要掌握的一些软技能。

一、沟通。沟通是绝大多数工程师的一大挑战。包括我自己经常不知道跟人聊什么，常常处于被动沟通的状态，很难想到什么能主动与人沟通的事情。但实际上，善于找人沟通的人，或许技术实力一般，往往也会有更多的发展机会。

二、做演讲。不要使用行话，其实写文章也是，直白的语句听起来会更加顺畅自然。做演讲，不要害怕受到听众的质疑，如果有问题就汇聚大家的想法，最终汇总出统一的答案。这其实也是管理者必备的能力。

三、自我销售。只要是在打工，其实就是在卖自己，卖不是一味的要价，而是不停的展现自己的实力，让各个买方来来出价。

四、与众不同。除了工作所需的技能，需要有第二技能，以便区别的他人。比如说写作，表达能力，或者只是能够提出好的问题。

五、谈判自我价值。大意是要主动的争取机会，争取更有挑战的工作，而不仅只是接受分配。

六、自信。自信的表达出自己能做什么，而不是吞吞吐吐。

## Tip
在开发 React Native(0.43) 过程中，我发现将 ScrollView 放置在 TouchableOpacity 上时会无法响应滚动事件，就对它们的实现做了些了解。在 RN 中 TouchableX 组件是由 View 实现的，在 JS 端可通过`onStartShouldSetResponder`控制 View 是否成为 Responder，以及做高亮和事件回调处理。

ScrollView 其实也是一个 View 类型的，在它的内部封装了一个 UIScrollView 用于存储内容视图。ScrollView 只需 contentSize 超过 bounds 便能支持滚动，不需要在 JS 端手动设置为 Responder。

在原生端有一个控制 ScrollView 滚动的方法`touchesShouldCancelInContentView`，如果返回 YES，ScrollView 将取消子视图的响应事件，用于自身消费，否则不会。

	- (BOOL)_shouldDisableScrollInteraction
	{
	  // Since this may be called on every pan, we need to make sure to only climb
	  // the hierarchy on rare occasions.
	  UIView *JSResponder = [RCTUIManager JSResponder];
	  // self.superview 代表 RCTScrollView，也就是 JS 端的 ScrollView
	  if (JSResponder && JSResponder != self.superview) {
	    BOOL superviewHasResponder = [self isDescendantOfView:JSResponder];
	    return superviewHasResponder;
	  }
	  return NO;
	}
	
	- (BOOL)touchesShouldCancelInContentView:(__unused UIView *)view
	{
	  return ![self _shouldDisableScrollInteraction];
	}

这里的实现是，如果在 JS 端当前视图层级中，最顶层的 Responder 是 ScrollView 的祖先视图`touchesShouldCancelInContentView`就返回 NO，也就是不拦截事件，这样事件的响应者会变为 ScrollView 的子视图，ScrollView 自身就无法滚动了。

这里我们进一步分析一下 UIScrollView 中 `touchesShouldCancelInContentView`方法的设计细节。

根据 UIResponder 原始的事件传递和响应机制，UIScrollView 上有一个子视图 view，如果触摸在 view 上，无论触发什么事件都会被这个 view 消费（假设 view hitTest 能返回自身），那么 UIScrollView 就没有机会滚动了。

UIScrollView 不能滚动怎么行？因此需要在 UIScrollView 实现上做点手脚，如果触摸事件在它的子视图上，并且有滚动意图，可以拦截事件，用于自身滚动，这个功能可以使用`touchesShouldCancelInContentView`方法控制。如果这个方法返回 NO，UIScrollView 不会阻断在子视图上的滑动事件，如果为 YES，检测到滚动事件时就会将其截断，将自身作为响应者。

因为 UIControl 类型的视图经常需要交互，比如 UISlider 控件需要拖动交互，所以当前滚动事件的响应者不该是 UIScrollView。因此，实际实现是 UIScrollView 检测到事件的目标响应者是 UIControl 类型时，`touchesShouldCancelInContentView`方法会返回 NO，不阻断，其他类型的响应者， UIScrollView 会将其阻断，在 hitTest 中将自身作为事件响应者返回给 UIApplication。

回到前面 RN ScrollView 的问题，其实个人不是很明白为什么要这么设计。不过从注释上看，是希望 ScrollView 上 JS Responder 子视图的事件不被 ScrollView 阻断，达到与 UIControl 一样的效果。

然而 RN 的代码实现是用`[self isDescendantOfView:JSResponder]`检测 ScrollView 是不是 JSResponder 的后代视图，如果是，会将 ScrollView 的子视图作为滚动事件的响应者，导致自身无法滚动。其实没看明白这么做的目的是什么，按照注释的话，应该是检测 ScrollView 子视图是不是 JSResponder 才对的。

对于开头说的导致无法滚动的问题，其实解决办法也简单，手动将 ScrollView 的子视图设置为  JS Responder 就行了。

## Share
在知乎上看到一篇文章，讲技术的价值，让我对程序员的价值，以及个人的学习方式有了些新的认识。

技术是否有价值是需要前提条件的，如果当前没有需求和市场，再牛逼的技术也没有价值。某项技术的价值是由认可的人数决定，而不是说技术越难越有价值。

有时候自己虽然看似在学，但进步却甚微，本质是没有让学习成果产生价值，学完就放在脑子里，以为就掌握了。其实经不住时间的考验，只有用于实践，做输出产生价值，才能算没有白学，并且实践也能使理解更加深刻。

在决定学习一项新技能时，要尽可能思考它对当前的工作是否有价值，或者能否产出对他人有帮助的代码或文章，而不只是自嗨。

根据个人的经验，能否顺利产出会受到完美度要求的影响，总是希望给出最好的结果，不停的打磨，这种精神虽然好，但最终很可能把自己的性子磨没了，结果就是不了了之。

如果希望学习一项技术，应该对这项技术在什么时候做价值输出有一个预估。比如说学了之后写文章做分享，需要在截止时间内发出，它可以是不完美的。

就像爱因斯坦小板凳的故事，板凳虽然粗糙但它是板凳，而如果到截止时间只完成一条精致的腿，那就什么都不是了。

