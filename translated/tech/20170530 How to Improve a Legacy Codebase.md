# 如何改善遗留的代码库

这在每一个程序员，项目管理员，团队领导的一生中都会至少发生一次。原来的程序员早已离职去度假了，留下了一坨几百万行屎一样的代码和文档（如果有的话），一旦接手这些代码，想要跟上公司的进度简直让人绝望。

你的工作是带领团队摆脱这个混乱的局面

当你的第一反应过去之后，你开始去熟悉这个项目，公司的管理层都在关注着你，所以项目只能成功，然而，看了一遍代码之后却发现很大的可能会失败。那么该怎么办呢？

幸运（不幸）的是我已经遇到好几次这种情况了，我和我的小伙伴发现将这坨热气腾腾的屎变成一个健康可维护的项目是非常值得一试的。下面这些是我们的一些经验：

### 备份

在开始做任何事情之前备份与之可能相关的所有文件。这样可以确保不会丢失任何可能会在另外一些地方很重要的信息。一旦修改其中一些文件，你可能花费一天或者更多天都解决不了这个愚蠢的问题，配置数据通常不受版本控制，所以特别容易受到这方面影响，如果定期备份数据时连带着它一起备份了，还是比较幸运的。所以谨慎总比后悔好，复制所有东西到一个绝对安全的地方吧，除非这些文件是只读模式否则不要轻易碰它。

### 必须确保代码能够在生产环境下构建运行并产出，这是重要的先决条件。

之前我假设环境已经存在，所以完全丢了这一步，Hacker News 的众多网友指出了这一点并且证明他们是对的：第一步是确认你知道在生产环境下运行着什么东西，也意味着你需要在你的设备上构建一个跟生产环境上运行的版本每一个字节都一模一样的版本。如果你找不到实现它的办法，一旦你将它投入生产环境，你很可能会遭遇一些很糟糕的事情。确保每一部分都尽力测试，之后在你足够信任它能够很好的运行的时候将它部署生产环境下。无论它运行的怎么样都要做好能够马上切换回旧版本的准备，确保日志记录下了所有情况，以便于接下来不可避免的 “验尸” 。

### 冻结数据库

直到你修改代码之前尽可能冻结你的数据库，在你特别熟悉代码库和遗留代码之后再去修改数据库。在这之前过早的修改数据库的话，你可能会碰到大问题，你会失去让新旧代码和数据库一起构建稳固的基础的能力。保持数据库完全不变，就能比较新的逻辑代码和旧的逻辑代码运行的结果，比较的结果应该跟预期的没有差别。

### 写测试

在你做任何改变之前，尽可能多的写下端到端测试和集成测试。在你能够清晰的知道旧的是如何工作的情况下确保这些测试能够正确的输出（准备好应对一些突发状况）。这些测试有两个重要的作用，其一，他们能够在早期帮助你抛弃一些错误观念，其二，在你写新代码替换旧代码的时候也有一定防护作用。

自动化测试，如果你也有 CI 的使用经验请使用它，并且确保在你提交代码之后能够快速的完成所有测试。

### 日志监控

如果旧设备依然可用，那么添加上监控功能。使用一个全新的数据库，为每一个你能想到的事件都添加一个简单的计数器，并且根据这些事件的名字添加一个函数增加这些计数器。用一些额外的代码实现一个带有时间戳的事件日志，这是一个好办法知道有多少事件导致了另外一些种类的事件。例如：用户打开 APP ，用户关闭 APP 。如果这两个事件导致后端调用的数量维持长时间的不同，这个数量差就是当前打开的 APP 的数量。如果你发现打开 APP 比关闭 APP 多的时候，你就必须要知道是什么原因导致 APP 关闭了（例如崩溃）。你会发现每一个事件都跟其他的一些事件有许多不同种类的联系，通常情况下你应该尽量维持这些固定的联系，除非在系统上有一个明显的错误。你的目标是减少那些错误的事件，尽可能多的在开始的时候通过使用计数器在调用链中降低到指定的级别。（例如：用户支付应该得到相同数量的支付回调）。

这是简单的技巧去将每一个后端应用变成一个就像真实的簿记系统一样，所有数字必须匹配，只要他们在某个地方都不会有什么问题。

随着时间的推移，这个系统在监控健康方面变得非常宝贵，而且它也是使用源码控制修改系统日志的一个好伙伴，你可以使用它确认 BUG 出现的位置，以及对多种计数器造成的影响。

我通常保持 5 分钟（一小时 12 次）记录一次计数器，如果你的应用生成了更多或者更少的事件，你应该修改这个时间间隔。所有的计数器公用一个数据表，每一个记录都只是简单的一行。

### 一次只修改一处

不要完全陷入在提高代码或者平台可用性的同时添加新特性或者是修复 BUG 的陷阱。这会让你头大而且将会使你之前建立的测试失效，现在必须问问你自己，每一步的操作想要什么样的结果。

### 修改平台

如果你决定转移你的应用到另外一个平台，最主要的是跟之前保持一样。如果你觉得你会添加更多的文档和测试，但是不要忘记这一点，所有的业务逻辑和相互依赖跟从前一样保持不变。

