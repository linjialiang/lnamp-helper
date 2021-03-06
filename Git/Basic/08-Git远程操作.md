# Git 远程操作

> 本小节会涉及到的知识点：
>
> - 从远程仓库抓取数据
> - 向远程仓库推送数据
> - `git pull` 遇到的合并冲突解决

## 从远程仓库抓取数据

> - `git fetch` 用于抓取数据
> - `git pull` 指令用于抓取+合并数据

### 抓取数据-克隆现有的仓库

> 可以直接使用 `git pull` 抓取数据：

```shell
$ git pull
Already up to date.
```

### 抓取数据-在现有目录中初始化仓库

> 本地仓库是从已有项目通过 `git init` 指令直接初始化的

### 附录：本地仓库和远程仓库是相互独立的项目，又该如何合并呢？

> 方法一：使用 `push -u`

| 案例步骤 | 新建一个项目时                          |
| -------- | --------------------------------------- |
| 1        | 本地建立目录并添加内容                  |
| 2        | 将本地内容做 git 初始化                 |
| 3        | 本地仓库新建内容做并提交 git            |
| 3        | 新建远程仓库                            |
| 4        | 为本地仓库添加远程仓库地址              |
| 5        | 将本地内容 push 到远程（首次需要加 -u） |

```shell
mkdir /home/demo
cd /home/demo
git init
touch README.md
git add README.md
git commit -m "第一次提交"
git remote add linjialiang-gitee https://gitee.com/linjialiang/demo.git
git push -u linjialiang-gitee master
```

> 方法二： 使用 `--allow-unrelated-histories` 来解决
>
> - <span style="color:#c21111;">该方法还适用于：合并分支时遇到 `unrelated history` 提示而无法合并分支的情况</span>
> - 原理： `git merge` 命令拒绝合并不共享祖先的历史记录。使用 `--allow-unrelated-histories` 选项可以覆盖 git 的这种安全机制（<span style="color:#c21111;">如非必要，慎用！</span>）

```shell
$ git pull github-git master --allow-unrelated-histories

From https://github.com/linjialiang/git
 * branch            master     -> FETCH_HEAD
Merge made by the 'recursive' strategy.
 LICENSE | 201 ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
 1 file changed, 201 insertions(+)
 create mode 100644 LICENSE
```

## 向远程仓库推送数据

> `git push` 是向远程推送数据的指令

### 1. 推送数据-克隆现有的仓库

> 直接使用 `git push`

```shell
$git push
Everything up-to-date
```

### 2. 推送数据-在现有目录中初始化仓库

> 使用 `git push <remote-name> <local-branch>:<remote-branch>`

```shell
$ git push github-git master:master
Counting objects: 160, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (159/159), done.
Writing objects: 100% (160/160), 26.23 KiB | 1.25 MiB/s, done.
Total 160 (delta 72), reused 0 (delta 0)
remote: Resolving deltas: 100% (72/72), done.
To https://github.com/linjialiang/git.git
   5b1f698..b407d71  master -> master
```

## 便捷的指令：

1.  “本地分支”如何跟踪“远程分支”？

    > “在现有目录中初始化仓库”和远程交互的时代码很繁琐，能不能简写呢？
    >
    > - 指令： `git branch --set-upstream-to=<remote-name>/<branch> <local-branch>` ，如果远程没有改分支会自动创建分支，并推送数据

    ```shell
    $ git branch --set-upstream-to=github-git/master master
    Branch 'master' set up to track remote branch 'master' from 'github-git'.
    ```

    > 跟踪后 `git push` `git pull` 就不必加上 `指定的远程仓库url简写` 及其 `指定的远程分支`！

    ```shell
    $ git push
    Everything up-to-date

    $ git pull
    Already up to date.
    ```

    > 到此“在现有目录中初始化仓库”和“克隆现有的仓库”已经没有太大区别了

2.  “本地分支”如何取消对“远程分支”的跟踪？

    > `$ git branch --unset-upstream <local-branch>` 指令可以取消跟踪

    ```shell
    $ git branch --unset-upstream master
    ```

    > 删除后 `git pull` `git push` 需要带上 `指定的远程仓库url简写` 及其 `指定的远程分支`

    ```shell
    $ git pull
    There is no tracking information for the current branch.
    Please specify which branch you want to merge with.
    See git-pull(1) for details.

        git pull <remote> <branch>

    If you wish to set tracking information for this branch you can do so with:

        git branch --set-upstream-to=github-git/<branch> master
    ```

    ```shell
    $ git pull github-git master
    From https://github.com/linjialiang/git
     * branch            master     -> FETCH_HEAD
    Already up to date.
    ```

3.  查看本地分支与远程分支的跟踪情况

    > 指令 `$ git branch --verbose --verbose`

    ```shell
    $ git branch --verbose --verbose
    * 5.1       980ba6f [github-tp5/5.1] 增加配置参数
      dev-study 30c4fed [github-tp5/dev-study] 修改了composer.lock文件，这是使用composer升级导致的
      study     2fc8768 [github-tp5/study] 钩子和行为做了部分修改
    ```

## 合并冲突

> 如果远程做了修改，本地也做了修改，就会遇到下面的问题：
>
> - 直接 `git push` 会报错，需要先 `git pull`
> - 在 `git pull` 的时候可能就会出现冲突，假如出现需要合并冲突的问题：

```shell
<<<<<<< HEAD
\`\`\`shell

\`\`\`
=======
2. 将分支2合并到分支1
> 在远程我们也做了修改 在远程我们也做了修改
>>>>>>> 73610a4f34b2819366860f53c1572fcb4522eb34
```

> 上面的代码就是在告诉我们遇到了冲突，并且告诉我们需要手动处理的代码
>
> - 手动处理结束后，使用 `git add` `git commit` 最终将合并的信息提交上去！

## 附录一：多个远程仓库，如何抓取非跟踪远程仓库数据

1.  git 抓取数据远程仓库数据的过程

    > - 如果本地某些分支跟踪了该远程仓库的部分分支，这部分分支通过 `git pull` 会直接与本地分支进行合并
    > - 如果分支未跟踪，则应该通过 `git fetch <remote-name>` 抓取到本地，在使用 `git merge <remote-name>/<remote-branch>` 进行合并

2.  为何未跟踪分支不使用 `git pull <remote-name>` 呢？

    > - 因为这种情况，使用 git pull 和 git fetch 效果一样

3.  举例
    > - `top-think-think` 是一个未跟踪的远程仓库别名
    > - `5.1` 是本地的一个分支， `top-think-think` 未跟踪本地 `5.1`

```shell
$ git fetch top-think-think
...
```

```shell
$ git merge top-think-think/5.1
...
```

## 附录二：git 如何正确更新远程分支缓存呢？

> `git pull <remote-name>` `git fetch <remote-name>` 都能起到更新远程分支缓存的作用，但是遇到下面的情况这两个方法并不管用。

1. 情况：在远程仓库已经被删除了的分支，我们使用 `git branch --remotes` 指令依然能查看到；
2. 问题：我们使用 `git pull <remote-name>` `git fetch <remote-name>` 后依然能查看到这些远程分支的缓存；
3. 解决：使用 `git remote prune <remote-name>` 或 `git fetch --prune <remote-name>` 可以更新这些已经被远程仓库删除了的分支
