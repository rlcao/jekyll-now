---
layout: post
title:  "一步一步教你做Github Pull Request代码提交"
categories: Github
tags: GitHub
author: Simple Hacker
comments: true
---
### 背景
Git想必大家都在用，但很多同学对Git使用还停留在单人作战的层次。每当有代码冲突的时候，就使出杀手锏：删除本地代库从头再来。码最近部门代码提交流程进行规范，应组织要求写了一篇Pull Request代码提交流程文档。希望能对您在git使用进阶的过程中提供一点帮助。
<!---excerpt_separator-->

废话少说上图:
![image](https://user-images.githubusercontent.com/6065072/27866830-64608c6c-61ca-11e7-9cff-1d7ff0820a1a.png)
如上图所示，一个完整的操作流程包括：
1. 创建自己Fork仓库
2. 克隆本地Git仓库
3. 修改源码文件并提交本地仓库
4. PUSH本地仓库提交到自己Fork Git仓库
5. 创建Pull Request
6. 接受Pull Request

### 创建Fork仓库
1. 点击Fork按钮
![image](https://user-images.githubusercontent.com/6065072/27866937-c60bf0a0-61ca-11e7-8462-d74a4747250a.png)
2. 选择目的用户或组
![image](https://user-images.githubusercontent.com/6065072/27866966-dd846cd0-61ca-11e7-9d42-845434b04e56.png)

### 克隆本地Git仓库
```
[vagrant@localhost git-demo-repo] $ git clone ssh://git@172.19.12.69:80/caoronglu/git-demo-repo.git
Cloning into 'git-demo-repo'...                                                           
remote: Counting objects: 6, done.                                                       
remote: Compressing objects: 100% (4/4), done.                                           
remote: Total 6 (delta 1), reused 0 (delta 0)                                             
Receiving objects: 100% (6/6), done.                                                     
Resolving deltas: 100% (1/1), done.                                                       
```

### 修改源码并提交到本地仓库
作为预防性措施，我们将代码修改到topic分枝上。这样做便于同时工作在多个任务上，每个任务修改在一个分枝上, 创建Pull Request的时候Reviewer只会看到跟当前任务相关的修改，避免产生不必要的干扰。同时方便在更改被拒绝的情况下，保持Master分支与远程Master分支一致。

1. 创建topic分支
```
[vagrant@localhost git-demo-repo]$ git checkout -b pr-demo
Switched to a new branch 'pr-demo'
```
2. 修改代码
```
[vagrant@localhost git-demo-repo]$ echo "This is an example file" >> file.txt
```
3. 添加到stage区
```
[vagrant@localhost git-demo-repo]$ git add file.txt
```
4. 提交代码到本地仓库
```
[vagrant@localhost git-demo-repo]$ git commit -m "this is for pull request"
[pr-demo be52989] this is for pull request
 1 file changed, 1 insertion(+)
```

### PUSH本地仓库提交到自己Fork Git仓库
```
[vagrant@localhost git-demo-repo]$  git push --set-upstream origin pr-demo
Counting objects: 3, done.
Delta compression using up to 2 threads.
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 303 bytes | 0 bytes/s, done.
Total 3 (delta 0), reused 0 (delta 0)
remote:
remote: To create a merge request for pr-demo, visit:
remote:   http://git.iaas.jd.com/caoronglu/git-demo-repo/merge_requests/new?merge_request%5Bsource_branch%5D=pr-demo
remote:
To ssh://172.19.12.69:80/caoronglu/git-demo-repo.git
 * [new branch]      pr-demo -> pr-demo
Branch pr-demo set up to track remote branch pr-demo from origin.                
```

### 创建Pull Request
1. 进入Pull Request页面
![image](https://user-images.githubusercontent.com/6065072/27866995-f5827106-61ca-11e7-86d7-673ae7648844.png)
2. 选择源目的分支
![image](https://user-images.githubusercontent.com/6065072/27869669-3c7b11dc-61d3-11e7-99cd-97a2657d064d.png) 
3. 填写Pull Request信息
![image](https://user-images.githubusercontent.com/6065072/27867050-1db1c9d8-61cb-11e7-810e-152df95fadd9.png) 

### 审查Pull Request代码
![image](https://user-images.githubusercontent.com/6065072/27869795-9c2f5282-61d3-11e7-84e1-6c227bd2803a.png)

### Git使用小技巧：
* **同步远程Git仓库和本地仓库**

1. 添加中心Git仓库
```
[vagrant@localhost git-demo-repo]$ git remote add upstream  ssh://git@172.19.12.69:80/iaas-ops/git-demo-repo.git
[vagrant@localhost git-demo-repo]$ git remote -v
origin  ssh://git@172.19.12.69:80/caoronglu/git-demo-repo.git (fetch)
origin  ssh://git@172.19.12.69:80/caoronglu/git-demo-repo.git (push)
upstream        ssh://git@172.19.12.69:80/iaas-ops/git-demo-repo.git (fetch)
upstream        ssh://git@172.19.12.69:80/iaas-ops/git-demo-repo.git (push)
```

2. 从远程仓库upstream拉取代码
```
[vagrant@localhost git-demo-repo]$ git pull upstream master
From ssh://172.19.12.69:80/iaas-ops/git-demo-repo
 * branch            master     -> FETCH_HEAD
Already up-to-date.
```

* **对Pull Request代码审查中的更改请求提交代码**
1. 直接提交更改到Pull Request的源分支

* **解决合并conflict**
1. 拉取Pull Requst目的分支代码到源分支
2. 解决合并过程中的冲突
3. 提交代码到本地仓库
4. Push更改到Fork仓库

* **如何减少Pull Request合并冲突**
1. 从upstream远程仓库拉取最新代码
```
[vagrant@localhost git-demo-repo]$ git pull upstream master 
```
2. rebase topic分支
```
[vagrant@localhost git-demo-repo]$ git checkout <topic branch>
[vagrant@localhost git-demo-repo]$ git rebase
```
3. 解决冲突（如果有）并提交
```
[vagrant@localhost git-demo-repo]$ git add <conflict file>
[vagrant@localhost git-demo-repo]$ git commit -m "commit message"
[vagrant@localhost git-demo-repo]$ git push origin <topic branch>
```

如果您还有什么问题，或者觉得有些技巧想分享给大家的，请在下边留言。
