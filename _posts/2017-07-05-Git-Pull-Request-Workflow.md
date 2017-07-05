GitLab代码提交流程
如上图所示，一个完整的操作流程包括：

	1. 创建自己Fork仓库
	2. 克隆到本地Git仓库
	3. 修改源码文件并提交本地仓库
	4. PUSH本地仓库提交到自己Fork Git仓库
	5. 创建Pull Request
	6. 接受Pull Request


创建Fork仓库

	1. 点击Fork按钮


	1. 选择目的用户或组

克隆本地Git仓库
[vagrant@localhost git-demo-repo] $ git clone ssh://git@172.19.12.69:80/caoronglu/git-demo-repo.git
Cloning into 'git-demo-repo'...                                                           
remote: Counting objects: 6, done.                                                       
remote: Compressing objects: 100% (4/4), done.                                           
remote: Total 6 (delta 1), reused 0 (delta 0)                                             
Receiving objects: 100% (6/6), done.                                                     
Resolving deltas: 100% (1/1), done.                                                       
修改源码并提交到本地仓库

	1. 创建topic分支

[vagrant@localhost git-demo-repo]$ git checkout -b pr-demo
Switched to a new branch 'pr-demo'

	1. 修改代码


[vagrant@localhost git-demo-repo]$ echo "This is an example file" >> file.txt

	1. 添加到stage区


$ git add file.txt

	1. 提交代码到本地仓库


[vagrant@localhost git-demo-repo]$ git commit -m "this is for pull request"
[pr-demo be52989] this is for pull request
 1 file changed, 1 insertion(+)
PUSH本地仓库提交到自己Fork Git仓库
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
创建Pull Request

	1. 进入Pull Request页面


	1. 选择源目的分支



	1. 填写Pull Request信息

审查Pull Request代码

Git Pull Requst小技巧：

	* 如何同步远程Git仓库和本地仓库？
	* 
		* 添加中心Git仓库                                                                                                 


[vagrant@localhost git-demo-repo]$ git remote add upstream  ssh://git@172.19.12.69:80/iaas-ops/git-demo-repo.git
[vagrant@localhost git-demo-repo]$ git remote -v                                                               
origin  ssh://git@172.19.12.69:80/caoronglu/git-demo-repo.git (fetch)                                           
origin  ssh://git@172.19.12.69:80/caoronglu/git-demo-repo.git (push)                                           
upstream        ssh://git@172.19.12.69:80/iaas-ops/git-demo-repo.git (fetch)                                   
upstream        ssh://git@172.19.12.69:80/iaas-ops/git-demo-repo.git (push)                                     

	* 
		* 从远程仓库upstream拉取代码


[vagrant@localhost git-demo-repo]$ git pull upstream master
From ssh://172.19.12.69:80/iaas-ops/git-demo-repo
 * branch            master     -> FETCH_HEAD
Already up-to-date.

	* 如何针对Pull Request代码审查中的更改请求提交代码？
	* 
		* 直接提交更改到Pull Request的源分支

	* 如何处理合并conflict？
	* 
		* 拉取Pull Requst目的分支代码到源分支
		* 解决合并过程中的冲突
		* 提交代码到本地仓库
		* Push更改到Fork仓库

	* 如何减少Pull Request合并冲突？
	* 
		* 从upstream远程仓库拉取最新代码


[vagrant@localhost git-demo-repo]$ git pull upstream master 

	* 
		* rebase topic分支


[vagrant@localhost git-demo-repo]$ git checkout <topic branch>
[vagrant@localhost git-demo-repo]$ git rebase

	* 
		* 解决冲突（如果有）并提交


[vagrant@localhost git-demo-repo]$ git add <conflict file>
[vagrant@localhost git-demo-repo]$ git commit -m "commit message"
[vagrant@localhost git-demo-repo]$ git push origin <topic branch>
