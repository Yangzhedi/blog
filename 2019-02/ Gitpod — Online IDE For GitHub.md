原文作者：[Sven Efftinge](https://medium.com/@sven.efftinge)

原文链接：https://medium.com/gitpod/gitpod-gitpod-online-ide-for-github-6296b907a886



经过一年多的密集工作，我们很自豪地宣布[Gitpod](http://gitpod.io/)的公测版。

Gitpod是一个在线IDE，可以从任何GitHub页面启动。只需**在任何GitHub-URL前加上“https：//gitpod.io#”**，或使用我们的[浏览器扩展](https://chrome.google.com/webstore/detail/gitpod-online-ide/dodmmooeoklaejobgleioelladacbeki?hl=en)为GitHub页面添加一个按钮。

在几秒钟之内，Gitpod就可以为您提供一个完整的开发环境，包括一个VS Code驱动的IDE和一个可以由项目定制化配置的云Linux容器。

配套视频已上传bilibili：https://www.bilibili.com/video/av42010571

<iframe src="//player.bilibili.com/player.html?aid=42010571&cid=73754969&page=1" scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true"> </iframe>

### 又一个在线IDE？

Gitpod不是另一个在线IDE，旨在取代桌面开发。 相反，Gitpod是GitHub的自然扩展。GitHub的编辑功能非常有限，迫使我们经常要切换到本地计算机。**Gitpod可以延续我们在GitHub的“生活”。**

此外，Gitpod非常简单：您不需要使用很大的存储空间和复杂的控制台来维护您的项目或工作区，而是在GitHub上安全的存储和版本化您的项目配置。

### 保持在GitHub流程中

Gitpod是高度依赖上下文的，因此它根据上下文以正确的模式打开IDE：

如果您正在查看GitHub上某个特定提交的特定文件，此时启动Gitpod工作区将检测出正确的版本并打开文件。

从issue启动Gitpod工作区将会自动创建分支并预配置提交消息。

**从pull request启动Gitpod工作区将您的权限转换为`code review `模式**。

### **集成Github**

进入IDE后，您可以通过各种方式与GitHub进行交互。 除了明显的Git集成之外，您还可以执行诸如**在编辑器中内联注释，批准甚至合并PR之类的操作**。

![Inline Comments For Pull Requests](https://user-gold-cdn.xitu.io/2019/2/2/168acff57e373cf4?w=1000&h=574&f=png&s=219702)

### 自动安装

Gitpod基于Kubernetes构建，您可以将任何Docker镜像用于您的开发环境。 这就赋予了每个人都可以完全自动化配置的能力，避免了手动浏览冗长和过时的设置文档。不会再出现“我的电脑是ok的”场景。

Gitpod可以为您的协作者提供**一键贡献体验**。

更多配置Gitpod的信息，请访问[此处](http://docs.gitpod.io/40_Configuration.html)。

### 用完即走

Gitpod工作区是一次性的。 您只需在需要时创建一个新的即可。 完成操作后，IDE会将您带回GitHub，以便继续之后的操作。

您通常不需要回到任何工作空间，在您需要时使用即可。

### 完整终端功能

Gitpod为开发人员提供了全功能的终端来运行任何过程，例如编译，linting或对你的应用进行一些测试。任何docker镜像都可以工作，您甚至可以将Gitpod配置为在启动时自启动某些任务。

![ ](https://user-gold-cdn.xitu.io/2019/2/2/168acff57e25baab?w=1000&h=575&f=png&s=261200)

### 开源

Gitpod中的IDE基于[Theia](http://theia-ide.org/)，这是一个开源项目，早在2017年初我们（TypeFox）就与爱立信的朋友一起开发。您也可以将其视为**VS Code的在线版本**。我们喜欢VS Code，但需要一些额外的属性，比如更具可扩展性的架构以及在连接到远程后端的浏览器中运行的能力。Theia是一个真正的开源项目，由Eclipse Foundation托管，由TypeFox，Ericsson，Red Hat，Arm等的各类工程师开发。

![Language Tooling In Action](https://user-gold-cdn.xitu.io/2019/2/2/168acff57e5e1a7a?w=1000&h=575&f=png&s=204298)

### 多语言支持

Theia基于VS Code及其语言服务协议，支持大多数主流编程语言。

 下表提供了现状态的良好概述。

![123](https://user-gold-cdn.xitu.io/2019/2/2/168acff58404b565?w=1000&h=418&f=png&s=39356)

其他语言如**C＃，Swift，Clojure，Groovy，Objective-C，Markdown，Less，XML**以及许多其他语言也支持语法着色。因为为Theia创建扩展程序非常容易，所以语法支持将很快更具广度（更多语言）和深度。

### 免费使用！

您可以将Gitpod与任何GitHub库一起使用。 登录是通过GitHub OAuth完成的。 起初，Gitpod只会要求访问公共代码库。 如果您想将它与私有库一起使用，Gitpod将再次请求获得更多权限。

现在就试试吧，如果您还没有自己的代码库，可以选择下面的几个试试：

- **JavaScript**:
  [https://gitpod.io#**https://github.com/ooade/NextSimpleStarter**](https://gitpod.io/#https://github.com/ooade/NextSimpleStarter)
- **Go**:
  [https://gitpod.io#**https://github.com/demo-apps/go-gin-app**](https://gitpod.io/#https://github.com/demo-apps/go-gin-app)
- **Java**:
  [https://gitpod.io#**https://github.com/spring-guides/gs-spring-boot**](https://gitpod.io/#https://github.com/spring-guides/gs-spring-boot)
- **Ruby**:
  [https://gitpod.io#**https://github.com/gitpod-io/rails_sample_app**](https://gitpod.io/#https://github.com/gitpod-io/rails_sample_app)
- **Python**:
  [https://gitpod.io#**https://github.com/sibtc/django-beginners-guide**](https://gitpod.io/#https://github.com/sibtc/django-beginners-guide)
- **PHP**:
  [https://gitpod.io#**https://github.com/symfony/demo**](https://gitpod.io/#https://github.com/symfony/demo)

### 路在何方？路在脚下

很多新功能已经可以使用了，例如Git的集成和搜索功能。 而且还有很多正在开发的令人兴奋的新功能，例如**调试，协作和支持GitLab和Bitbucket**，甚至即将推出**对VS代码扩展的支持**。

除了更多功能外，我们还专注于通过改善小缺点，修复bug和提高性能来改善整体体验。 新版本将不断推出。

如果您有反馈或发现错误，请在[此处](https://github.com/gitpod-io/gitpod/issues)提交。

**Happy coding!**