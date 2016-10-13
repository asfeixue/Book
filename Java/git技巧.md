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