### 修改架构

接下来处理的是改变应用的结构（如果需要）。这一点上，你可以自由的修改高层的代码，通常是降低模块间的横向联系，这样可以降低代码活动期间对终端用户造成的影响范围。如果老代码是庞大的，那么现在正是让他模块化的时候，将大段代码分解成众多小的，不过不要把变量的名字和他的数据结构分开。

Hacker News [mannykannot][1] 网友指出，修改架构并不总是可行，如果你特别不幸的话，你可能为了改变一些架构必须付出沉重的代价。我也赞同这一点，我应该加上这一点，因此这里有一些补充。我非常想补充的是如果你修改高级代码的时候修改了一点点底层代码，那么试着限制只修改一个文件或者最坏的情况是只修改一个子系统，所以尽可能限制修改的范围。否则你可能很难调试刚才所做的更改。

### 底层代码的重构

现在，你应该非常理解每一个模块的作用了，准备做一些真正的工作吧：重构代码以提高其可维护性并且使代码做好添加新功能的准备。这很可能是项目中最消耗时间的部分，记录你所做的任何操作，在你彻底的记录模块并且理解之前不要对它做任何修改。之后你可以自由的修改变量名、函数名以及数据结构以提高代码的清晰度和统一性，然后请做测试（情况允许的话，包括单元测试）。

### 修复 bugs

现在准备做一些用户可见的修改，战斗的第一步是修复很多积累了一整年的bugs，像往常一样，首先证实 bug 仍然存在，然后编写测试并修复这个 bug，你的 CI 和端对端测试应该能避免一些由于不太熟悉或者一些额外的事情而犯的错误。

### 升级数据库


如果在一个坚实且可维护的代码库上完成所有工作，如果你有更改数据库模式的计划，可以使用不同的完全替换数据库。
把所有的这些都做完将能够帮助你更可靠的修改而不会碰到问题，你会完全的测试新数据库和新代码，所有测试可以确保你顺利的迁移。

### 按着路线图执行

祝贺你脱离的困境并且可以准备添加新功能了。

### 任何时候都不要尝试彻底重写

彻底重写是那种注定会失败的项目，一方面，你在一个未知的领域开始，所以你甚至不知道构建什么，另一方面，你会把所以的问题都推到新系统马上就要上线的前一天，非常不幸的是，这也是你失败的时候，假设业务逻辑存在问题，你会得到异样的眼光，那时您会突然明白为什么旧系统会用某种奇怪的方式来工作，最终也会意识到能将旧系统放在一起工作的人也不都是白痴。在那之后。如果你真的想破坏公司（和你自己的声誉），那就重写吧，但如果你足够聪明，彻底重写系统通常不会成为一个摆到桌上讨论的选项。

### 所以，替代方法是增量迭代工作

要解开这些线团最快方法是，使用你熟悉的代码中任何的元素（它可能是外部的，他可以是内核模块），试着使用旧的上下文去增量提升，如果旧的构建工具已经不能用了，你将必须使用一些技巧（看下面）至少当你开始做修改的时候，试着尽力保留已知的工作。那样随着代码库的提升你也对代码的作用更加理解。一个典型的代码提交应该最多两行。

### 发布！

每一次的修改都发布到生产环境，即使一些修改不是用户可见的。使用最少的步骤也是很重要的，因为当你缺乏对系统的了解时，只有生产环境能够告诉你问题在哪里，如果你只做了一个很小的修改之后出了问题，会有一些好处：

*   很容易弄清楚出了什么问题
*   这是一个改进流程的好位置
*   你应该马上更新文档展示你的新见解

### 使用代理的好处
如果你做 web 开发时在旧系统和用户之间加了代理。你能很容易的控制每一个网址哪些请求旧系统，哪些重定向到新系统，从而更轻松更精确的控制运行的内容以及谁能够看到。如果你的代理足够的聪明，你可以使用它发送一定比例的流量到个人的 URL，直到你满意为止，如果你的集成测试也连接到这个接口那就更好了。

### 是的，这会花费很多时间
这取决于你怎样看待它的，这是事实会有一些重复的工作涉及到这些步骤中。但是它确实有效，对于进程的任何一个优化都将使你对这样系统更加熟悉。我会保持声誉，并且我真的不喜欢在工作期间有负面的意外。如果运气好的话，公司系统已经出现问题，而且可能会影响客户。在这样的情况下，如果你更多地是牛仔的做事方式，并且你的老板同意可以接受冒更大的风险，我比较喜欢完全控制整个流程得到好的结果而不是节省两天或者一星期，但是大多数公司宁愿采取稍微慢一点但更确定的胜利之路。

--------------------------------------------------------------------------------

via: https://jacquesmattheij.com/improving-a-legacy-codebase

作者：[Jacques Mattheij][a]
译者：[aiwhj](https://github.com/aiwhj)
校对：[校对者ID](https://github.com/校对者ID)

本文由 [LCTT](https://github.com/LCTT/TranslateProject) 原创编译，[Linux中国](https://linux.cn/) 荣誉推出

[a]:https://jacquesmattheij.com/
[1]:https://news.ycombinator.com/item?id=14445661