title: git commit 时指定时间和作者等信息
date: 2017-06-12 14:32:00
tags: [git, 技术]
categories: 泛技术

---

最近遇到了这么一个问题，需要在提交代码的时候指定提交的时间和作者等信息，而不是当前用户和当前时间提交（不要问我为什么，就是有这么个需求）。

经过一通的搜索查找资料，终于找到一个还算是比较方便的办法。

```
GIT_AUTHOR_DATE="2017-06-01 12:33:08" \
    GIT_COMMITTER_DATE="2017-06-01 12:33:08" \
    git commit . \
    --author="author name <author@email.com>" \
    -m"some commit message"
```

是不是很简单。

