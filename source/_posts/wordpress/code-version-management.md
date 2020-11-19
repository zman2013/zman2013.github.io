---
title: 代码版本管理 SVN Git
tags:
  - Git
  - merge
  - SVN
  - 代码版本管理
id: '164'
categories:
  - - SVN Git
date: 2016-02-28 16:57:52
---

_介绍团队协作中代码管理的基本规则及命令_ 系统开发中不可避免几个人对同一个包进行修改。 背景：代码主干(trunk/master)已经稳定，团队协作进行一般的功能迭代开发。 流程：

1.  团队每个成员基于主干最新版本建立分支。
2.  每个成员基于各自分支进行功能开发，并确保每天将主干最新代码合并到自己的分支。
3.  每个成员每完成一个功能，确保不影响正常业务的前提下，将改动的代码合并到主干。

**SVN相关命令** 1.  建立分支

//基于主干创建分支
//svn cp -m 'create branch' trunk-url new-branch-url
svn cp -m 'create branch ...' http://svn.com/svn/pay/trunk http://svn.com/svn/pay/branches/branch-01

2\. 主干 -> 分支

//切换到branch-01目录下
cd ~/workspace/pay/branch-01
//把trunk上的改动合并到branch-01
//svn merge trunk-url
svn merge http://svn.com/svn/pay/trunk
//突处理冲突，提交

3\. 分支 -> 主干

//切换到主干目录下
cd ~/workspace/pay/trunk
//把branche-01上的改动merge到trunk
//svn merge --reintegrate branch-url
svn merge --reintegrate http://svn.com/svn/pay/branches/branch-01
//处理冲突，提交

**\*** 不要用 svn merge -r XX:HEAD branch-url 其他常用命令

1\. 查看修改状态
   svn status
2. 删除分支
   svn rm -m "del branch-01" http://svn.com/svn/pay/branches/branch-01
3. 查看指定branch的不同version之间的diff（默认为本地代码对应分支）
   svn diff -r999:111 http://svn.com/svn/pay/branches/branch-01
4. 查看版本修改信息
   svn log --verbose --stop-on-copy http://svn.com/svn/pay/branches/branch-01

**Git 相关命令** 1. 建立远程分支

//建议直接通过网页UI建立分支，或者通过以下命令建立分支
//clone主干到本地
git clone https://git.com/zmannotes/pay.git
cd pay
//push到本地分支masetr到远程分支branch-01，此时如果git服务器没有远程分支，会自动创建远程分支branch-01
//git push origin <local\_branch\_name>:<remote\_branch\_name>
git push origin master:branch-01
//基于远程分支branch-01建立本地分支branch-01
git checkout -b branch-01 origin/branch-01

2. 主干 -> 分支

//切换到分支branch-01
git checkout branch-01
//合并主干到分支branch-01
git merge master
//处理冲突,提交,发布

3. 分支 -> 主干

//切换到主干
git checkout master
//合并分支branch-01改动到主干
git merge branch-01
//处理冲突,提交,发布

其他常用命令

1\. 当前修改状态
   git status
2. 删除分支
 git branch -d branch-name
3. 获取远程分支信息
   git fetch
4. 获取所有(本地和远程)分支信息 (\*表示当前正在使用的分支）
   git branch -a
5. 显示远程库映射
   git remote -v

**Git vs SVN** Q：哪个更好？ A：这个问题比较可怕，两个阵营的人吵成一锅粥，你觉得哪个好就哪个好！反正我用Git更顺手，不多说了...