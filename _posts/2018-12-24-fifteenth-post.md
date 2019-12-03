---
title: 好用的一行脚本(也有不是一行的…)
author: Sophosss
layout: post
---
1. 查看系统当前网络连接数
   
   ```shell
   netstat -n | awk '/^tcp/ {++S[$NF]} END {for(a in S) print a, S[a]}'
   ```
   
   $NF就是一行数据最后一列的那个值，S为数组，里面的元素就是连接状态，S={LISTEN,TIME_WAIT....}，S[LISTEN]默认为0，++S[LISTEN]用来记录出现 LISTEN 的个数。
   
2. ```shell
   #!/bin/bash
   if [ $# == 0 ]
   then
       ls -ld `pwd`
   else
       for i in `seq 1 $#`
       do
           a=$i
           echo "ls ${!a}"
           ls -l ${!a} |grep '^d'
       done
   fi
   ```

   这里是一个特殊用法，在shell中，\$1为第一个参数，​\$2为第二个参数，以此类推，那么这里的数字要是一个变量如何表示呢？比如n=3,我想取第三个参数，能否写成 $$n？

   shell中是不支持的，那怎么办？ 就用脚本中的这种方法： a=​\$n, echo ${!a}

3. 按照 CPU/内存的使用情况列出前10 的进程

   ```shell
   #内存
   ps -axo %mem,pid,euser,cmd | sort -nr | head -10
   #CPU
   ps -aeo pcpu,user,pid,cmd | sort -nr | head -10
   ```

4. 显示系统整体的 CPU利用率和闲置率

   ```shell
   grep "cpu " /proc/stat | awk -F ' ' '{total = $2 + $3 + $4 + $5} END {print "idle \t used\n" $5*100/total "% " $2*100/total "%"}'
   ```

5. 按线程状态统计线程数

   ```shell
   jstack $pid | grep java.lang.Thread.State:|sort|uniq -c | awk '{sum+=$1; split($0,a,":");gsub(/^[ \t]+|[ \t]+$/, "", a[2]);printf "%s: %s\n", a[2], $1}; END {printf "TOTAL: %s",sum}'
   ```

6. JVM 内存使用及垃圾回收状态统计

   ```shell
   #显示最后一次或当前正在发生的垃圾收集的诱发原因
   jstat -gccause $pid
   
   #显示各个代的容量及使用情况
   jstat -gccapacity $pid
   
   #显示新生代容量及使用情况
   jstat -gcnewcapacity $pid
   
   #显示老年代容量
   jstat -gcoldcapacity $pid
   
   #显示垃圾收集信息（间隔1秒持续输出）
   jstat -gcutil $pid 1000
   ```

7. 快速杀死所以 Java 进程

   ```shell
   ps aux | grep java | awk '{ print $2 }' | xargs kill -9
   ```

8. 查找/目录下占用磁盘空间最大的top10文件

   ```shell
   find / -type f -print0 | xargs -0 du -h | sort -rh | head -n 10
   ```

   