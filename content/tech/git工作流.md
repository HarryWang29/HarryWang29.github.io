---
title: "Git工作流"
date: 2022-05-13T22:15:31+08:00
description: "多人开发同一代码时，如何处理"
tags: 
  - "git"
---

> git工作流其实含有很多种，git/github/gitlab等等，但是大多数文章中只是描述了主要流程中的特性，并没有描述多人并行开发时注意点，本文主要说明多人并行开发时，git如何使用

## 书籍推荐

首先推荐一本书——[git官方文档](https://git-scm.com/book/zh/v2)，中文翻译+pdf下载，非常方便。

个人认为，作为普通使用者来讲，此书只需要看1/2/3/7就可以了，其他的可以作为扩展内容进行阅读，本文后续内容会有部分摘抄自此文档。

## 前置知识

开源社区基础开发git流：

![未命名文件.png](https://user-images.githubusercontent.com/8288067/187078494-e08edc42-966f-46ea-be32-ce8a28c4e046.png)

1. 从 `开源仓库`中fork到 `个人仓库`
2. clone仓库到本地（其实这里无论是从原来仓库clone，还是从自己的仓库clone都可以，主要涉及本地remote如何设置）
3. 在本地修改代码，实现需求
4. commit修改
5. push代码到 `个人仓库` 
6. 从 `个人仓库` 发起pr到 `开源仓库` 
7. 由 `开源仓库` 管理员审核后，merge 到 `开源仓库` 中
8. 若开源仓库后续发布新的版本，则从 `开源仓库` pull到本地

引申到公司代码开发：

![image](https://user-images.githubusercontent.com/8288067/187078911-625a840a-8fcd-425e-8aa5-d584f79f8c26.png)

> 最近在使用了企业版gitee后，发现评审模式下，员工对保护分支进行push时，会自动生成一个auto-xxx的pr分支（例如git push origin new-work => git push origin auto-new-work; 再从auto-new-work提交一个pr到new-work分支），观察下来发先，确实可以省去fork这一流程

## 分支

相信大家对于创建分支命令已经非常熟悉了：

```bash
git branch test
git checkout test
或者
git checkout -b test
```

在此推荐几个命令：

```bash
git branch --merged #查看所有已经合并到当前分支的分支
git branch --no-merged #查看所有包含未合并工作的分支
#详细解释可以执行如下命令查看
git branch --help

git stash      # 将当前分支的改动储藏起来
               # 当你有一些代码改动，又不想commit时，可使用此命令将变动暂存起来
git stash pop  # 将最后一起储藏起来的变动释放出来
#详细解释可以执行如下命令查看
git stash --help
```

此处我们还是查看文章 **[3.2 Git 分支 - 分支的新建与合并](https://git-scm.com/book/zh/v2/Git-分支-分支的新建与合并)**

## 工作流

在 Git Flow 中，有两个长期存在且不会被删除的分支：master 和 develop。

在这两个分支中，master 主要用于对外发布稳定的新版本，该分支时常保持着软件可以正常运行的状态，由于要维护这一状态，所以不允许开发者直接对 master 分支的代码进行修改和提交，其他分支的开发工作进展到可以发布的程度后，将会与 master 分支进行合并，并且这一合并只在发版时进行，发布时将会附加版本编号的 Git 标签。

develop 则用来存放我们最新开发的代码，这个分支是我们开发过程中代码中心分支，这个分支也不允许开发者直接进行修改和提交。程序员要以 develop 分支为起点新建 feature 分支，在 feature 分支中进行新功能的开发或者代码的修正，也就是说 develop 分支维系着开发过程中的最新代码，以便程序员创建 feature 分支进行自己的工作。

注意 develop 合并的时候，不要使用 fast-farward merge，建议加上 --no-ff 参数，这样在 master 上就会有合并记录，关于这两个的区别，大家可以参数松哥之前的 Git 教程，这里不再赘述。

除了这两个永久分支，还有三个临时分支：feature branches、hotfixes 以及 release branches。

作者：陈琰AC
链接：[https://www.jianshu.com/p/7eba1f0b5b42](https://www.jianshu.com/p/7eba1f0b5b42)
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

以上基本上都是引用现有的一些文字，现在我们看下实际开发情况：

我们假设下一个版本(v0.0.1)共有两个需求分别为feat1和feat2，现将需求分配给两个人分别为user1和user2。

问题：

按照当前的开发习惯，我们由管理员在公司仓库中，创建feat-0.0.1分支，两个人同时都在feat-0.0.1开发，拉取feat-0.0.1代码，将修改后的代码推送至feat-0.0.1，若这时user1负责的需求feat1因为一些不可抗力，被延迟一个版本甚至要求完全不上线，那么feat-0.0.1的代码已经完全搅和在一起了，我们只能让user1在feat-0.0.1中把feat1的代码自己删除，才能保证feat2正常的上线

解决：

1. user1和user2分别在本地创建feat1和feat2的分支
2. 各自在自己的分支中分别写入需求代码
3. 需求完成开发后，要进入测试
4. 在企业仓库中创建 `feat1` 和 `feat2` 分支
5. 使用常用分支 `test` ，各自将分支代码通过pr合入 `test` 分支中
6. 将 `test` 分支发布到test环境
7. 测试人员开始测试
8. 初步测试通过后，准备回归测试
9. 创建 `release-0.0.1` 分支，将 `feat1` 和 `feat2` 代码合入
10. 将  `release-0.0.1` 分支发到预发环境，并进行测试
11. 测试完成，将 `release-0.0.1` 和入 `master` 分支并准备发版
12. 删除 `release-0.0.1` 分支代码
13. 创建 `v0.0.1` tag

以上解决步骤看起来可能复杂，其实最后只有一条根本的原则， `feat1` 分支中不能包含 `feat2` 的任何代码，反之亦然，这样无论是任何原因导致了，需求无法上线，都可以快速的做出应对，解决了由于一些不可抗力导致代码回滚

## 后记
这个方案在描述完成后，有小伙伴提出那我们仓库的分支岂不是会非常多？此处可使用[分支](#分支)中提到的命令查看是否有把分支合并到master/main分支中

如下是我之前一个开发的项目分支：

![image](https://user-images.githubusercontent.com/8288067/187079282-07459456-f4da-4bac-a578-c6d2c572e5c6.png)

