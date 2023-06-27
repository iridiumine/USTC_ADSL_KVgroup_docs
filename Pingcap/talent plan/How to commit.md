## 了解 TiDB

参与一个开源项目第一步总是了解它，特别是对 TiDB 这样一个大型的项目，了解的难度比较高，这里列出一些相关资料，帮助 newcomers 从架构设计到工程实现细节都能有所了解：

- [Overview](https://github.com/pingcap/docs#tidb-introduction)
- [How we build TiDB](https://pingcap.com/blog/2016-10-17-how-we-build-tidb/)
- [TiDB 源码阅读系列文章](https://pingcap.com/blog-cn/#%E6%BA%90%E7%A0%81%E9%98%85%E8%AF%BB)

## 一个典型的开源项目是由什么组成的

### The Community（社区）

- 一个项目经常会有一个围绕着它的社区，这个社区由各个承担不同角色的用户组成。
    
- **项目的拥有者**：在他们账号中创建项目并拥有它的用户或者组织。
    
- **维护者和合作者**：主要做项目相关的工作和推动项目发展，通常情况下拥有者和维护者是同一个人，他们拥有仓库的写入权限。
    
- **贡献者**：发起拉取请求（pull request）并且被合并到项目里面的人。
    
- **社区成员**：对项目非常关心，并且在关于项目的特性以及 pull requests 的讨论中非常活跃的人。
    

### The Docs（文档）

项目中经常出现的文件有：

- **Readme**：几乎所有的 Github 项目都包含一个 README.md 文件，readme 文件提供了一些项目的详细信息，包括如何使用，如何构建。有时候也会告诉你如何成为贡献者。
    
    - [TiDB README](https://github.com/pingcap/tidb/blob/master/README.md)
- **Contributing**：项目以及项目的维护者各式各样，所以参与贡献的最佳方式也不尽相同。如果你想成为贡献者的话，那么你要先阅读那些有 CONTRIBUTING 标签的文档。Contributing 文档会详细介绍了项目的维护者希望得到哪些补丁或者是新增的特性。
    
    文件里也可以包含需要写哪些测试，代码风格，或者是哪些地方需要增加补丁之类的内容。
    
    - [TiDB Contributing 文档](https://github.com/pingcap/tidb/blob/master/CONTRIBUTING.md)
- **License**：LICENSE 文件就是这个开源项目的许可证。一个开源项目会告知用户他们可以做什么，不可做什么(比如：使用，修改，重新分发)，以及贡献者允许其他人做哪些事。开源许可证有多种，你可以在 [认识各种开源协议及其关系](http://www.ruanyifeng.com/blog/2011/05/how_to_choose_free_software_licenses.html) 了解更多关于开源许可证的信息。
    
    - TiDB 遵循 [Apache-2.0 License](https://github.com/pingcap/tidb/blob/master/LICENSE)
    - TiKV 遵循 [Apache-2.0 License](https://github.com/pingcap/tikv/blob/master/LICENSE)
- **Documentation**：许多大型项目不会只通过自述文件去引导用户如何使用。在这些项目中你经常可以找到通往其他文件的超链接，或者是在仓库中找到一个叫做 docs 的文件夹.
    
- [TiDB Docs](https://github.com/pingcap/tidb/tree/master/docs)

当然，最高效地熟悉 TiDB 的方式还是使用它，在某些场景下遇到了问题或者是想要新的 feature，去跟踪代码，找到相关的代码逻辑，在这个过程中很容易对相关模块有了解，不少 Contributor 就是这样完成了第一次贡献。 

我们还有一系列的 Infra Meetup，大约两周一次，如果方便到现场的同学可以听到这些高质量的 Talk。除了北京之外，其他的城市（上海、广州、成都、杭州）也开始组织 Meetup，方便更多的同学到现场来面基。

## 发现可以参与的事情

对 TiDB 有基本的了解之后，就可以选一个入手点。在 TiDB repo 中我们给一些简单的 issue 标记了 [for-new-contributors](https://github.com/pingcap/tidb/issues?q=is%3Aissue+is%3Aopen+label%3A%22for+new+contributors%22)  标签，这些 issue 都是我们评估过容易上手的事情，可以以此为切入点。另外我们也会定期举行一些活动，把框架搭好，教程写好，新 Contributor 按照固定的模式即可完成某一特性开发。

当然除了那些标记为 for-new-contributors 的 issue 之外，也可以考虑其他的 issue，标记为  [help-wanted](https://github.com/pingcap/tidb/issues?q=is%3Aissue+is%3Aopen+label%3A%22help+wanted%22)  标签的 issue 可以优先考虑。除此之外的 issue 可能会比较难解决，需要对 TiDB 有较深入的了解或者是对完成时间有较高的要求，不适合第一次参与的同学。

当然除了现有的 issue 之外，也欢迎将自己发现的问题或者是想要的特性提为新的 issue，然后自投自抢 :) 。 

当你已经对 TiDB 有了深入的了解，那么可以尝试从  [Roadmap](https://pingcap.com/docs-cn/v3.0/roadmap/)  上找到感兴趣的事项，和社区讨论一下如何参与。

## 讨论方案

找到一个感兴趣的点之后，可以在 issue 中进行讨论，如果是一个小的 bug-fix 或者是小的功能点，可以简单讨论之后开工。即使再简单的问题，也建议先进行讨论，以免出现解决方案有问题或者是对问题的理解出了偏差，做了无用功。 更多关于 issue 如何工作的信息，请点击 [Issues guide](http://guides.github.com/features/issues) 。

#### Issues Pro Tips

- **检查你的问题是否已经存在** 重复的问题会浪费大家的时间，所以请先搜索打开和已经关闭的问题，来确认你的问题是否已经提交过了。
    
- **清楚描述你的问题**
    
    TiDB Issue 模版如下：
    
    ![TiDB Issue 模版](https://img1.www.pingcap.com/prod/2_2522aff338.png)
    
    TiDB Issue 模版
    
    TiKV Issue 模版如下：
    
    ![TiKV Issue 模版](https://img1.www.pingcap.com/prod/3_eb48a0d369.png)
    
    TiKV Issue 模版
    
- **给出你的代码链接** 使用像 [JSFiddle](http://jsfiddle.net/) 或者 [CodePen](http://codepen.io/) 等工具，贴出你的代码，好帮助别人复现你的问题。
    
- **详细的系统环境介绍** 例如使用什么版本的浏览器，什么版本的库，什么版本的操作系统等其他你运行环境的介绍。
    
    ```
    go 版本： go version
    Linux 版本： uname -a
    ```
    
- **详细的错误输出或者日志** 使用 [Gist](http://gist.github.com/) 贴出你的错误日志。如果你在 issue 中附带错误日志，请使用 ``` 来标记你的日志。以便更好的显示。

但是如果要做的事情比较大，可以先写一个详细的设计文档，提交到  [docs/design](https://github.com/pingcap/tidb/tree/master/docs/design)  目录下面，这个目录下有设计模板以及一些已有的设计方案供你参考。一篇好的设计方案要写清楚以下几点：

- 背景知识
    
- 解决什么问题
    
- 方案详细设计
    
- 对方案的解释说明，证明正确性和可行性
    
- 和现有系统的兼容性
    
- 方案的具体实现 
    
用一句话来总结就是写清楚“你做了什么，为什么要做这个，怎么做的，为什么要这样做”。如果对自己的方案不太确定，可以先写一个 Google Doc，share 给我们简单评估一下，再提交 PR。

## 提交 PR

- **[Fork](http://guides.github.com/activities/forking/) 代码并且 clone 到你本地** 通过将项目的地址添加为一个 remote，并且经常从 remote 合并更改来保持你的代码最新，以便在提交你的 pull 请求时，尽可能少的发生冲突。详情请参阅 [Syncing a fork](https://help.github.com/articles/syncing-a-fork) 。
    
- **创建 [branch](http://guides.github.com/introduction/flow/)** 来修改你的代码，目前 TiDB 相关的项目默认的 branch 命名规则是 user/name。例如 disksing/grpc，简单明确，一目了然。
    
- **描述清楚你的问题**方便其他人能够复现。或者说明你添加的功能有什么作用，并且清楚描述你做了哪些更改。
    
- **注意测试** 如果项目中包含逻辑修改，那么必须包含相应的测试，在 CI 中会包含测试覆盖率的检测，如果测试覆盖率下降，那么是不可以合并到 master 的。
    
- **包含截图** 如果您的更改包含 HTML/CSS 中的差异，请添加前后的屏幕截图。将图像拖放到您的 pull request 的正文中。
    
- **保持良好的代码风格**这意味着使用与你自己的代码风格中不同的缩进，分号或注释，但是使维护者更容易合并，其他人将来更容易理解和维护。目前 TiDB 项目的 CI 检测包含代码风格的检查，如果代码风格不符合要求，那么是不可以合并到 master 的。
    

### Open Pull Requests

一旦你新增一个 pull request，讨论将围绕你的更改开始。其他贡献者和用户可能会进入讨论，但最终决定是由维护者决定的。你可能会被要求对你的 pull request 进行一些更改，如果是这样，请向你的 branch 添加更多代码并推送它们，它们将自动进入现有的 pull request。

![Open Pull Requests](https://img1.www.pingcap.com/prod/4_86f1bc587c.png)

Open Pull Requests

如果你的 pull request 被合并，这会非常棒。如果没有被合并，不要灰心。也许你的更改不是项目维护者需要的。或者更改已经存在了。发生这种情况时，我们建议你根据收到的任何反馈来修改代码，并再次提出 pull request。或创建自己的开源项目。

### TiDB 合并流程

PR 提交之后，请耐心等待维护者进行 Review。

目前一般在一到两个工作日内都会进行 Review，如果当前的 PR 堆积数量较多可能回复会比较慢。 代码提交后 CI 会执行我们内部的测试，你需要保证所有的单元测试是可以通过的。期间可能有其它的提交会与当前 PR 冲突，这时需要修复冲突。

维护者在 Review 过程中可能会提出一些修改意见。修改完成之后如果 reviewer 认为没问题了，你会收到 LGTM（looks good to me）的回复。当收到两个及以上的 LGTM 后，该 PR 将会被合并。

> 标注：GitHub Guides：如何参与一个 GitHub 开源项目 [英文原文](https://guides.github.com/activities/contributing-to-open-source/) 。

## PR review

PR review 的过程就是 reviewer 不断地提出 comment，PR 作者持续解决 comment 的过程。

**每个 PR 在合并之前都需要至少得到两个 Committer/Maintainer 的 LGTM，一些重要的 PR 需要得到三个，比如对于 DDL 模块的修改，默认都需要得到三个 LGTM。**

#### Tips：

- 提了PR 之后，可以 at 一下相关的同学来 review；
    
- Address comment 之后可以 at 一下之前提过 comment 的同学，标准做法是 comment 一下 “**PTAL @xxx**”，这样我们内部的 Slack 中可以得到通知，相关的同学会受到提醒，让整个流程更紧凑高效。
    

## 与项目维护者之间的交流

**目前标准的交流渠道是 GitHub issue**，请大家优先使用这个渠道，我们有专门的同学来维护这个渠道，其他渠道不能保证得到研发同学的及时回复。这也是开源项目的标准做法。

无论是遇到 bug、讨论具体某一功能如何做、提一些建议、产品使用中的疑惑，都可以来提 issue。在开发过程中遇到了问题，也可以在相关的 issue 中进行讨论，包括方案的设计、具体实现过程中遇到的问题等等。

**最后请大家注意一点，除了 pingcap/docs-cn 这个 repo 之外，请大家使用英文。**

## 更进一步

当你完成上面这些步骤的之后，恭喜你已经跨过第一个门槛，正式进入了 TiDB 开源社区，开始参与 TiDB 项目开发，成为 TiDB Contributor。

如果想更进一步，深入了解 TiDB 的内部机制，掌握一个分布式数据库的核心模块，并能做出改进，那么可以了解更多的模块，提更多的 PR，进一步向 Committer 发展（ [Become a Committer](https://github.com/pingcap/community/blob/master/become-a-committer.md)  这篇文档解释了什么是 Committer）。目前 TiDB 社区的 Committer 还非常少，我们希望今后能出现更多的 Committer 甚至是 Maintainer。

从 Contributor 到 Committer 的门槛比较高，需要对一些模块有非常深入的了解。当然，成为 Committer 之后，会有一定的权利，比如对一些 PR 点 LGTM 的权利，参加 PingCAP 内部的技术事项、开发规划讨论的权利，参加定期举办的 TechDay/DevCon 的权利。目前社区中还有几位贡献者正走在从 Contributor 到 Committer 的道路上。

# 参考文档
1. [TiDB 开发指南](https://pingcap.github.io/tidb-dev-guide/contribute-to-tidb/introduction.html)
2. [TiDB 社区交流](https://github.com/pingcap/community/blob/master/communicating.md)
3. [Good First Issue](https://github.com/pingcap/tidb/issues?q=is%3Aopen+is%3Aissue+label%3A%22good+first+issue%22)
4. [如何从零开始参与大型开源项目](https://cn.pingcap.com/blog/how-to-contribute)
5. [TiDB 开源社区指南](https://cn.pingcap.com/blog/tidb-community-guide-1)
6. [TiDB in action](https://book.tidb.io/)