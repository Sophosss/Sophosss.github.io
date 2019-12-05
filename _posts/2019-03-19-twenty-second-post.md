---
title: 不定期更shell小坑坑
author: Sophosss
layout: post
---
Linux虽然敲起来挺爽的，然鹅总是有眼前一亮的感觉。

友情提示：Linux 不像 Windows，有回收站有撤销。它里面删了就是删了，所以说备份很重要！！都是泪。。

- 如果你要修改文件的后缀名，我劝你稳一点：`find . -type f -name "*.txt" > /tmp/txt.list`
- awk -F分割的时候如果遇到 .或|| 这种则需要转义，要两个\\\：`awk -F'\\|\\|'`
- awk 中 print 的时候如果要打印倒数第二行，记得加（），否则就是最后一行数值 -1：`print $(NF-1)`
- if语句一定要前后带空格 ：`if [ balabala ]`
- 去重之前需要先排个序：`sort |uniq -c |sort -nr`
- `xargs -n 1`表示每行只传递一个参数，比如适用于一段文本进行词频统计
- sed 中 N 的原理是，先读入一行到 sed 的模板空间，加个换行符（\n），再向 sed 模板空间追加下一行（之后sed 对模板空间中的内容执行s/\n/ /替换，并显示替换后的内容）：`sed 'N;s/\n/ /g' o1.txt`

