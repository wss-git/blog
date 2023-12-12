---
title: git 已提交的 commit 中提交者的用户名和邮箱
categories:
  - Git
excerpt: 'git 已提交的 commit 中提交者的用户名和邮箱'
description: 'git 已提交的 commit 中提交者的用户名和邮箱'
date: 2023-12-12 10:02:16
---


## 背景

在互联网冰冬下，为了公司蓬勃发展，又又又出幺蛾子了：统计每个人的代码量和需求量，但是又只能统计公司的邮箱。我们开源团队每个月全是 0，在这样的背景下，花点时间做个临时（长期）方案。

1. 开源仓库统一收敛到内部仓库的一个组织下
2. 将内部仓库的邮箱验证关掉
3. 定制流水线（ 公司内部流水线**太太太高大上**，玩不来。最终这一步变成了人肉作这一步 ）

- 定时运行，修改 commit 邮箱推送然后到另一个副本组织
- 事件触发器，推送到公网

## Git 脚本

> 运行命令 xx.sh old-email-1 new-name-1 new-email-1

### 仅修改当前分支

```sh
#!/bin/bash

# Set the old and new email and name values
OLD_EMAIL=$1
CORRECT_NAME=$2
CORRECT_EMAIL=$3

# Filter the current branch
git filter-branch --env-filter "
    if [ \$GIT_COMMITTER_EMAIL = '$OLD_EMAIL' ]
    then
        export GIT_COMMITTER_NAME='$CORRECT_NAME'
        export GIT_COMMITTER_EMAIL='$CORRECT_EMAIL'
    fi
    if [ \$GIT_AUTHOR_EMAIL = '$OLD_EMAIL' ]
    then
        export GIT_AUTHOR_NAME='$CORRECT_NAME'
        export GIT_AUTHOR_EMAIL='$CORRECT_EMAIL'
    fi
" --tag-name-filter cat -- --branches HEAD

# Delete the original refs after the filter is done
git for-each-ref --format='%(refname)' refs/original/ | xargs -n 1 git update-ref -d

# Force push the changes to the remote repository
git push --force
```

### 仅修改最近两个月的当前分支的邮箱和用户名

```sh
#!/bin/bash

# Set the old and new email and name values
OLD_EMAIL=$1
CORRECT_NAME=$2
CORRECT_EMAIL=$3

# Find the commit hash of the commit from two months ago
TWO_MONTHS_AGO=$(git log --branches --remotes --since="2 months ago" --pretty=format:"%H" | tail -1)

# Filter the current branch from two months ago to HEAD
git filter-branch --env-filter "
    if [ \$GIT_COMMITTER_EMAIL = '$OLD_EMAIL' ]
    then
        export GIT_COMMITTER_NAME='$CORRECT_NAME'
        export GIT_COMMITTER_EMAIL='$CORRECT_EMAIL'
    fi
    if [ \$GIT_AUTHOR_EMAIL = '$OLD_EMAIL' ]
    then
        export GIT_AUTHOR_NAME='$CORRECT_NAME'
        export GIT_AUTHOR_EMAIL='$CORRECT_EMAIL'
    fi
" --tag-name-filter cat $TWO_MONTHS_AGO..HEAD

# Delete the original refs after the filter is done
git for-each-ref --format='%(refname)' refs/original/ | xargs -n 1 git update-ref -d

# Force push the changes to the remote repository
git push --force
```

### 全局修改

```sh
#!/bin/bash

# Set the old and new email and name values
OLD_EMAIL=$1
CORRECT_NAME=$2
CORRECT_EMAIL=$3

# Loop through all branches and tags
for branch in $(git for-each-ref --format='%(refname:short)' refs/heads refs/tags); do
    # Check out each branch or tag
    git checkout $branch 2>/dev/null

    # Filter the current branch or tag
    git filter-branch --env-filter "
        if [ \$GIT_COMMITTER_EMAIL = '$OLD_EMAIL' ]
        then
            export GIT_COMMITTER_NAME='$CORRECT_NAME'
            export GIT_COMMITTER_EMAIL='$CORRECT_EMAIL'
        fi
        if [ \$GIT_AUTHOR_EMAIL = '$OLD_EMAIL' ]
        then
            export GIT_AUTHOR_NAME='$CORRECT_NAME'
            export GIT_AUTHOR_EMAIL='$CORRECT_EMAIL'
        fi
    " --tag-name-filter cat -- --branches --tags

    # Delete the original refs after the filter is done
    git for-each-ref --format='%(refname)' refs/original/ | xargs -n 1 git update-ref -d
done

# Switch back to the original branch
git checkout -

```

## 运行脚本

修改当前分支近三个月的 commit，然后推送到另一个指定仓库

> xx.sh fc

```sh
#!/bin/bash

if [ -z "$1" ]; then
  echo "Error: 请指定仓库名称"
  exit 1
fi

CURRENT_BRANCH=$(git rev-parse --abbrev-ref HEAD)
BK_CURRENT_BRANCH=${CURRENT_BRANCH}"_replace_committers"
git checkout -b $BK_CURRENT_BRANCH

# Set the old and new email and name values
OLD_EMAIL_WSS=wssgryx@163.com
CORRECT_NAME_WSS=wss
CORRECT_EMAIL_WSS=wss@inc.com

OLD_EMAIL_SHL=xxx@126.com
CORRECT_NAME_SHL=shl
CORRECT_EMAIL_SHL=shl@inc.com

# Find the commit hash of the commit from two months ago
TWO_MONTHS_AGO=$(git log --branches --remotes --since="3 months ago" --pretty=format:"%H" | tail -1)

# Filter the current branch from two months ago to HEAD
git filter-branch --env-filter "
    if [ \$GIT_COMMITTER_EMAIL = '$OLD_EMAIL_WSS' ]
    then
        export GIT_COMMITTER_NAME='$CORRECT_NAME_WSS'
        export GIT_COMMITTER_EMAIL='$CORRECT_EMAIL_WSS'
    fi
    if [ \$GIT_AUTHOR_EMAIL = '$OLD_EMAIL_WSS' ]
    then
        export GIT_AUTHOR_NAME='$CORRECT_NAME_WSS'
        export GIT_AUTHOR_EMAIL='$CORRECT_EMAIL_WSS'
    fi

    if [ \$GIT_COMMITTER_EMAIL = '$OLD_EMAIL_SHL' ]
    then
        export GIT_COMMITTER_NAME='$CORRECT_NAME_SHL'
        export GIT_COMMITTER_EMAIL='$CORRECT_EMAIL_SHL'
    fi
    if [ \$GIT_AUTHOR_EMAIL = '$OLD_EMAIL_SHL' ]
    then
        export GIT_AUTHOR_NAME='$CORRECT_NAME_SHL'
        export GIT_AUTHOR_EMAIL='$CORRECT_EMAIL_SHL'
    fi
" --tag-name-filter cat $TWO_MONTHS_AGO..HEAD

# Delete the original refs after the filter is done
git for-each-ref --format='%(refname)' refs/original/ | xargs -n 1 git update-ref -d

# Force push the changes to the remote repository
git remote add bk http://github.com/serverless-devs-backup/$1.git
git push bk $BK_CURRENT_BRANCH --force --no-verify

# Clear
git remote remove bk
git checkout $CURRENT_BRANCH
git branch -D $BK_CURRENT_BRANCH

```

