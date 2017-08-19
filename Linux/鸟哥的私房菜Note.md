##Linux鸟哥私房菜Note

Linux是多人多任务的操作系统，每个人都有自己的工作目录，假如登入用户是vbird,那么我的个人目录就是/home/vbird。

一般登入进去后显示[vbird@www ~]$ _： vbird表示目前用户的账号，www表示主机名，~指目前所在的目录，$是提示字符。（root的提示字符为#，而一般用户的提示字符为$）

从Linux**注销**：exit

**ls** ：列出目录下的文件

**echo $LANG**：显示目前所支持的语系

**LANG=en_US**：修改语系成为英文语系

显示日期与时间 ： date [+%Y%m%d] 或者 [+%H:%M]

显示日历 ： cal [month] [year]   ex:cal 10 2016 or cal 2009

计算器：bc  （离开quit）

[ctrl+d]组合键表示键盘输入结束，也可以取代exit的输入。

man是manual(操作说明)的简写。man+命令可以查看命令的具体使用。空格键可以翻页，q可以离开man的环境。







