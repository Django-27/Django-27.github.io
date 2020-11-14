 #  使用 tmux 的小记录 [Tmux中文手册](http://louiszhai.github.io/2017/09/30/tmux/#%E6%96%B0%E5%A2%9E%E9%9D%A2%E6%9D%BF)
常用的指令
```
tmux new -s 回话名称
tmux ls
tmux a //进入最近的会话
tmux a -t  会话名//进入指定会话
tumx kill-session -t 会话名 //干掉指定会话

ctl+b 前缀命令
ctl+b c 创建新窗口 ctl+b p 窗口间切换 w 查看所有的窗口 " 和 %  为横/竖分割面板 
, 修改窗口名称
$ 修改会话名称
? 查看帮助

```
.tumx.conf
```
set -g status-bg black
set -g status-fg white

set -g prefix C-j
nbind C-b
bind C-j send-prefix
```

解决tmux启动「can't create socket」的问题
```
strace -e trace=file tmux
rm -rf 相应的文件夹
```

```
Tmux 快捷键前缀: Ctrl-b
查看： tmux show-options -g | grep prefix
显示所有会话：Ctrl-b + s  等价于 tmux ls
新建会话: tmux new -s session_name
进入一个会话: tmux a  或 tmux a -t  session_name
中断一个会话：tmux detach 或 Ctrl-b + d
关闭会话：tmux kill-session -t session_name

返回主 shell: C-b d
恢复tmux: tmux attach

显示帮助：C-b ?
下一个内置的补救: C-b 空格
把当前窗口变为新窗口: C-b !
分割唱口: C-b " or %
显示分割窗口编号: C-b q
调到下一个分割pane：C-b o  # 关闭pane使用exit进行退出
确认后退出 tmux: C-b & 
创建新的窗口： C-b c
选择几号窗口： C-b 0~9
选择下一个窗口: C-b n
选择最后使用的窗口：C-b l
选择前一个窗口：C-b p
以菜单方式显示及选择窗口: C-b w
以菜单方式显示和选择会话: C-b s
显示时钟: C-b t


# 有关 tmux 的配置文件
1 默认会先从 /etc/tmux.conf 加载系统级的配置项
2 然后从 ~/.tmux.conf 加载用户级的配置项
3 也可以使用参数 -f 指定一个配置文件

查看： tmux show-options -g | grep prefix
设置前导：tmux set -g prefix C-j
在.tmux.conf中设置：
    # Send prefix
    set-option -g prefix C-j
    unbind-key C-j
    bind-key C-j send-prefix
    
    # Use Alt-arrow keys to switch panes
    bind -n M-Left select-pane -L
    bind -n M-Right select-pane -R
    bind -n M-Up select-pane -U
    bind -n M-Down select-pane -D
    
    setw -g mouse-resize-pane on
    setw -g mouse-select-pane on
    setw -g mouse-select-window on
    setw -g mode-mouse on

    # Shift arrow to switch windows
    bind -n S-Left previous-window
    bind -n S-Right next-window
    
    # Set easier window split keys
    bind-key v split-window -h
    bind-key h split-window -v
    
    # Easy config reload
    bind-key r source-file ~/.tmux.conf \; display-message "tmux.conf reloaded"

unbind C-b
bind C-x send-prefix
tmux kill-server 杀掉所有的
tmux ls
```

```
修改服务器时区
timedatectl status
timedatectl set-timezone "Asia/Shanghai"  // ps->  ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```
```
1.mac 终端打开就出现以下信息,无法输入命令
    Last login: *** ttys000
    [进程已完毕]
    
    原因：不知谁改动了 终端-》偏好设置-》启动-》shell打开方式
    解决的方法：命令改为:/bin/bash
2.mac 提示没安装 Homebrew, -bash: brew: command not found
    bash 下执行 ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

```
# grep (Global search Regular Expression and Print out the line)
- grep [选项] [查找模式] [文件名1，文件名2，……]
- 特殊字符“*”用来生成一个文件名列表，该列表包含当前目录下所有的文件
- grep  -r -B1 import *   # A1 B1 C1 分别是输出匹配行对应的前一行、后一行、上下文各一行
- grep  -r -q import *    # -q 表示使用静默模式，不显示结果，使用 echo $?  查看结果，0有匹配、1无匹配结果
- egrep 相当于 grep -E 即支持扩展正则模式，区别就是二者在正则的书写方式不同
- fgrep 即fast grep，不支持正则匹配，只能匹配固定的字符串，速度很快，也挺好可以没事换着用一下
```
-i：在搜索的时候忽略大小写
-n：显示结果所在行号
-c：统计匹配到的行数，注意，是匹配到的总行数，不是匹配到的次数
-o：只显示符合条件的字符串，但是不整行显示，每个符合条件的字符串单独显示一行
-v：输出不带关键字的行（反向查询，反向匹配）
-w：匹配整个单词，如果是字符串中包含这个单词，则不作匹配
-e：实现多个选项的匹配，逻辑or关系
-P：表示使用兼容perl的正则引擎
```
# find
- find * -name "*.pyc" | xargs rm -rf
- df -h 查看磁盘整体使用情况; du -h 查看磁盘目录使用情况
- 查看端口是否畅通: netstat -a

