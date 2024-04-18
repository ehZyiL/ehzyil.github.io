---
title: 修改某个人在Git中所有提交记录的名字
date: 2024-04-17
tags:
  - 踩坑
  - git
categories:
  - 记录
author: ehzyil
description: 
cover: 
banner:
---

1. **确认本地全局邮箱/用户名**：  
    使用命令`git config user.name`和`git config user.email`查看当前设置的用户名和邮箱。如果需要修改，可以使用`git config --global user.name "新用户名"`和`git config --global user.email "新邮箱地址"`进行设置【3】。
    
2. **使用`git filter-branch`命令**：  
    `git filter-branch`是一个强大的工具，可以用来修改历史记录中的信息。你可以通过编写一个shell脚本来执行这个命令，脚本中包含如下内容：
```
    #!/bin/sh
git filter-branch --env-filter '
OLD_EMAIL="旧邮箱地址"
CORRECT_NAME="新名字"
CORRECT_EMAIL="新邮箱地址"
if [ "$GIT_COMMITTER_EMAIL" = "$OLD_EMAIL" ]
then
  export GIT_COMMITTER_NAME="$CORRECT_NAME"
  export GIT_COMMITTER_EMAIL="$CORRECT_EMAIL"
fi
if [ "$GIT_AUTHOR_EMAIL" = "$OLD_EMAIL" ]
then
  export GIT_AUTHOR_NAME="$CORRECT_NAME"
  export GIT_AUTHOR_EMAIL="$CORRECT_EMAIL"
fi
' --tag-name-filter cat -- --branches --tags

```
    
    这个脚本会遍历所有的提交记录，查找提交者邮箱为`旧邮箱地址`的记录，并将其中的用户名和邮箱修改为`新名字`和`新邮箱地址`【3】【2】。
    
3. **执行脚本**：  
    保存上述脚本为一个`.sh`文件，并在Git Bash中执行它。执行时间可能会较长，特别是当提交记录很多时【3】。
    
4. **推送修改到远程仓库**：


---
references:
  - 检索 如何修改Git提交历史中的author，email和name等信息 - 知乎([https://zhuanlan.zhihu.com/p/455741996](https://zhuanlan.zhihu.com/p/455741996)) ...  
  - 检索 修改Git Commit提交记录的用户名Name和邮箱Email ...([https://www.cnblogs.com/schips/p/change_git_commit_user_info.html](https://www.cnblogs.com/schips/p/change_git_commit_user_info.html)) ...  
  - 检索 Git 修改历史 commits 中的用户名和邮箱 - lelelong - 博客园([https://www.cnblogs.com/longjunhao/p/15005916.html](https://www.cnblogs.com/longjunhao/p/15005916.html)) ...
    ...
