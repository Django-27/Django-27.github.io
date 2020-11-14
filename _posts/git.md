# git tips
- 1 github 配置SSH key, cd ~/.ssh 
- 2 ssh-keygen -t rsa -C "xxxxxxx@outlook.com" -f "github_id_rsa"
- 3 cat github_id_rsa.pub, 把内容复制到github的ssh中保存
- 4 编辑 ~/.ssh/config , 添加快捷配置
- 5 测试：ssh -T git@github.com

git config --global user.name "**"
git config --global user.email "**"

- git config -e           会打开该项目所属的配置文件（作用域最小，值针对当前项目有效）
- git config -e --global  会打开C:\Users\Sunny\.gitconfig下的配置文件（作用域中等，为登陆这台计算机的用户）
- git config -e --system  会打开D:\Program Files\Git\etc\gitconfig（作用域最大，整台计算机，不管登陆那个帐号，不管哪个项目）
- 优先级git config > git config --global > git config --system, 所以有些配置可以写到全局，有些只能写到项目内（如name、email）
```
本地分支、远程没有：git push origin local_br:remote_branch   #  git push  A  B:C    # 当B=C，可省略为 git push A B
如果本地新建了一个分支 branch_name，但是在远程没有(无关联): git push --set-upstream origin branch_name
如果远程新建了一个分支，本地没有该分支: git checkout --track origin/branch_name 
远程已有、但未关联本地：git push -u origin/remote_branch
删除本地：git branch -d local_branch
删除远程：git push origin :remote_branch
拉取远程分支到本地：git checkout -b local_branch origin/remote_branch  (git fetch origin remote_branch:local_branch)

git的服务器端(remote)端包含多个repository，每个repository可以理解为一个项目,而每个repository下有多个branch
"origin"就是指向某一个repository的指针。
服务器端的"master"（强调服务器端是因为本地端也有master）就是指向某个repository的一个branch的指针。
```
![image](https://github.com/Django-27/workspace/blob/master/pic/git-tips1.png)

```
一、开发分支（dev）上的代码达到上线的标准后，要合并到 master 分支
git checkout dev
git pull
git checkout master
git merge dev
git push -u origin master
二、当master代码改动了，需要更新开发分支（dev）上的代码
git checkout master 
git pull 
git checkout dev
git merge master 
git push -u origin dev
```

#《Git 权威指南》 阅读笔记
```
git rm --cached winxp.img  #  删除已经提交的文件；直接从暂存区删除文件，工作区不变
git commit --amend  # 重写提交说明
git rebase -i <commit-id>  # 修改某个历史提交的提交说明

git add -u  # 将所有本地文件变更（添加和删除、修改）加入暂存区
git add -A  # 将本地删除文件、新增文件都登记到提交暂存区
git add -p  # 对一个文件内的修改进行有选择性的添加

git log | less  # 通过管道将输出交给一个分页器
git config --global core.pager 'less -+$LESS -FRX'

git config --global user.name aaa
git config --global user.email aaa@163.com  #  删除全局配置 git config --unset --global user.email

git log --stat | less  # --stat 参数， 可以看到每次变更文件的统计
git ls-tree -l HEAD  
# 2**64 = 209万TB，1TB=1024GB

git reset --mixed <commit-id>  # 默认，撤回add操作
git reset --soft  <commit-id>  # 只改变引用指向，不改变工作区和暂存区 ；即撤销comit操作，git reset --soft HEAD^
git reset --hard  <commit-id>  # 强回退, 改动将丢失

cat .git/HEAD  # 查看当前的HEAD指向
git reflog     # 查看HEAD的变迁过程
git rev-list HEAD  | wc -l   # 查看提交次数
git blame -L n, m  file.py  # 查看文件改动记录
git commit --amend  # 单步悔棋
git reset --soft HEAD^^  # 多不悔棋(合并分散的commit到一个总的commit)，回到最近两次提交前，在统一commit

git cherry-pick  # 从众多提交中拣选出一个，并应用到当前的工作分支上
git tag D HEAD^^  # 前两次，打上标签
git tag F HEAD~5  # 前5次，打上标签

git revert HEAD   # 翻转提交，即修正错误的提交

git show-ref  # 查看所包含的引用
du -sh .git   # 查看占用空间
# git gc  # 清理优化，除非必要，否则没必要使用
```

# 让 shell 显示当前 git 的分支名称
```
vim ~/.bashrc

function git-branch-name {
  git symbolic-ref HEAD 2>/dev/null | cut -d"/" -f 3
}
function git-branch-prompt {
  local branch=`git-branch-name`
  if [ $branch ]; then printf " [%s]" $branch; fi
}
PS1="\u@\h \[\033[0;36m\]\W\[\033[0m\]\[\033[0;32m\]\$(git-branch-prompt)\[\033[0m\] \$ "

source ~/.bashrc
```
恢复git误删除的文件或者文件夹
``` 
    git reset HEAD [文件或者文件夹]
    git checkout [文件或者文件夹]
```
# GIT显示漂亮日志的小技巧
```
git log --graph --oneline --decorate --all

git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit --"
git lg


来看一个实际的例子，如果要查看 Git 仓库中，2008 年 10 月期间，Junio Hamano 提交的但未合并的测试文件，可以用下面的查询命令[：](https://git-scm.com/book/zh/v2/Git-%E5%9F%BA%E7%A1%80-%E6%9F%A5%E7%9C%8B%E6%8F%90%E4%BA%A4%E5%8E%86%E5%8F%B2)
$ git log --pretty="%h - %s" --author=gitster --since="2008-10-01" \
   --before="2008-11-01" --no-merges -- t/
5610e3b - Fix testcase failure when extended attributes are in use
acd3b9e - Enhance hold_lock_file_for_{update,append}() API
f563754 - demonstrate breakage of detached checkout with symbolic link HEAD
d1a43f2 - reset --hard/read-tree --reset -u: remove unmerged new paths
51a94af - Fix "checkout --track -b newbranch" on detached HEAD
b0ad11e - pull: allow "git pull origin $something:$current_branch" into an unborn branch
在近 40000 条提交中，上面的输出仅列出了符合条件的 6 条记录。
```


# git github gitlib arc land 
origin/master  表示远程数据库“origin”的分支“master”的位置
origin/HEAD    表示远程数据库“origin”当前提交的位置
master         表示本地数据库分支“master”的位置

<<<<<<< 这里是发生冲突的部分  >>>>>>>>   ==分割线上方是本地数据库的内容, 下方是远程数据库的编辑内容

stash是临时保存文件修改内容的区域；stash可以暂时保存工作树和索引里还没提交的修改内容，您可以事后再取出暂存的修改，应用到原先的分支或其他的分支上

```
在Git中，用HEAD表示当前版本;上一个版本就是HEAD^，上上一个版本就是HEAD^^，当然往上100个版本写100个^比较容易数不过来，所以写成HEAD~100
```
git reset --hard HEAD^  # 回到上一个版本, 
git reflog  # 用来记录你的每一次命令,重启之后也还在

Git的版本库里存了很多东西，其中最重要的就是称为stage（或者叫index）的暂存区，还有Git为我们自动创建的第一个分支master，以及指向master的一个指针叫HEAD

![image](https://github.com/Django-27/workspace/blob/master/pic/git-diff-2.png)

git diff HEAD -- file.txt  # 命令可以查看工作区和版本库里面最新版本的区别

git checkout -- file.txt  # 可以丢弃(撤销)工作区的修改;让这个文件回到最近一次git commit或git add时的状态

git reset HEAD file.txt  # 可以把暂存区的修改撤销掉（unstage），重新放回工作区

git reset commit_id  完成Commit命令的撤销，但是不对代码修改进行撤销， 重新提交对本地代码的修改

git reset命令既可以回退版本，也可以把暂存区的修改回退到工作区。当我们用HEAD时，表示最新的版本, 再用git  status查看一下，现在暂存区是干净的
```
场景1：当你改乱了工作区某个文件的内容，想直接丢弃工作区的修改时，用命令git checkout -- file。

场景2：当你不但改乱了工作区某个文件的内容，还添加到了暂存区时，想丢弃修改，分两步，第一步用命令git reset HEAD file，就回到了场景1，第二步按场景1操作。

场景3：已经提交了不合适的修改到版本库时，想要撤销本次提交，参考版本回退一节，不过前提是没有推送到远程库。
```
要关联一个远程库，使用命令git remote add origin git@server-name:path/repo-name.git
关联后，使用命令git push -u origin master第一次推送master分支的所有内容， 第一次使用-u,Git不但会把本地的master分支内容推送的远程新的master分支，还会把本地的master分支和远程的master分支关联起来，在以后的推送或者拉取时就可以简化命令
后，每次本地提交后，只要有必要，就可以使用命令git push origin master推送最新修改

每次提交，Git都把它们串成一条时间线，这条时间线就是一个分支。截止到目前，只有一条时间线，在Git里，这个分支叫主分支，即master分支。HEAD严格来说不是指向提交，而是指向master，master才是指向提交的，所以，HEAD指向的就是当前分支

git checkout -b dev  # 创建dev分支，然后切换到dev分支
git branch命令会列出所有分支，当前分支前面会标一个*号
git merge dev_2    # 命令用于合并指定分支dev_2到当前分支
git branch -d dev_2  # 合并完成后，就可以放心地删除dev_2分支了

通常，合并分支时，如果可能，Git会用Fast forward模式，但这种模式下，删除分支后，会丢掉分支信息;如果要强制禁用Fast forward模式，Git就会在merge时生成一个新的commit，这样，从分支历史上就可以看出分支信息

git merge --no-ff -m "merge with no-ff" dev  # 准备合并dev分支，请注意--no-ff参数，表示禁用Fast forward

git stash  # Git还提供了一个stash功能，可以把当前工作现场“储藏”起来，等以后恢复现场后继续工作
git stash list  
git stash apply, 恢复，但是恢复后，stash内容并不删除，你需要用git stash drop来删除
git stash pop，恢复的同时把stash内容也删了

git branch -D feature_2  # 没有合并分支的删除是会被友情提示不要删除，-D 表示强制删除

git remote  # 查看远程版本库的信息，-v查看更详细的信息,git remote show origin

当你从远程仓库克隆时，实际上Git自动把本地的master分支和远程的master分支对应起来了，并且，远程仓库的默认名称是origin
git push origin master  # 推送分支，就是把该分支上的所有本地提交推送到远程库
git push origin dev  # 如果要推送其他分支，比如dev
master分支是主分支，因此要时刻与远程同步
dev分支是开发分支，团队所有成员都需要在上面工作，所以也需要与远程同步

git checkout -b dev origin/dev # 创建远程origin的dev分支到本地
git push origin dev  # 推送dev分支到远程

推送失败，因为你的小伙伴的最新提交和你试图推送的提交有冲突，解决办法也很简单，Git已经提示我们，先用git pull把最新的提交从origin/dev抓下来，然后，在本地合并，解决冲突，再推送
git pull  # 提示“no tracking information”，则说明本地分支和远程分支的链接关系没有创建
git branch --set-upstream dev origin/dev # git pull也失败了，原因是没有指定本地dev分支与远程origin/dev分支的链接;根据提示，设置dev和origin/dev的链接
git pull
git commit -m "merge & fix hello.py"
git push origin dev



命令git tag <name>用于新建一个标签，默认为HEAD，也可以指定一个commit id
git tag -a <tagname> -m "blablabla..."  # 可以指定标签信息
git tag -s <tagname> -m "blablabla..."  # -s用私钥签名一个标签,可以用PGP签名标签,必须首先安装gpg（GnuPG）
命令git tag可以查看所有标签
git tag -d v0.1  # 可以删除一个本地标签
git push origin --tags  # 一次性推送全部尚未推送到远程的本地标签
```
如果标签已经推送到远程，要删除远程标签就麻烦一点，先从本地删除：
git tag -d v0.9
然后，从远程删除。删除命令也是push，但是格式如下
git push origin :refs/tags/<tagname>

```
#  21 差缺补漏 
u  <--->   Ctrl+r   后悔药 VS 后悔药的后悔药
p  <--->   P        粘贴寄存器中的内容，光标后 VS 光标签
y  命令实现拷贝， y [数字] motion
r  替换光标下的字符
### 使用ipdb调试python
```
pip install ipdb
import ipdb; ipdb.set_trace() 

n(下一个)
ENTER(重复上次命令)
q(退出)
p<变量>(打印变量)  还有 pp 
c(继续)
l(查找当前位于哪里)
s(进入子程序)
r(运行直到子程序结束)
!<python 命令>
h(帮助)

pp locals()  and  pp globals()
```
git checkout -b dev origin/dev # 新建并克隆分支
git checkout -- <file> # 撤销工作区中的修改
git branch -d dev
git branch --set-upstream dev origin/dev 建立关联
git pull 本地同步

vim  插件 
Plugin 'vim-powerline'
Plugin 'molokai'                      " 配色
Plugin 'Solarized'                    " 配色

Plugin 'The-NERD-tree'
Plugin 'winmanager'
Plugin 'taglist.vim'
Plugin 'minibufexpl.vim'              " buffer插件
Plugin 'ctags.vim'  
1.在vim的normal模式下直接按wm调出界面
2.<C + h,j,k,l>可以切换各个分区
3.minibufexpl只能在多个窗口下才显示

Plugin 'L9'                           " FuzzyFinder依赖包
Plugin 'FuzzyFinder'                  " 文件快速查找
1.绑定,,作为启动
2.<C + n,p>上下翻页
3.设置的是精确匹配
4.查找的是当前目录下的文件

Plugin 'SirVer/ultisnips'             " Track the engine.
Plugin 'honza/vim-snippets'           " Snippets are separated from the engine. Add this if you want them: Plugin '

Plugin 'commentary.vim'               " 注释多行
1.gc注释一行
2.gcc注释多行


v w  y  可视 选中 加粘贴

### vim tabe tabc tabs  等
```
:tabe file  打开
:tabe file  关闭
:tabo       关闭所有的标签页
:tabs       列出所有的文件
:shell exit 直接进shell并退回
:f  new_name  重新命名 
:tabm  次序
:tabn  tabp  下一个，上一个
:n 数字/名字  完成跳转
:split + name 
:vsplit + name
:ls  当前buffer的情况
gt 跳转到下一个
```

# 22 碎片化学习 2017-8-12 12：30
### vim 中的剪贴赋值粘贴
```
:reg "          表示查看未命名的寄存器内容,存放赋值、删除的内容 
:reg +          宝石查看系统剪贴板的内容
vim支持的寄存器非常多，常用的有0-9a-zA-A+"
:reg [name]     查看指定器存器的内容

"+yy       赋值当前行到剪贴板
"+p        将剪贴板的内容粘贴到光标后
"ayy       赋值当前行到寄存器a总
"ap        将寄存器a中的内容粘贴到光标后
```