# 灵活应用多种方法pv-uv统计（一道面试题）
有一个注意点，设计到排序，不同的方法使用的条件多少、或条件不同，顺序可能不同
## 方法一：用linux命令对日志文件进行pv-uv统计
```javascript
20200701,192.168.1.3,weixin.com
20200701,192.163.1.4,qq.com
20200702,192.168.1.4,appache.org
20200702,192.168.1.5,baidu.com
20200701,162.168.1.5,189.cn
20200701,162.168.1.5,189.cn
20200701,162.168.1.6,126.com
20200701,162.168.1.6,126.com
20200701,162.168.1.6,126.com
20200701,162.168.1.6,126.com
20200703,162.168.1.6,163.com
20200701,162.168.1.7,sina.com
统计指定日期的 pv: cat pv_vu.log | grep 20200701   |  wc -l
统计指定日期的 uv: cat pv_vu.log | grep 20200701   | awk -F '[,]' '{print $2}' | uniq -c | sort -k1r | head -n 3
```
- awk -F 指定分割符号, print 表示要打印输出，$2 指定要输出的列字段没有就是整行输出
- uniq -c 指定重复行的个数
- sort 排序，-k1 指定以第一列排序，-k1r 指定逆序
## 方法二，在mysql中进行统计pv-uv
```javascript
CREATE TABLE `pv_uv_log` (
  `date` varchar(256) DEFAULT NULL,
  `ip` varchar(255) DEFAULT NULL,
  `url` varchar(255) DEFAULT NULL,
  `id` int(11) NOT NULL AUTO_INCREMENT,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=13 DEFAULT CHARSET=utf8;
INSERT INTO `mysql`.`pv_uv_log` (`date`, `ip`, `url`) VALUES ('20200701', '192.168.1.3', 'weixin.com'),('20200701', '192.163.1.4', 'qq.com'),('20200702', '192.168.1.4', 'appache.org'),('20200702', '192.168.1.5', 'baidu.com'),('20200701', '162.168.1.5', '189.cn'),('20200701', '162.168.1.5', '189.cn'),('20200701', '162.168.1.6', 'qq.com'),('20200701', '162.168.1.6', '126.com'),('20200701', '162.168.1.6', '126.com'),('20200701', '162.168.1.6', '126.com'),('20200703', '162.168.1.6', '163.com'),('20200701', '162.168.1.7', 'sina.com')
---
统计指定日期的 pv: 
统计指定日期的 uv: SELECT ip, count(1) AS uv from pv_uv_log where date = '20200701' GROUP BY ip order by uv DESC limit 3
```
## 方法三，使用py脚本
``` javascript
file = '/var/log/001.log'
pv_day, uv_day = {}, {}
with open(file, 'r') as f:
    for line in f.readline():
        datatime_str, ip, url = line.split(',')

        if datatime_str in pv_day:
            pv_day[datatime_str] += 1
        else:
            pv_day[datatime_str] = 1

        uv_str = '{}{}'.format(datatime_str, ip)
        if uv_str in uv_day:
            uv_day[uv_str] += 1
        else:
            uv_day[uv_str] = 1
```

# 操作系统命令 type 和 printenv 和 hash 的理解
- type 可以查看买命令是内部命令还是外部命令
如：type pip -> pip is /Users/sunny/code/envs/kivy-qiyuan/bin/pip
如：cd is a shell builtin
- printenv 通过这条命令可已查看系统的环境变量信息(不仅有echo $PATH的内容，还有很多)
- hash 是一个缓存表：系统初始hash表为空，当外部命令执行时，默认会从PATH路径下寻找该命令，找到后会将这条命令的路径记录到hash表中，当再次使用该命令时，shell解释器首先会查看hash表，存在将执行之，如果不存在，将会去PATH路径下寻找，利用hash缓存k-v表可大大提高命令的调用速率
如：hash -r 可以实现清除这个缓存表
- date, date -u, cal, cal -y 可以实现查看系统时间，还有指定年日历