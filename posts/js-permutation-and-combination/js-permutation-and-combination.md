## 排列组合问题/Permutation and Combination


排列和组合是两个问题，因为有相似性，所以通常放一起。

所谓排列，是根顺序有关的，比如从 `1,2,3` 中取三个数出来，序列 `1,2,3` 与 `1,3,2` 是两个不同的序列。这便是排列。

而组合则与顺序无关，比如从 `1,2,3` 这一序列中取两个数出来， `1,2` 与 `2,1` 只能算一种取法，只关心元素不关心顺序。

先来看排列。


### 排列/Permutation

考察 LeetCode 上面这个排列题目：

> Given a collection of distinct numbers, return all possible permutations.
>
> For example,
> `[1,2,3]` have the following permutations:

```js
[
 [1,2,3],
 [1,3,2],
 [2,1,3],
 [2,3,1],
 [3,1,2],
 [3,2,1]
]
```

转述一下：对于给定非重复序列`[1,2,3]`，找出其所有可能的排列。相当于从 n 个元素中取出 n 个，看有多少种排列。

这样说来，就和中学数学的公式可以联系起来了。

![排列组合公式](https://raw.githubusercontent.com/wayou/wayou.github.io/master/posts/js-permutation-and-combination/assets/formula.png)

__排列组合公式__


通过上面的公式，我们可以算出 `A3,3 = 3!/(3-3)! = 3!/1 = 3x2x1 = 6`。
即一共有 6 种可能的排列，以此来验证我们后面实现的算法得到的结果是否准确。

接下来的任务是实现一个方法，找出所有排列。

```js
/**
 * @param {number[]} nums
 * @return {number[][]}
 */
var permute = function (nums) {
   // 实现
};
```

#### 问题分析

假设让人脑来解决这个问题，我们屡一下思路：

- 取出第一个元素 `1`, 剩下的 `[2,3]` 中有两种取法
- 先取 `2` 再取 `3`， 得到 `[1,2,3]`
- 先取 `3` 再取 `2`， 得到 `[1,3,2]`
- `1` 打头的取完了，考虑先取 `2`，从剩下的 `[1,3]` 中，也有两种取法
- 于是分别得到 `[2,1,3]`，`[2,3,1]`
- 最后先取 `3`，也能得到两种排列 `[3,1,2]`，`[3,2,1]`
- 最后得到完整的结果为 `[[1,2,3],[1,3,2],[2,1,3],[2,3,1],[3,1,2],[3,2,1]]`

将上面的步骤抽象到类似伪代码的表示方式，可以方便我们将问题细化，最后得到一个递归的实现思路。

假设我们已经写好这么一个方法 `permute` 了，它的功能是对于输入数组，输出其所有排列。

于是上面的思路可以描述成：


```js
permute([1,2,3]) = 1 + permute([2,3]) 
                 + 2 + permute([1,3])
                 + 3 + permute([1,2])

permute([2,3]) = 2 + permute([3])
               + 3 + permute([2])

permute([3]) = 3
```

从合体数组中取出一个，剩下的数组中进行看成新的输入。如此重复，直到最后的输入变成一个元素，一个元素的排列就是其本身。第一步得到的结果都往下传递，到达到后一个元素时，我们便会得到一条完整的结果，最后所有的结果汇总便是总的结果。

所以，我们应该有一个变量来存放最后的总结果，然后对于每次递归的终点我们有对应的变量存放单个结果。

```js

var permute = function (nums) {
    var result = [];

    function process(input/*上一次处于后剩下的元素*/, prevResult/*上一次处理后的结果*/) {
        var input = input.slice();
        if (input.length > 1) {
            for (var i = 0; i < input.length; i++) {
                //遍历输入，将其中的每个元素都取出压入一次结果中，对于剩下的元素递归调用进行同样的操作
                //...
                process(nextInput, currentResult);
            }
        } else {
            // 输入为1个元素了，表示我们寻找到了终点
            result.push(/*这里我们会得到一条结果，压入总结果中*/)
        }
    }

    process(nums);

    return result;
};
```

每次操作，当输入长度不为 1 时，说明没有递归到最终，于是我们将输入中每个元素取出来放入一个临时结果中，将这个临时结果传递给下一次操作。每次操作都会往这个结果里增加一个元素。直到进行到输入还剩下一个元素的时候，我们便会得到一个完整的结果。将所有的结果合并，便得到了全部结果。

```js
/**
 * @param {number[]} nums
 * @return {number[][]}
 */
var permute = function (nums) {
    var result = [];

	/**
	 * @param {number[]} input 输入数组
	 * @param {number[]} prevResult 上一次得到的结果
	 * @return {number[]}
	 */
    function process(input, prevResult) {
        var input = input.slice();
        if (input.length > 1) {
            for (var i = 0; i < input.length; i++) {
                var currentResult = prevResult || [];
                currentResult = currentResult.concat(input[i]);
                var nextInput = input.slice();
                nextInput.splice(i, 1);
                process(nextInput, currentResult);
            }
        } else {
            var currentResult = prevResult || [];
            var row = currentResult.concat(input);
            result.push(row);
        }
    }

    process(nums);

    return result;
};
console.log(permute([1, 2, 3]))
```

### 组合/Combination

同样地，来看这个来自 LeetCode 关于组合的题目：

> Given two integers n and k, return all possible combinations of k numbers out of 1 ... n.
>
> For example,
> If n = 4 and k = 2, a solution is:

```js
[
  [2,4],
  [3,4],
  [2,3],
  [1,2],
  [1,3],
  [1,4],
]
```

即，对于给定正整数 n，从序列 1~n 中取出 k 个数，找出所有取法。

比如从 `[1,2,3,4]` 中取 2 个数，通过组合公式我们可以得出可能的取法一共有

`C4,2 = 4!/2!(4-2)! = 4!/2!2! = (4x3x2x1)/(2x1)(2x1) = 6`

6种取法。

下面我们来实现 `combine` 函数，找出所有的组合。

思路和排列的类似，也是先将问题进行拆分，直到不能再拆。

假设从 `[1,2,3,4]` 中取 2 个表示为 `combine([1,2,3,4],2)`。那么取出 `1` 后，我们接下来需要在剩下的 `[2,3,4]` 中 1 个元素，最后达到要求的 2 个元素。而从剩下的 `[2,3,4]` 取 1 个元素可类似地表示为 `combine([2,3,4],1)`。到这里又看到了递归的影子。当问题拆分到从数组中取一个元素时，就拆不动了，因为从 n 个元素中取 1 个元素，有 n 种取法，无需再拆分。

```js
combine([1,2,3,4],2) = 1 + combine([2,3,4],1)
                 + 2 + permute([3,4],1)
                 + 3 + permute([4],1)

permute([2,3,4],1) = 2 + 3 + 4
```

稍加调试将上面的思路转成代码我们得到：

```js
/**
 * @param {number} n
 * @param {number} k
 * @return {number[][]}
 */
var combine = function (n, k) {
    var result = [];

    /**
     * @param {*} i 序号
     * @param {*} n 总数
     * @param {*} k 要取的个数
     * @param {*} a 上一次的结果
     */
    function process(i, n, k, a) {
        i = i || 1;
        a = a || [];
        if (k > 1) {
            for (var i = i; i < n + 1; i++) {
                var row = [];
                row = row.concat(a)
                row.push(i);
                process(i + 1, n, k - 1, row);
            }
        } else {
            for (var i = i; i < n + 1; i++) {
                var row = a.slice();
                row.push(i)
                result.push(row)
            }
        }
    }
    process(1, n, k);
    return result;
};

console.log(combine(4, 2))
```


### 相关资源

* [LeetCode Permutations](https://leetcode.com/problems/permutations/description/)
* [LeetCode Combinations](https://leetcode.com/problems/combinations/description/)
* [Permutation & Combination - JSFiddle](http://jsfiddle.net/jinwolf/Ek4N5/29/)
* [Implement All Permutations of a Set in JavaScript](https://initjs.org/all-permutations-of-a-set-f1be174c79f8)

