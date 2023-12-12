---
title: 统计代码行数
categories:
  - git
excerpt: '统计代码行数'
description: '统计代码行数'
date: 2023-12-12 10:09:07
---


```sh
#!/bin/bash

function bold() {
  echo -e "\033[1m$1\033[0m"
}

# 获取最早的提交时间
first_commit=$(git log --reverse --pretty=format:"%ct" | head -1)

# 获取最早的提交年份和月份
if [ $(uname) = "Darwin" ]; then
  year=$(date -r $first_commit "+%Y")
  month=$(date -r $first_commit "+%m")
else
  year=$(date -d @$first_commit "+%Y")
  month=$(date -d @$first_commit "+%m")
fi

# 获取当前年份和月份
current_year=$(date "+%Y")
current_month=$(date "+%m")

if [ $# -ne 0 ]; then
  if [ $current_year -lt $1 ]; then
    echo "invalid year"
    exit 0
  fi

  if [ $year -gt $1 ]; then
    echo "invalid year"
    exit 0
  fi

  year=$1
  month="01"
  if [ $current_year -ne $1 ]; then
    current_month="12"
  fi
fi


# 遍历每个月份和每个开发者，统计提交记录
while [ "$year$month" -le "$current_year$current_month" ]
do
  # 获取指定月份的起始和结束时间
  if [ $(uname) = "Darwin" ]; then
    since=$(date -j -f "%Y-%m-%d" "$year-$month-01" "+%Y-%m-%d")
    until=$(date -v+1m -v1d -v-1d -jf "%Y-%m-%d" "$since" "+%Y-%m-%d")
  else
    since=$(date -d "$year-$month-01" "+%Y-%m-%d")
    until=$(date -d "$since +1 month -1 day" "+%Y-%m-%d")
  fi

  bold "$since-$until"

  # 获取所有开发者的提交记录
  authors=$(git shortlog -sn --no-merges | awk '{print $2}')

  # 遍历每个开发者，统计指定时间段内的提交记录
  for author in $authors
  do
    if git log --author="$author" --since="$since" --until="$until" --no-merges | grep -q "commit "; then
      # echo "在 $since 到 $until 时间段内，$author 有提交记录"
      result=$(git log --author="$author" --since="$since" --until="$until" --pretty=tformat: --numstat | awk '{ add += $1 ; subs += $2 } END { printf "%-15s  \033[32madd++\033[0m: %-8s \033[31mdel--\033[0m: %-8s\n", "'"$author"'", add, subs}')
      echo "$result"
    #else
      # echo "在 $since 到 $until 时间段内，$author 没有提交记录"
    fi

  done

  # 计算下一个月份的年份和月份
  if [ "$month" -eq 12 ]
  then
    year=$((year+1))
    month=01
  else
    month=$(printf "%02d" $((10#$month+1)))
  fi

  echo ""
done
```
