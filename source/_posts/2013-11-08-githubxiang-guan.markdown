---
layout: post
title: "git相关"
date: 2013-11-08 01:10
comments: true
categories: git
---
整理来源：___ http://www.worldhello.net/gotgithub/index.html ___

__一张git操作的思维导图：__ 

* [下载](http://sgxiang.github.io/images/git.png)

_______


	$ ssh -T git@github.com    //检测连接github shh服务命令



####【clone】


	$ git clone git@github.com:gotgithub/helloworld.git
	$ git add README.md
	$ git commit -m "README for this project"
	$ git push origin master


####【新建】


	$ git init
	$ git add README.md
	$ git commit -m "readme for thisproject"
	$ git remote add origin git@github.com:sgxiang/some.git
	$ git push -u origin master


<!--more-->

####移除文件
    $rm test.txt
    $git rm.test.txt

如果删除之前修改过并且已经放到暂存区域的话，则必须要用强制删除选项 -f，以防误删除文件后丢失修改的内容。

仅删除仓库中的 `$ git rm --cached test.txt`

#####重命名文件
    
    $ git mv oldName newName
    //相当于
    //$ mv oldName newName
    //$ git rm oldName
    //$ git add newName

####【信息】


	$ git config user.name "sgxiang"
	$ git config user.email "sgxiang1992@icloud.com”

####撤销操作

	$ git reset HEAD test.txt
	
####远程仓库

	$ git remote -v  //查看
	$ git remote add name gitURL //添加


####【分支】


	$ git checkout -b mybranch  //新建分支并切换
	$ touch hello1
	$ git add hello1
	$ git commit -m "add hello1 for mybranch"
	$ git push -u origin my branch
	//$ git merge hotfix合并分支
	
![](http://git.oschina.net/progit/figures/18333fig0309-tn.png)

####【删除分支】


	$ git checkout master
	$ git branch -D mybranch    //删除本地的
	$ git push origin :mybranch    //删除远程的


####【里程碑】


轻量里程碑 `git tag <tagname> [<commit>]`

带说明的里程碑 `git tag -a <tagname> [<commit>]`

带签名的里程碑 `git tag -s <tagname> [<commit>]`

例子：

	$ touch hello1
	$ git add hello1
	$git commit -m “add hello1"
	$ git tag -m "tag on initial commit" mytag1 HEAD^
	$ git tag -m “tag on new commit” mytag2
	$ git tag mytag3
	$ git push origin refs/tags/*


删除本地里程碑  `$ git tag -d mytag3`

删除远程里程碑  `$ git push origin :mytag3`






####【创建项目主页】


在版本库中创建一个名为gh-pages的分支然后添加静态网页即可

通过sgxiang.github.io/<项目名>访问


___ 创建纯净的分支不继承master ___

[1]

	$ git checkout -b gh-pages
	$ rm .git/index
	$ printf "hello world" >index.html
	$ git add index.html
	$ git reset —hard $(echo “branch gh-pages init.” | git commit-tree $(git write-tree))  //用git底层创建根提交，重置gh-pages
	$ git push -u origin gh-pages
	
[2]

	$ git symbolic-ref HEAD refs/heads/gh-pages
	$ rm .git/index
	$ printf "hello” >index.html
	$ git add index.html
	$ git commit -m "branch gh-pages init"
	$ git push -u origin gh-pages
	
[3]

	$ git init ../helloworld-web
	$ cd ../helloworld-web
	$ printf "hello" >index.html
	$ git add index.html
	$ git commit -m "branch gh-pages init"
	$ cd ../helloworld
	$ git fetch ../helloworld-web
	$ git checkout -b gh-pages FETCH_HEAD
	$ git push -u origin gh-pages
	
[4]

在项目管理页面勾选github pages会自动创建，检出定制

	$ git fetch
	$ git checkout gh-pages


 
####【合并收到的pull request】


	$ git remote add newPull https://github.com/newPull/newP.git
	$ git fetch newP From https://github.com/newPull/newP
	$ git merge newP/master
	$ git log -graph -2
	$ git push


####【共享版本库】

在一个用户提交之后，另一个用户应该获取新提交再本地提交合并

	$ git fetch 
	$ git merge
	
___ $ git pull = git fetch + git merge ___


遇到冲突  `$ git mergetool` 工具解决

恢复  `$ git reset —hard`


___ 除了合并操作还有变基操作 ___

	$ git fetch origin  //获取远程版本库的提交到本地的远程分支
	$ git rebase origin/master //将本地master分支的提交变基到新的远程分支中
	$ git push 
	// git pull —rebase
	// git config branch.master.rebase true
	// git config —global branch.autosetuprebase true




