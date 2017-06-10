---
title: Git无法添加某些文件夹
date: 2017-06-11 00:33:54
tags: Git
---
有时, git 会无法添加 *vendor* 下的某些文件夹, 即使通过 `git add -f vendor` 也无法添加, 通过 `git status` 也没有关于那些文件夹的状态追踪. 一般来说, 是这些文件夹(通常是一个 git 库) 里面包含了 `.git` 文件夹, 它被识别为了 git 的 **submodule**, 列出这些 submodule 可以使用:

~~~ bash
git ls-files --stage | grep 160000
~~~

## 解决方案

### 移除 `.git`

一个简单(但不优雅)的处理方案是, 移除掉那些文件夹下的 `.git` 文件夹, 让这些不被识别为 submodule,
可通过以下的命令来移除:

~~~ sh
# zsh
rm -rf vendor/**/.git

# bash
find vendor/ -type d -name ".git" -exec rm -rf {} \;
~~~

然后执行:

~~~ sh
git rm --cached -r vendor
git add vendor
~~~

即可.

### 添加 `.gitignore` 匹配模式

在 `.gitignore` 中添加如下的匹配模式

~~~ .gitignore
/vendor/**/.git
~~~

<!-- more -->

## 参考

- <http://stackoverflow.com/questions/24472596/git-fatal-pathspec-is-in-submodule>
- <http://stackoverflow.com/questions/23882794/git-cannot-push-folder-part-of-another-git-project>
- <https://getcomposer.org/doc/faqs/should-i-commit-the-dependencies-in-my-vendor-directory.md>

