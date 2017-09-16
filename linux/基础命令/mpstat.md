# mpstat
可以查看多核心CPU中，每个计算核心的统计数据。
看每个cpu核心的详细当前运行状况信息：

mpstat -P ALL 2
```
03:09:44 PM  CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest   %idle
03:09:46 PM  all    2.25    0.00    0.50    0.00    0.00    0.00    0.00    0.00   97.25
03:09:46 PM    0    6.97    0.00    1.49    0.00    0.00    0.00    0.00    0.00   91.54
03:09:46 PM    1    1.01    0.00    0.50    0.00    0.00    0.00    0.00    0.00   98.49
03:09:46 PM    2    0.50    0.00    0.50    0.00    0.00    0.00    0.00    0.00   99.00
03:09:46 PM    3    0.00    0.00    0.50    0.00    0.00    0.00    0.00    0.00   99.50
```

```
%user      在internal时间段里，用户态的CPU时间(%)，不包含nice值为负进程  (usr/total)*100
%nice      在internal时间段里，nice值为负进程的CPU时间(%)   (nice/total)*100
%sys       在internal时间段里，内核时间(%)       (system/total)*100
%iowait    在internal时间段里，硬盘IO等待时间(%) (iowait/total)*100
%irq       在internal时间段里，硬中断时间(%)     (irq/total)*100
%soft      在internal时间段里，软中断时间(%)     (softirq/total)*100
%idle      在internal时间段里，CPU除去等待磁盘IO操作外的因为任何原因而空闲的时间闲置时间(%) (idle/total)*100
```