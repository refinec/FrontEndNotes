# Issue

在 Issue 一览表中我们可以看到，每一个 Issue 标题的下面都分配了诸如“#24”的编号。只要在提交信息的描述中加入“#24”，在 Issue 中显示该提交的相关信息，使关联的提交一目了然。这里只需轻轻点击一下便可以显示相应提交的具体内容，在代码审查时省去了从大量提交日志中搜索相应提交的麻烦，非常方便.

如果一个处于 Open 状态的 Issue 已经处理完毕，只要在该提交中以下列任意一种格式描述提交信息，对应的 Issue 就会被 Close。

- fix     #24
- fixes     #24
- fixed     #24
- close     #24
- closes     #24
- closed     #24
- resolve     #24
- resolves     #24
- resolved     #24

利用这个方法，每次提交并 push 之后，就不必再大费周章地到 GitHub 的 Issue 中寻找相应 Issue 再手动 Close，省去不少麻烦。

在 GitHub 上，如果给 Issue 添加源代码，它就会变成 Pull Request。Issue 与 Pull Request 的编号相互通用，通过 GitHub 的 API 可以将特定的 Issue 转换为 Pull Request。