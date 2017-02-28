git技巧

默认git是对文件名大小写不敏感的，配置git使其对文件名大小写敏感如下：
git config core.ignorecase false

或者更改git配置文件：
[core]
	repositoryformatversion = 0
	filemode = true
	bare = false
	logallrefupdates = true
	ignorecase = false
	precomposeunicode = true

将ignorecase设置为false即可。

##删除commit
不小心提交了错误的代码到库上，想要删除最近一次提交的所有信息（包括代码和日志）。
删除最后一次的提交记录：
    
    git reset --hard HEAD^
删除最后3次的提交记录：

    git reset --hard HEAD~3
已经提交到远端库上了？那就还需要将本地库强行同步到远端库上（注意执行这个命令前先检查一下有没有别人的提交记录，不然会覆盖他人的修改）：
    
    git push --force