---
title: git使用日志
date: 2021-05-21 15:38:06
---


### git使用记录

#### 一般流程
```bash
git clone git@github.com:michaelliao/learngit.git
git checkout -b dev origin/dev
```
```bash
git clone git@github.com:michaelliao/learngit.git
git checkout -b dev
git branch -u origin/dev	//git branch --set-upstream dev origin/dev
```

#### 修改提交
```
git commit --amend -m "new message"
```

```bash
git add <file_name>
git commit --amend HEAD~1
```

#### 版本回退
```bash
git log --oneline
git reset --hard 1094a
git push -f
```

#### 撤销回退
```bash
git reflog
git reset --hard 1094a
git reset --soft 1094a //保留更改
git reset --hard origin/<branch_name> //重置原始版本
```

#### 删除中间某次提交
```bsah
git revert -n 1094a
git commit -m "revert 1094a"
git push
```

#### 隐藏工作区
```bash
git stash
git stash list
git stash pop
```

#### 添加远程仓库
```bash
git remote add origin git@github.com:michaelliao/learngit.git
git push -u origin master
```

#### 配置别名
全局配置文件存放在`~/.gitconfig`
将日志打印的优雅：
```bash
git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"
```


**注意：**本地存在多个分支时，新建分支要先切换到主分支。

