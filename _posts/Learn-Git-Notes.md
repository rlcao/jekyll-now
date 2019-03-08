---
layout: post
title:  "Git Recap Notes"
categories: Git
tags: Git
author: Simple Hacker
comments: true
---

# Topics
* Git Architecture
* Git Internals
* Git Basics
* Git Advanced
* Git Tips

# Git Architecure
## Distributed Version Control System
![Any Repo Can Be A Server](https://user-images.githubusercontent.com/6065072/53620694-d9198000-3c2e-11e9-83fc-bebb63f64fec.png)
## The Git Three-Tree Architecture
**File in 3 States**
* unmodified(comitted)
* modified
* staged
**staging area**
It is the place where git prepares a tree before it:
* writes it into the repository or
* checks it out into a working directory
When you run git status git makes two comparisons:
* it compares index file with current working directory — changes reported as “not staged for commit”
* it compares index file with HEAD commit — changes reported as “to be committed”
![3 Areas in Git](https://user-images.githubusercontent.com/6065072/53620764-18e06780-3c2f-11e9-8f8a-493e9de51f88.png)

## A Key-Value Database and DAG 
**Git Object Types**
1. blob
2. tree
3. commit

**A commit is a wrapper around the tree object that:**
* allows attaching a message to a tree (group of files)
* allows specifying parent (commit)

**Commit Chain in Git**
![Commit Chains in Git](https://user-images.githubusercontent.com/6065072/53621193-ae302b80-3c30-11e9-8fbb-760ab6ff9143.png)
# Git Internals
### Hard Way to Commit in Git
**git internal commands to be used:**
```bash
## git hash-object
## git cat-file
## git mktree
## git read-tree
## git update-index
## git write-tree
## git commit-tree
```

1. Prepare Monitor Windows
* monitor the file change in working directory:
```
while true; do clear; tree -C; sleep 1; done;
```
* monitor staging area change:
```
while true; do clear; git ls-files -s; sleep 1; done;
```
* monitor repository change by:
```
while true; do clear; git lg; done;
#lg is an git alias command
```
2. Create file blob
```
$ echo "file #1 content" | git hash-object -t blob -w --stdin
6d8e5bd086a1866206ce369a622d4c2bc73cf95a
$ git cat-file -p 6d8e5bd086a1866206ce369a622d4c2bc73cf95a
file #1 content
$ tree .git/objects/6d
.git/objects/6d
└── 8e5bd086a1866206ce369a622d4c2bc73cf95a
0 directories, 1 file
$ file .git/objects/6d/8e5bd086a1866206ce369a622d4c2bc73cf95a
.git/objects/6d/8e5bd086a1866206ce369a622d4c2bc73cf95a: VAX COFF executable not stripped
```
3. Create tree object
```
# 3.1 Create a tree object
$ printf "%s %s %s\t%s\n" 100644 blob 6d8e5bd086a1866206ce369a622d4c2bc73cf95a file1.txt | git mktree
3c7cae50b5979d7c53562024994e1efdee80df1b
$ tree .git/objects
.git/objects
├── 3c
│   └── 7cae50b5979d7c53562024994e1efdee80df1b
├── 6d
    └── 8e5bd086a1866206ce369a622d4c2bc73cf95a
$ git cat-file -p 3c7cae50b5979d7c53562024994e1efdee80df1b
100644 blob 6d8e5bd086a1866206ce369a622d4c2bc73cf95a	file1.txt
# 3.2 create nested folder named as "nested-folder"(optional)
printf "%s %s %s\t%s\n" 040000 tree 3c7cae50b5979d7c53562024994e1efdee80df1b nested-folder | git mktree
581553e3af9e4d5635e1d72b0d9ec1d1666a8162
$ tree .git/objects
.git/objects
├── 3c
│   └── 7cae50b5979d7c53562024994e1efdee80df1b
├── 58
│   └── 1553e3af9e4d5635e1d72b0d9ec1d1666a8162
├── 6d
│   └── 8e5bd086a1866206ce369a622d4c2bc73cf95a
```
4. Load tree in staging area

**how index works:**

index is not only to support a staging area, but also to facilitate the ability of Git to detect changes to files in your working directory; to me-diate the branch-merge process, so you can resolve conflicts on a file-by-file basis and safely abort the merge at any time; and to convert staged files and folders into tree objects whose references are written to the next commit object. Git also uses the index to retain information about files in the working tree and about objects retrieved from the DAG—and thus further leveraging the index as a type of cache.

**git ls-files -s stage number usage**

The different stage numbers are not really used during git-add command. They are used for handling merge conflicts. In a nutshell:

- Slot 0: “normal”, un-conflicted, all-is-well entry.
- Slot 1: “base”, the common ancestor version.
- Slot 2: “ours”, the target (HEAD) version.
- Slot 3: “theirs”, the being-merged-in version.

Git index, or Git cache, has 3 important properties:
The index contains all the information necessary to generate a single (uniquely determined) tree object.
The index enables fast comparisons between the tree object it defines and the working tree.
It can efficiently represent information about merge conflicts between different tree objects, allowing each pathname to be associated with sufficient information about the trees involved that you can create a three-way merge between them.
```
$ git read-tree 581553e3af9e4d5635e1d72b0d9ec1d1666a8162
$ git ls-files -s
100644 6d8e5bd086a1866206ce369a622d4c2bc73cf95a 0	nested-folder/file1.txt
$ git st
Changes to be committed:
  (use "git rm --cached <file>..." to unstage)
	new file:   nested-folder/file1.txt
Changes not staged for commit:
  (use "git add/rm <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)
	deleted:    nested-folder/file1.txt
$ git checkout -- nested-folder/file1.txt
$ git st
On branch master
No commits yet
Changes to be committed:
  (use "git rm --cached <file>..." to unstage)
	new file:   nested-folder/file1.txt
```
```
# 3.1 git add will change the index/stage area
$ rm -rf nested-folder/
$ git add .
$ git st
On branch master
No commits yet
nothing to commit (create/copy files and use "git add" to track)
# 3.2 restore the working directory
$ git read-tree 581553e3af9e4d5635e1d72b0d9ec1d1666a8162
$ git st
On branch master
No commits yet
Changes to be committed:
  (use "git rm --cached <file>..." to unstage)
	new file:   nested-folder/file1.txt
Changes not staged for commit:
  (use "git add/rm <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)
	deleted:    nested-folder/file1.txt
dhcp-10-191-9-156:learn-git ronglucao$ git checkout-index -a
dhcp-10-191-9-156:learn-git ronglucao$ tree
.
└── nested-folder
    └── file1.txt
1 directory, 1 file
```

5. Add another file
```
$ echo "file #2 content" >file2.txt
$ git st
On branch master
No commits yet
Changes to be committed:
  (use "git rm --cached <file>..." to unstage)
	new file:   nested-folder/file1.txt
Untracked files:
  (use "git add <file>..." to include in what will be committed)
	file2.txt
$ git add file2.txt
$ git ls-files -s
100644 f5567f67eafaafd88182bb2c68d8f18c6d472489 0	file2.txt
100644 6d8e5bd086a1866206ce369a622d4c2bc73cf95a 0	nested-folder/file1.txt
# 5. create tree object using index contents
$ git write-tree
1e0f7b32ff6e11f0da74ac3f02865f46193e2031
$ git st
On branch master
No commits yet
Changes to be committed:
  (use "git rm --cached <file>..." to unstage)
	new file:   file2.txt
	new file:   nested-folder/file1.txt
# That’s because although we’ve created and saved our tree into the repository we didn’t update the HEAD file that is used for comparison. Since we can only put a commit hash or a branch reference into the HEAD file but have neither now we will leave HEAD file as is for now.	
```

6. Create git commit object
```
$ echo "initial commit" | git commit-tree 581553e3af9e4d5635e1d72b0d9ec1d1666a8162
08d48d2429ab843d2da6508c119d22944b34812d
```
7. Update HEAD ref
```
$ cat .git/HEAD
ref: refs/heads/master
$ echo 08d48d2429ab843d2da6508c119d22944b34812d >>.git/refs/heads/master
$ git st
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)
	new file:   file2.txt
# 6. A branch is a text file pointing to a commit
$ git checkout -b dev
A	file2.txt
Switched to a new branch 'dev'
$ cat .git/refs/heads/dev
08d48d2429ab843d2da6508c119d22944b34812d
```
**The last git commit command as we already know performs multiple under-the-hood operations:**
* creates a tree from the index file
* writes this tree to the repository
* creates a commit object that wraps this tree
* sets the initial commit as a parent of the new commit since it’s the commit that we currently have in the HEAD file
* update corresponding refs/heads/<branch>
```
$ git commit -m "second commit file2.txt"
[dev a0668c1] second commit file2.txt
 1 file changed, 1 insertion(+)
 create mode 100644 file2.txt
$ cat .git/refs/
heads/ tags/
dhcp-10-191-9-156:learn-git ronglucao$ cat .git/refs/heads/
dev     master
dhcp-10-191-9-156:learn-git ronglucao$ cat .git/refs/heads/dev
a0668c170724be6785e22ff299c843f28e100615
$ git st
On branch dev
nothing to commit, working tree clean
$ git ls-files -s
100644 f5567f67eafaafd88182bb2c68d8f18c6d472489 0	file2.txt
100644 6d8e5bd086a1866206ce369a622d4c2bc73cf95a 0	nested-folder/file1.txt
# below commit will create 3 additional objects: comitting nested-folder/file2.txt
dhcp-10-191-9-156:learn-git ronglucao$ git ls-files -s
100644 f5567f67eafaafd88182bb2c68d8f18c6d472489 0	file2.txt
100644 6d8e5bd086a1866206ce369a622d4c2bc73cf95a 0	nested-folder/file1.txt
100644 f5567f67eafaafd88182bb2c68d8f18c6d472489 0	nested-folder/file2.txt
$ git commit -m "checkin #3 adding another file in nested-folder"
[dev 7eafe14] checkin #3 adding another file in nested-folder
 1 file changed, 1 insertion(+)
 create mode 100644 nested-folder/file2.txt
$ git lg
* 7eafe14 - (20 minutes ago) checkin #3 adding another file in nested-folder - rlcao (HEAD -> dev)
* a0668c1 - (29 minutes ago) second commit file2.txt - rlcao
* 08d48d2 - (67 minutes ago) initial commit - rlcao (tag: master-tag, master)
```
**A tag is a text file pointing to a commit**
```
# create a tag using tag command
$ git tag master-tag master
$ diff .git/refs/heads/master .git/refs/tags/master-tag
```

## Git Basics
### Git Initial Setup
**Configuration in 3 Levels**
1. /etc/gitconfig
```
$ git config --system user.name "Ronglu Cao"
```
2. ~/.gitconfig
```
$ git config --global user.name "Ronglu Cao"
```
3. <gitrepo>/.git/config
```
$ git config --local user.name "Ronglu Cao"
```
**Here is my settings**
```
git config --list
user.email=caoronglu@gmail.com
user.name=rlcao
alias.co=checkout
alias.br=branch
alias.ci=commit
alias.st=status
alias.lg=log --graph --abbrev-commit --decorate --format=format:'%C(bold blue)%h%C(reset) - %C(bold green)(%ar)%C(reset) %C(white)%s%C(reset) %C(dim white)- %an%C(reset)%C(bold yellow)%d%C(reset)' --all
......
```
### Clone from Remote Repository
**Clone will download all objects in remote repository, but only update master references**
```
$ git clone https://ronglu.cao%40oracle.com@alm.oraclecorp.com/sresyseng/s/sresyseng_sre-ops-scripts_12926/scm/learn-git.git .
Cloning into '.'...
remote: Counting objects: 10, done
remote: Finding sources: 100% (10/10)
remote: Getting sizes: 100% (7/7)
remote: Total 10 (delta 0), reused 10 (delta 0)
Unpacking objects: 100% (10/10), done.
dhcp-10-191-9-156:learn-git-clone ronglucao$ git branch
* master
$ tree .git
.git
├── HEAD
├── config
├── description
├── hooks
│   ├── applypatch-msg.sample
│   ├── commit-msg.sample
│   ├── fsmonitor-watchman.sample
│   ├── post-update.sample
│   ├── pre-applypatch.sample
│   ├── pre-commit.sample
│   ├── pre-push.sample
│   ├── pre-rebase.sample
│   ├── pre-receive.sample
│   ├── prepare-commit-msg.sample
│   └── update.sample
├── index
├── info
│   └── exclude
├── logs
│   ├── HEAD
│   └── refs
│       ├── heads
│       │   └── master
│       └── remotes
│           └── origin
│               └── HEAD
├── objects
│   ├── 08
│   │   └── d48d2429ab843d2da6508c119d22944b34812d
│   ├── 1e
│   │   └── 0f7b32ff6e11f0da74ac3f02865f46193e2031
│   ├── 3c
│   │   └── 7cae50b5979d7c53562024994e1efdee80df1b
│   ├── 58
│   │   └── 1553e3af9e4d5635e1d72b0d9ec1d1666a8162
│   ├── 6d
│   │   └── 8e5bd086a1866206ce369a622d4c2bc73cf95a
│   ├── 7e
│   │   └── afe1486fc8c8c2ed0ddc05f369bdd7e0f4ae5c
│   ├── a0
│   │   └── 668c170724be6785e22ff299c843f28e100615
│   ├── b5
│   │   └── ce29581df92620aa72a2a8262f7c566586b7d8
│   ├── b7
│   │   └── 95d6d51f3c0172eb29ef260d57e944cde807cf
│   ├── f5
│   │   └── 567f67eafaafd88182bb2c68d8f18c6d472489
│   ├── info
│   └── pack
├── packed-refs
└── refs
    ├── heads
    │   └── master
    ├── remotes
    │   └── origin
    │       └── HEAD
    └── tags
25 directories, 32 files
```
### Pull from Remote Repository
```
$ git pull origin dev
From https://<gitserver>/learn-git
 * branch            dev        -> FETCH_HEAD
Updating 08d48d2..7eafe14
Fast-forward
 file2.txt               | 1 +
 nested-folder/file2.txt | 1 +
 2 files changed, 2 insertions(+)
 create mode 100644 file2.txt
 create mode 100644 nested-folder/file2.txt
# Above command is equavilent to below two commands on current branch:
$ git fetch origin dev && git merge dev
```
### Switch to another branch
index is used to keep track of the file changes over the three areas: working directory, staging area, and repository. And when you add changes to your staging area, git updates the information in the index about those changes and creates new blob objects, but puts them in the same .git/objects directory with all the other blobs that belong to previous commits.
```
$ git ls-files -s
$ git checkout dev
$ git ls-files -s
```

### Commit to Local Repository
```
# create a file named as file3.txt
$ echo "file #3 content" >> file3.txt
# below command will create blob && update index list
$ git add file3.txt
$ git hash-object file3.txt
713df13899674585b05a829600002d8a87759acc
$ git ls-files -s
100644 713df13899674585b05a829600002d8a87759acc 0	file3.txt
100644 6d8e5bd086a1866206ce369a622d4c2bc73cf95a 0	nested-folder/file1.txt
# commit operation will create required git tree objects and the commit object
$ git commit -m "adding file3.txt"
[master 08ebda4] adding file3.txt
 1 file changed, 1 insertion(+)
 create mode 100644 file3.txt
$ git cat-file -p 08ebda465f82106f890049ee44948a48801d1409
tree 7910da30f6480fb313a87c1639b3da9e709db0a2
parent 08d48d2429ab843d2da6508c119d22944b34812d
author rlcao <caoronglu@gmail.com> 1551456996 +0800
committer rlcao <caoronglu@gmail.com> 1551456996 +0800

adding file3.txt
# So 3 more objects were added in git objects, in total.
## 3.1 file blob object
## 3.2 new tree object
## 3.3 new commit object
```

### Push to Remote Repository
**Updates remote refs using local refs, while sending objects necessary to complete the given refs.**

1. add remote branch
```
git remote add origin https://ronglu.cao%40oracle.com@alm.oraclecorp.com/sresyseng/s/sresyseng_sre-ops-scripts_12926/scm/learn-git.git
```
2. push to remote branch
```
$ git push --set-upstream origin master
fatal: Authentication failed for 'https://<gitserver>/learn-git.git/'
$ git remote show origin
Password for 'https://<gitserver>':
* remote origin
  Fetch URL: https://<gitserver>/learn-git.git
  Push  URL: https://<gitserver>/learn-git.git
  HEAD branch: (unknown)
$ git push --set-upstream origin master
Counting objects: 4, done.
Writing objects: 100% (4/4), 272 bytes | 272.00 KiB/s, done.
Total 4 (delta 0), reused 0 (delta 0)
remote: Updating references: 100% (1/1)
To https://<gitserver>/learn-git.git
 * [new branch]      master -> master
Branch 'master' set up to track remote branch 'master' from 'origin'.
```

### 3 Way Merge
1. Before merge
```
* 08ebda4 - (13 minutes ago) adding file3.txt - rlcao (HEAD -> master)
| * 7eafe14 - (6 hours ago) checkin #3 adding another file in nested-folder - rlcao (origin/dev, dev)
| * a0668c1 - (7 hours ago) second commit file2.txt - rlcao
|/
* 08d48d2 - (7 hours ago) initial commit - rlcao (tag: master-tag, origin/master)
```
2. Merge
```
$ git merge dev
Merge made by the 'recursive' strategy.
 file2.txt               | 1 +
 nested-folder/file2.txt | 1 +
 2 files changed, 2 insertions(+)
 create mode 100644 file2.txt
 create mode 100644 nested-folder/file2.txt
```
3. After merge
```
$ git lg
*   2680cf4 - (12 seconds ago) Merge branch 'dev' - rlcao (HEAD -> master)
|\
| * 7eafe14 - (6 hours ago) checkin #3 adding another file in nested-folder - rlcao (origin/dev, dev)
| * a0668c1 - (7 hours ago) second commit file2.txt - rlcao
* | 08ebda4 - (14 minutes ago) adding file3.txt - rlcao
|/
* 08d48d2 - (7 hours ago) initial commit - rlcao (tag: master-tag, origin/master)
$ git cat-file -p 2680cf4
tree 9a0346ff8121b6f2def8e540d530ab18513325a3
parent 08ebda465f82106f890049ee44948a48801d1409
parent 7eafe1486fc8c8c2ed0ddc05f369bdd7e0f4ae5c
author rlcao <caoronglu@gmail.com> 1551457799 +0800
committer rlcao <caoronglu@gmail.com> 1551457799 +0800
Merge branch 'dev'
```
4. In case of conflict
```
# 0: before merge
$ git lg
* 5bb0eb5 - (7 seconds ago) checkin on file2.txt on master - rlcao (HEAD -> master)
* c5ad138 - (2 minutes ago) from master branch on file3.txt - rlcao
*   2680cf4 - (22 hours ago) Merge branch 'dev' - rlcao
|\
* | 08ebda4 - (22 hours ago) adding file3.txt - rlcao
| | * ba03503 - (51 seconds ago) on dev on file2.txt - rlcao (dev)
| |/
| * 7eafe14 - (28 hours ago) checkin #3 adding another file in nested-folder - rlcao (origin/dev)
| * a0668c1 - (28 hours ago) second commit file2.txt - rlcao
|/
* 08d48d2 - (29 hours ago) initial commit - rlcao (tag: master-tag, origin/master)
$ git branch
  dev
* master
# 1: merge
$ git merge dev
Auto-merging file2.txt
CONFLICT (content): Merge conflict in file2.txt
Automatic merge failed; fix conflicts and then commit the result.
dhcp-10-191-9-156:learn-git ronglucao$ git st
On branch master
Your branch is ahead of 'origin/master' by 6 commits.
  (use "git push" to publish your local commits)

You have unmerged paths.
  (fix conflicts and run "git commit")
  (use "git merge --abort" to abort the merge)

Unmerged paths:
  (use "git add <file>..." to mark resolution)

	both modified:   file2.txt

no changes added to commit (use "git add" and/or "git commit -a")
$ git ls-files -s
100644 f5567f67eafaafd88182bb2c68d8f18c6d472489 1	file2.txt
100644 49fc971e0603490e830c8d74994a95c544edb3c0 2	file2.txt
100644 71d466510312a0d004cc0e8859252b6cfd87c088 3	file2.txt
100644 c1cdc153e4906db6737749b38eade43b1134738d 0	file3.txt
100644 6d8e5bd086a1866206ce369a622d4c2bc73cf95a 0	nested-folder/file1.txt
100644 f5567f67eafaafd88182bb2c68d8f18c6d472489 0	nested-folder/file2.txt
$ git cat-file -p f5567f67eafaafd88182bb2c68d8f18c6d472489
file #2 content
$ git cat-file -p 49fc971e0603490e830c8d74994a95c544edb3c0
file #2 content on master
$ git cat-file -p 71d466510312a0d004cc0e8859252b6cfd87c088
file #2 content updated on dev
# 2: resolve conflict by taking ours
$ git checkout --ours file2.txt
$ git add file2.txt
$ git ls-files -s
100644 49fc971e0603490e830c8d74994a95c544edb3c0 0	file2.txt
100644 c1cdc153e4906db6737749b38eade43b1134738d 0	file3.txt
100644 6d8e5bd086a1866206ce369a622d4c2bc73cf95a 0	nested-folder/file1.txt
100644 f5567f67eafaafd88182bb2c68d8f18c6d472489 0	nested-folder/file2.txt
dhcp-10-191-9-156:learn-git ronglucao$ git st
On branch master
Your branch is ahead of 'origin/master' by 6 commits.
  (use "git push" to publish your local commits)

All conflicts fixed but you are still merging.
  (use "git commit" to conclude merge)
$ ls -lrth MERGE*
-rw-r--r--  1 ronglucao  staff    41B Mar  2 22:17 MERGE_HEAD
-rw-r--r--  1 ronglucao  staff     0B Mar  2 22:17 MERGE_MODE
-rw-r--r--  1 ronglucao  staff    45B Mar  2 22:17 MERGE_MSG
$ cat MERGE_HEAD
ba035037624cf04ca394df0c1c32638ccebf3fb8
$ cat MERGE_MSG
Merge branch 'dev'

# Conflicts:
#	file2.txt
dhcp-10-191-9-156:.git ronglucao$ git cat-file -p ba035037624cf04ca394df0c1c32638ccebf3fb8
tree e86545711c011508a99267d0e449e324ccdf3eb1
parent 7eafe1486fc8c8c2ed0ddc05f369bdd7e0f4ae5c
author rlcao <caoronglu@gmail.com> 1551536168 +0800
committer rlcao <caoronglu@gmail.com> 1551536168 +0800
# 3. commit to write merge into repository
$ git commit -m "merge from dev"
[master c01e043] merge from dev
$ ls -lrth MERGE*
ls: MERGE*: No such file or directory
$ git lg
*   c01e043 - (4 seconds ago) merge from dev - rlcao (HEAD -> master)
|\
| * ba03503 - (9 minutes ago) on dev on file2.txt - rlcao (dev)
* | 5bb0eb5 - (8 minutes ago) checkin on file2.txt on master - rlcao
* | c5ad138 - (10 minutes ago) from master branch on file3.txt - rlcao
* |   2680cf4 - (22 hours ago) Merge branch 'dev' - rlcao
|\ \
| |/
| * 7eafe14 - (28 hours ago) checkin #3 adding another file in nested-folder - rlcao (origin/dev)
| * a0668c1 - (28 hours ago) second commit file2.txt - rlcao
* | 08ebda4 - (22 hours ago) adding file3.txt - rlcao
|/
* 08d48d2 - (29 hours ago) initial commit - rlcao (tag: master-tag, origin/master)
```

### checkout files from commit
```
$ git checkout ba03503 -- file2.txt
$ git st
On branch master
Your branch is ahead of 'origin/master' by 8 commits.
  (use "git push" to publish your local commits)

Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

	modified:   file2.txt

$ cat file2.txt
file #2 content updated on dev
dhcp-10-191-9-156:learn-git ronglucao$ git lg
*   c01e043 - (22 hours ago) merge from dev - rlcao (HEAD -> master)
|\
| * ba03503 - (22 hours ago) on dev on file2.txt - rlcao (dev)
* | 5bb0eb5 - (22 hours ago) checkin on file2.txt on master - rlcao
* | c5ad138 - (22 hours ago) from master branch on file3.txt - rlcao
* |   2680cf4 - (2 days ago) Merge branch 'dev' - rlcao
|\ \
| |/
| * 7eafe14 - (2 days ago) checkin #3 adding another file in nested-folder - rlcao (origin/dev)
| * a0668c1 - (2 days ago) second commit file2.txt - rlcao
* | 08ebda4 - (2 days ago) adding file3.txt - rlcao
|/
* 08d48d2 - (2 days ago) initial commit - rlcao (tag: master-tag, origin/master)
```

### Change commit history
**Note: only change history not pushed to any other repository**
1. change latest commit on current branch
```
# commit --amend to change commit message
$ git commit --amend
# above command is equivalent to below commands:
 $ git reset --soft HEAD^
$ ... do something else to come up with the right tree ...
$ git commit -c ORIG_HEAD
```
2. change some file to its older state
```
$ git checkout <commit> -- <filepath>
```
3. rebase as merge
```
$ git lg
* 76d3cb1 - (2 seconds ago) on master to test rebase - rlcao (HEAD -> master)
| * 716f7b2 - (45 seconds ago) on dev to test rebase - rlcao (dev)
|/
*   9c5183b - (23 hours ago) merge from dev - rlcao
|\
| * ba03503 - (23 hours ago) on dev on file2.txt - rlcao
* | 5bb0eb5 - (23 hours ago) checkin on file2.txt on master - rlcao
* | c5ad138 - (23 hours ago) from master branch on file3.txt - rlcao
* |   2680cf4 - (2 days ago) Merge branch 'dev' - rlcao
|\ \
| |/
| * 7eafe14 - (2 days ago) checkin #3 adding another file in nested-folder - rlcao (origin/dev)
| * a0668c1 - (2 days ago) second commit file2.txt - rlcao
* | 08ebda4 - (2 days ago) adding file3.txt - rlcao
|/
* 08d48d2 - (2 days ago) initial commit - rlcao (tag: master-tag, origin/master)
$ git checkout dev
Switched to branch 'dev'
$ git rebase master
First, rewinding head to replay your work on top of it...
Applying: on dev to test rebase
dhcp-10-191-9-156:learn-git ronglucao$ git lg
* ad5fe2f - (3 minutes ago) on dev to test rebase - rlcao (HEAD -> dev)
* 76d3cb1 - (3 minutes ago) on master to test rebase - rlcao (master)
*   9c5183b - (23 hours ago) merge from dev - rlcao
|\
| * ba03503 - (23 hours ago) on dev on file2.txt - rlcao
* | 5bb0eb5 - (23 hours ago) checkin on file2.txt on master - rlcao
* | c5ad138 - (23 hours ago) from master branch on file3.txt - rlcao
* |   2680cf4 - (2 days ago) Merge branch 'dev' - rlcao
|\ \
| |/
| * 7eafe14 - (2 days ago) checkin #3 adding another file in nested-folder - rlcao (origin/dev)
| * a0668c1 - (2 days ago) second commit file2.txt - rlcao
* | 08ebda4 - (2 days ago) adding file3.txt - rlcao
|/
* 08d48d2 - (2 days ago) initial commit - rlcao (tag: master-tag, origin/master)
```

### Save workspace
```
$ git st
On branch dev
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

	modified:   file2.txt

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

	modified:   file2.txt

:learn-git ronglucao$ git ls-files -s
100644 5fd20de86a3eff4f9073323d3708b4044750d7c8 0	file2.txt
100644 eabc866d2d3292a618ff47b8a622990eca6c1ee2 0	file3.txt
100644 6d8e5bd086a1866206ce369a622d4c2bc73cf95a 0	nested-folder/file1.txt
100644 f5567f67eafaafd88182bb2c68d8f18c6d472489 0	nested-folder/file2.txt
$ git cat-file -p 5fd20de86a3eff4f9073323d3708b4044750d7c8
file #2 content updated new content on master on dev testing stashing
$ cat file2.txt
file #2 content updated new content on master on dev testing stashing
adding another line
$ git stash
Saved working directory and index state WIP on dev: ad5fe2f on dev to test rebase
$ git stash list
stash@{0}: WIP on dev: ad5fe2f on dev to test rebase
$ git st
On branch dev
nothing to commit, working tree clean
$ git ls-files -s
100644 7f3051ca995cb84c4ade87289872ac9f95aa42e4 0	file2.txt
100644 eabc866d2d3292a618ff47b8a622990eca6c1ee2 0	file3.txt
100644 6d8e5bd086a1866206ce369a622d4c2bc73cf95a 0	nested-folder/file1.txt
100644 f5567f67eafaafd88182bb2c68d8f18c6d472489 0	nested-folder/file2.txt
$ git cat-file -p 7f3051ca995cb84c4ade87289872ac9f95aa42e4
file #2 content updated new content on master
$ git stash pop # this will restore the workspace
```

### Update references
```
dhcp-10-191-9-156:heads ronglucao$ git lg
*   9c5183b - (23 hours ago) merge from dev - rlcao (HEAD -> master, dev)
|\
| * ba03503 - (23 hours ago) on dev on file2.txt - rlcao
* | 5bb0eb5 - (23 hours ago) checkin on file2.txt on master - rlcao
* | c5ad138 - (23 hours ago) from master branch on file3.txt - rlcao
* |   2680cf4 - (2 days ago) Merge branch 'dev' - rlcao
|\ \
| |/
| * 7eafe14 - (2 days ago) checkin #3 adding another file in nested-folder - rlcao (origin/dev)
| * a0668c1 - (2 days ago) second commit file2.txt - rlcao
* | 08ebda4 - (2 days ago) adding file3.txt - rlcao
|/
* 08d48d2 - (2 days ago) initial commit - rlcao (tag: master-tag, origin/master)
dhcp-10-191-9-156:heads ronglucao$ git update-ref refs/heads/dev ba03503
dhcp-10-191-9-156:heads ronglucao$ git lg
*   9c5183b - (23 hours ago) merge from dev - rlcao (HEAD -> master)
|\
| * ba03503 - (23 hours ago) on dev on file2.txt - rlcao (dev)
* | 5bb0eb5 - (23 hours ago) checkin on file2.txt on master - rlcao
* | c5ad138 - (23 hours ago) from master branch on file3.txt - rlcao
* |   2680cf4 - (2 days ago) Merge branch 'dev' - rlcao
|\ \
| |/
| * 7eafe14 - (2 days ago) checkin #3 adding another file in nested-folder - rlcao (origin/dev)
| * a0668c1 - (2 days ago) second commit file2.txt - rlcao
* | 08ebda4 - (2 days ago) adding file3.txt - rlcao
|/
* 08d48d2 - (2 days ago) initial commit - rlcao (tag: master-tag, origin/master)
```
### Getting Git on a Server
```
# 1. Export an existing repository into a new bare repository
$ git clone --bare my_project my_project.git
Cloning into bare repository 'my_project.git'...
done.
# above command is equivalent to below command:
$ cp -Rf my_project/.git my_project.git

# 2. Putting the Bare Repository on a Server
scp -r my_project.git user@git.example.com:/srv/git
# 2.1 At this point, other users who have SSH-based read access to the /srv/git directory on that server can clone your repository by running
$ git clone user@git.example.com:/srv/git/my_project.git
# 2.2 If a user SSHs into a server and has write access to the /srv/git/my_project.git directory, they will also automatically have push access.
```

# Git Advanced
### Git workflows
1. integration-manager
![image](https://user-images.githubusercontent.com/6065072/53697336-526dca00-3e0b-11e9-9e25-4de62cd0933e.png)
2. Dictator and Lieutenants Workflow
![image](https://user-images.githubusercontent.com/6065072/53697416-c4461380-3e0b-11e9-8d44-85be2cba1166.png)

### Git reset
**Current Checkin Chain**
A <- B <- C(current reference)
git reset <option> B
	
| Git Reset Option | Repo | Index | WorkingDirectory|
| ------ | ------ | ------ | ------ |
| --soft| B | C |C|
| --mixed(default)| B | B |C|
| --soft| B|B|B|

![image](https://user-images.githubusercontent.com/6065072/53697482-7b428f00-3e0c-11e9-8b57-8f032c7e1ff8.png)

### reflog to record ref history
```
$ git reflog master
c01e043 (HEAD -> master) master@{0}: commit (merge): merge from dev
5bb0eb5 master@{1}: commit: checkin on file2.txt on master
c5ad138 master@{2}: commit: from master branch on file3.txt
2680cf4 master@{3}: merge dev: Merge made by the 'recursive' strategy.
08ebda4 master@{4}: commit: adding file3.txt
```
# References
* How merge works - https://git-scm.com/book/en/v2/Git-Tools-Advanced-Merging
* Switch to another branch - https://hackernoon.com/understanding-git-index-4821a0765cf
* How index works - 
1. https://msdn.microsoft.com/en-us/magazine/mt493250.aspx
2. https://stackoverflow.com/questions/21309490/how-do-contents-of-git-index-evolve-during-a-merge-and-whats-in-the-index-afte
3. http://alblue.bandlem.com/2011/10/git-tip-of-week-index-revisited.html
* Index format - https://github.com/git/git/blob/master/Documentation/technical/index-format.txt
* Git internals - https://git-scm.com/book/en/v2/Git-Internals-Plumbing-and-Porcelain
* Revision Pointors - https://git-scm.com/book/en/v2/Git-Tools-Revision-Selection
