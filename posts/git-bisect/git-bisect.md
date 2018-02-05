## 说出来你可能不信，Git 可以自动找 Bug

之前提 [bug](https://bugs.chromium.org/p/chromium/issues/detail?id=784780&can=4&q=&colspec=ID%20Pri%20M%20Stars%20ReleaseBlock%20Component%20Status%20Owner%20Summary%20OS%20Modified) 给 Chrome，注意到处理进程中有时会为 bug 贴上 `Type-Bug-Regression` 的标签，意即回归性 bug（[Regression](https://en.wikipedia.org/wiki/Software_regression)）。指该功能以前是好的，后续新功能开发或其他改动使该功能不好了。

不一会儿便看到测试人员（可能是测试吧）指出了可能存在问题的某次提交：

> 看，这个提交之前的编译版本是 work 的！应该是这次提交引起的问题。
 
然后开发陆续进行定位修复。

至于测试是是怎么定位的，我猜他的工具箱里面有法宝。


### 不好意思，我是开发

作为开发，碰到问题我们试图正面去刚。

但情形也远不是别人想象的那样：

> Bug 嘛，轻车熟路，表示写码多年已是老司机，望闻问切就知道毛病出在哪儿，甚至对于问题代码在哪个文件哪一行都大致有数。操起键盘就是一顿敲。

这么行云流水，都不带假装思考的，莫不是先前埋下的彩蛋等着测试小哥发现，然后喝咖啡去了？

所谓刚正面就是从 Bug 表象去倒追代码，回溯到问题可能的地方最后解决掉。这是正常情况，因为还有种可能就是很难定位，没有头绪的那种。这种情况注定会是一场持久战，打下来无非就是时间花了最后还是没定位，无疑让人感到沮丧，还有就是问题定位解决了但有种『我去原来是这个原因我咋早没想到』。不管怎样都不算高效。

所以，如果遇到没有头绪或者感觉需要定位很久的问题，不防，用用「二分法」去定位 Bug。


### 二分法找 Bug

所谓「二分法」就是大概知道前面哪次提交的版本是正常的，截止到昨天的，上星期的，上个月的...不管怎样，在代码仓库的提交历史中，总能找到个时间点，这个 Bug 是不存在的。


```bash
$ glog
* c25fd4a (HEAD -> master) 5
* 5bd2f5a 4 -> 我是有问题的提交
* c46f3ee 3
* 12ec265 2
* 130ac9b 1
* f0815ba 0
```

拿上面这个提交记录为例，我们在最新的提交 `5` 上面发现了问题，随便往前找到了提交 `0` 发现它是好的。 于是问题肯定出现在 0\~5 这段提交中的某次代码提交（确切来说是 1\~5，0 已经被确认是好的了）。

机智的你当然不想从 1 到 5 挨个去试，很自然会想到找个中间值去试，将这些提交一分为二成两部分，这里我们假设选择 2 这个提交。

一试发现是好的。说明 0\~2 这一段提交都是好的。说明问题出在后续的提交中 3\~5。

二分大法果然好用，一下子将嫌疑提交减少了约一半。重复前面的步骤，将 3\~5 这段提交再一分为二，找到中间的提交 4。一试发现问题还有，说明 4 这个提交有毛病！

到这里就结束了么，没有。只因我们是上帝视角，前面告诉过你 4 这个提交就是真凶。此刻我们只知道 4,5 都是坏的，还需要验证一下 3 这个提交是否正常，只有把 3 验证了，才能唯一确定 4 是否就是问题源起的地方。

相当于在第二轮的二分中，我们确定了 4\~5 有问题，剩下 3 这个提交。继续二分，因为只有一个提交了，分不分都是它。

测试后发现它是没问题的，于是确定从 4 这个提交开始出现了问题。于是我们就可以开心地 `git checkout 4` 来 diff 出问题了。


二分的过程是在将定位问题的范围逐步缩小的过程。这一过程中，我们通过不断将范围一分为二，最后只试了三次将问题定位（2,4,3）。相比正常情况从 1 试到 5 确实省了不少工作量。如果一开始范围很大，提交数很多的情况下，这省下来的工作量就很可观了。


### 是时候祭出 Git 自带的神器了

说出来你可能不信，Git 内置了一个命令 `git bisect`，便是干上面这件事情的。

人工去选择中间的提交，然后再 `git checkout` 到相应的提交，无疑有点繁琐。

既然思路已经明朗，这些机械的事情完全可以交给工具去做嘛。而人需要做的就是，在 Git 将代码切到相应提交后，确认问题是否存在，然后告诉 Git 以便它进行下一步操作。

`man git-bisect` 可查看该命令详细说明，但就这里而言，只需掌握以下的命令便够我们使用了：

- `git bisect start` 表示进入二分查找状态
- `git bisect good [<rev>]` 这里 `<rev>` 可以是某次提交的 HASH 值，也可以是分支名，Tag 名等能唯一标识一次提交的信息，该命令将标记指定的提交为无问题
- `git bisect bad [<rev>]` 该命令标记提交为有问题
- `git bisect reset` 退出二分查找状态


假设我们现在处于 5 这个提交的位置，将前面查找问题的步骤用 `bisect` 命令走一遍会是如下的样子：

- `git bisect start` 以开始二分查找

![git bisect start](https://raw.githubusercontent.com/wayou/wayou.github.io/master/posts/git-bisect/assets/0-start-bisect.png)


- `git bisect bad` 标记当前位置为 `bad` 告诉 Git 这里的提交是有问题的

![标记最近的一次问题提交](https://raw.githubusercontent.com/wayou/wayou.github.io/master/posts/git-bisect/assets/1-mark-bad.png)


- 在提交历史中回溯，找出一个没有问题的提交，并且告诉 Git 从这个提交开始查找。

![标记查找的起点](https://raw.githubusercontent.com/wayou/wayou.github.io/master/posts/git-bisect/assets/2-mark-1st-good.png)

- 该命令执行后，我们的查找范围就确定了，Git 自动找出了一个中间位置的提交，将代码切到了该提交的位置。


- 在这个新的提交位置上，我们编译代码验证问题是否存在，然后也是通过标记 `good` 或 `bad` 来告诉 Git 该次提交是否正常。

![在新的提交位置验证并标记问题是否仍然存在](https://raw.githubusercontent.com/wayou/wayou.github.io/master/posts/git-bisect/assets/3-mark-2nd-good.png)

- 本次标记完成后，查找还没结束，于是 Git 又自动将我们定位到了一个新的位置。


- 这一步我们来到了提交信息为 `4` 的这次提交，重复前面的操作，验证问题是否存在并且进行标记。这里标记为 `bad` 表示 bug 出现在了这次提交。

![标记 bug 出现](https://raw.githubusercontent.com/wayou/wayou.github.io/master/posts/git-bisect/assets/4-mark-3rd-bad.png)

- 这次标记之后，如前文所分析，查找还没有结束，还剩下一个提交 3 需要标记确认，于是 Git 自动将我们切到了提交信息为 3 的提交。


- 标记 3 这个提交为 `good`

![标记 3 为 good](https://raw.githubusercontent.com/wayou/wayou.github.io/master/posts/git-bisect/assets/5-mark-4th-good.png)

- 这次标记完成后，查找结束，问题已经定位。Git 会显示出定位的结果，如图中展示的那样。剩下就是去修复 Bug 了。

- 最后别忘了通过 `git bisect reset ` 重置一下，退出 Git 的二分查找状态。

![退出十分查找](https://raw.githubusercontent.com/wayou/wayou.github.io/master/posts/git-bisect/assets/6-bisect-reset.png)


整套组合拳打下来，酣畅淋漓，让我们完成了一次愉快地 Bug 发现之旅。


### 总结

善用工具，事半功倍。不知道 Chrome 的测试小哥工具箱里的法宝，是不是此物。


### 相关资源

- [How to use git bisect?
](https://stackoverflow.com/questions/4713088/how-to-use-git-bisect)
- [man git-bisect](https://git-scm.com/docs/git-bisect)