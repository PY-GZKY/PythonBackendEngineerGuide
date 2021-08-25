> 记录一些 Git 常用操作以及避坑指南


## `Everything up-to-date`
当我们改动一些文件中的代码的时候，需要重新提交至 `Github` 或 `Gitee` 上，这个时候使用
`git status` 查看 `git` 状态。

```shell
$ git status
On branch master
Your branch is up to date with 'origin/master'.

nothing to commit, working tree clean

```

这表示当前我们没有需要提交的代码文件，意味着你并没有改动代码。

尝试改动 `index.html` 主页看看
```shell
$ git status
On branch master
Your branch is up to date with 'origin/master'.

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   index.html

```

再次查看 `git` 状态出现了`改动后并未提交`的提示。

我们需要:

```shell
git add index.html
git commit -m "update to index.html"
git push origin master
```

重新添加，提交并`push` 到远程仓库，当然你也可以添加 `-f` 参数执行`强制覆盖`远程仓库，只是我不建议你这么做 ！！

```shell
git push -f origin master
```




## `Git` 权限问题

这是一个很常见的问题，一般而言我们开始构建 `Git` 的流程应该是这样的:

```shell
cd demo/
git init
git add .
git commit -m "首次提交"
git remote add origin git@github.com:py-gzky/nice.git
push -u origin main
```

直到最后一步可能会提示 `push` 失败，理由是`没有权限`。

这个时候需要到远程仓库上添加`ssh公钥`。

首先在本地机器按如下命令来生成 `sshkey`:

```shell
ssh-keygen -t rsa -C "Tongge@qq.com"  
# Generating public/private rsa key pair...
```

至于` -C` 参数后面的邮箱地址可随意使用，无关大雅。
按照提示完成`三次回车`，
即可生成 `ssh key`。

通过查看 `~/.ssh/id_rsa.pub` 文件内容，获取到你的 `public key`

大概是这样的:
```shell
ssh-rsa AAAAB3NxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxpNkG+MA8tvRs
SAfp0aPFzNLELx1p1Ixxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx5R6SU/m0m4V3
/xE7CJNfj3nQ1U= Tongge@qq.com
```

复制添加这一段到 `Github`/`Gitee` 仓库公钥，再次 `push` 即可成功 ！！


## 隐藏的`.idea/`

`Python`开发者酷爱使用`Pycharm`编辑器进行代码开发。

某些时候运行代码会在项目中自动生成 `.idea/` 文件夹，这个文件对我们来说并没有什么作用。

但是每次查看 `git status` 的之后却总是有提示

```shell

$ git status

On branch master

Your branch is up to date with 'origin/master'.

Untracked files:

  (use "git add <file>..." to include in what will be committed)

        .idea/

nothing added to commit but untracked files present (use "git add" to track)

```

这可不太友好 ?? !!

- 通过 `.gitignore` 文件阻止
- `git clean`

```shell
# 删除 untracked files
git clean -f
 
# 连 untracked 的目录也一起删掉
git clean -fd
 
# 连 gitignore 的untrack 文件/目录也一起删掉 （慎用，一般这个是用来删掉编译出来的 .o之类的文件用的）
git clean -xfd
 
# 在用上述 git clean 前，墙裂建议加上 -n 参数来先看看会删掉哪些文件，防止重要文件被误删
git clean -nxfd
git clean -nf
git clean -nfd
```

## `pull-to-push`

在`多人协作`开发的时候，可能会出现下面的场景:

![](https://gitbook.tw/images/tw/github/fail-to-push/fail1.png)

- `Sherly` 跟 `Eddie` 在差不多的時間都从` Git Server` 上拉了一份代码到本地准备开发。
- `Sherly` 效率很高啊，先完成了，于是先把做好的成果推一份上去。
- `Eddie` 不久也完成了，但他发现写好的代码推动不上去了......

> 如何解决 ??

### `先拉再推`
现在你本地的代码是最新的(后于他人提交)，需要先`合并`远程仓库的代码，再`push`到远程仓库
```shell
$ git pull --rebase origin master 
remote: Counting objects: 3, done.
remote: Compressing objects: 100% (2/2), done.
remote: Total 3 (delta 1), reused 3 (delta 1), pack-reused 0
Unpacking objects: 100% (3/3), done.
From https://github.com/eddiekao/dummy-git
   37aaef6..bab4d89  master     -> origin/master
First, rewinding head to replay your work on top of it...
Applying: update index
```

```shell
git push origin master
```

也就是说有人先于你提交了代码，这个时候你需要拉取他提交的版本，在他的基础上进行开发，
这样不做才能顺利提交并且有迹可循，版本清晰 ！！

### `无视规则`，什么都听老子的 ！

出于对他人劳动成果的尊重我们采用先拉后推的形式提交代码。

但是如果你的权力足够大则可以打破这个规则，就是上面我们提过的强制提交。

这会直接覆盖掉别人的版本，别人的代码有可能就白改了，该代码库中只有你一家之言。

```shell
$ git push -f origin master
Counting objects: 19, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (17/17), done.
Writing objects: 100% (19/19), 2.16 KiB | 738.00 KiB/s, done.
Total 19 (delta 6), reused 0 (delta 0)
remote: Resolving deltas: 100% (6/6), done.
To https://github.com/eddiekao/dummy-git.git
 + 6bf3967...c4ea775 master -> master (forced update)
```

虽然这样一定会执行成功，
但接下來你就要去面对 `Sherly`，
跟她解释你为什么会把她的代码给覆盖掉。

> 以上例子来自 [怎麼有時候推不上去](https://gitbook.tw/chapters/github/fail-to-push.html) 在此表示感谢 ！！



## 本地多仓库账号冲突

`remote: You do not have permission push to this repository`

```shell

$ git push -u origin master
remote: You do not have permission push to this repository
fatal: unable to access 'https://gitee.com/zy8080/springbootjdbc.git/': The requested URL returned error: 403
```

## `Git` 更改账号密码

`控制面板`--》`用户账户`--》`管理windows凭据`--》找到`git`的`凭据`，`删除`他，后面`push`的时候会再弹出填账号密码的框,这个时候会重新生成凭证 (支持多凭证，不过一般情况下用不到)


## 一般提交步骤

`git init` 

`git add .`

`git commit -m "first commit"` 

`git remote add origin + 远程仓库地址（https）`

```shell
git pull origin master 
git push -u origin master（第一次加 -u）
```

```shell
git push -f origin master // 强制推送
```


### SSH

`-f` 选项指定生成文件的文件名

`ssh-keygen -t rsa  -C "v_duanjiawei@baidu.com" -f duanjiawei`

生成如下2个文件

`duanjiawei`  `duanjiawei.pub`

> PR: [https://zhuanlan.zhihu.com/p/87603185](https://zhuanlan.zhihu.com/p/87603185)