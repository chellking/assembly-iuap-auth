
# 代码与协作管理

代码最早是svn进行管理的，但在快速开发过程中分支管理一直是一个头疼的问题，前东家使用的商业代码版本管理软件，分支管理也是非常头疼，以至于最后都不敢建立分支。同时svn在code review上也是无能无力，必须还需要一个专门的code review平台来进行管理，对于小团队来说，有点得不偿失。因此在这样的情况下，我们选择了Git，并使用gitlab进行管理，好处是：
代码分支管理非常方便，通过一系列规范，分支管理非常清晰易掌握。
code review可以很方便进行，并且还是强制性的进行。代码评审对于项目质量来说非常关键，这方面吃了很多亏。
各种消息通知可以很容易进行邮件通知，做到及时性。
注：代码管理参考leancloud的管理方法，网址是：链接，在此基础上进行小部分修改。

## 功能驱动（FDD）

目前各种git流程管理都是基于FDD（feature-driven development）功能驱动模型进行开发，意思就是现有需求，基于需求建立功能分支（feature branch）或者问题修复分支（bug-fix branch），开发完毕后合并到主分支后，将功能分支进行删除。
我们的主流程也是基于此模型进行。

## 分支管理（branch）

前面讲述，我们目前积极推广微服务，将每一个模块作为一个项目来对待，那么一个模块在git里就是一个repo（仓库），那么在一个模块内进行并行开发的机会就很少，降低建立分支的概率。那么长期存在的分支只有两个：master和release分支，master是建立仓库时默认建立的，是开发分支；release分支是发布分支，主要用于build项目和tag管理。
在repo建立时，确定好人员管理分支的权限，正常一个仓库应该有一个master的权限，其他开发同学是developer权限，developer权限用于开发项目，但不能合并feature分支代码进master分支。
开发人员在开发时，首先对于需求根据功能点进行划分，每一个功能点最好控制在3天内开发完毕，然后对每一个功能点编写issue，在issue里对功能点进行说明，方便后面的code review，然后根据issue建立分支，分支名字为：feature-功能点名称-issue编号，从master分支同步代码。开发人员在开发完成代码后，进行单元测试，测试完毕后，和master分支代码同步，然后进行提交，同时在gitlab里进行merge request（常规名字是pull request），请求合并和代码评审，并写明需要参加评审的人员。此时评审人员就会收到需要评审邮件，通过邮件链接直接进去gitlab进行代码审查，代码审查没有问题后，进行合并代码和删除功能分支。
每一个功能点设置在3天，也是为了减低功能点规模，降低代码冲突的风险，同时加快代码评审的速度，把问题控制在早期，正常情况下，在下班前进行pull request，第二天早晨评审人员来了后首先进行代码评审，时间控制在半小时内。

### 第一步：新建分支

首先根据issue建立新功能分支
git checkout master
git pull
git checkout -b feature-x-issueNO master

### 第二步：开发代码和提交分支

在代码开发完毕后，可以提交分支：
git add -all
git status
git commit -m "comment"
其中注释要写明此次提交的信息说明，建议格式如下：
简要描述此次提交的内容
第一条修改内容
第二条修改内容
关联的issue编号

### 第三步：与主干同步

在分支开发完成后，push之前，要与master进行同步
git fetech origin
git rebase -i origin/master
这里最后一步是将多次commit合成一个commit，方便后期管理。在合并过程中，需要将第一个之外的pick改成s。

### 第四步：推送到远程仓库

合并完代码后，就需要把功能开发分支合并到远程仓库
git push --force origin feature-x-issueNO
git push 命令带上--force命令，是因为rebase可能改变了分支的历史，需要强制推送。

### 第五步：发出pull request

推送完代码后，在gitlab后就会出现merge request的提示，这时就要进行pull request，写上注释，写明评审人和合并分支。截图如下：

### 第六步：code review

第五步完成后，评审人就会收到邮件，直接在gitlab进行查看分支合并历史，对代码进行评审，如果评审通过，进行代码合并并删除分支。如果评审不通过，写明原因，退回开发人员。

### 第七步：生产部署

在代码测试完毕后，先将release分支删除，重新建立release分支，从master分支同步代码。然后根据release分支进行build代码，部署上线成功后，打tag，服务器tag标签为上线日期，比如：2016.03.31，客户端的则根据三段式规则，比如：v3.0.1, v..
bug-fix分支

在上线后，可能出现紧急问题，如果不是阻断性问题，尽量放到下一版本中，如果必须立即修复，那么按照下列步骤进行：
如果此问题已经在master分支里解决，那么直接在release分支里cherry-pick master分支的对应的commit，如果没有转到2.
建立bug-fix分支，开发完成后发起pull request请求代码评审，评审通过后，代码合并到master分支，将bug-fix分支删除。
在release分支里cherry-pick master分支的对应的commit，部署上线成功后，打tag，如果是服务器，tag为上个版本后面加英文字母，比如，2016.03.31a，如果是客户端，那patch加1。