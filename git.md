---
marp: false
author: Khaled Gabr
theme: custom-theme

---

## Version Control (Git)

# What is the Version Control (Git)?

Version control systems (VCSs) are tools used to track changes to source code (or other collections of files and folders). As the name implies, these tools help maintain a history of changes; furthermore, they facilitate collaboration. VCSs track changes to a folder and its contents in a series of snapshots, where each snapshot encapsulates the entire state of files/folders within a top-level directory. VCSs also maintain metadata like who created each snapshot, messages associated with each snapshot, and so on.

# Why is version control useful?

Even when you’re working by yourself, it can let you look at old snapshots of a project, keep a log of why certain changes were made, work on parallel branches of development, and much more. When working with others, it’s an invaluable tool for seeing what other people have changed, as well as resolving conflicts in concurrent development.

---

# Version Control Requirements

- Track Everything

```bash
<root> (tree)
|
+- foo (tree)
|  |
|  + bar.txt (blob, contents = "HI Robotics Team")
|
+- baz.txt (blob, contents = "git is Amazing")
```

In Git terminology, a file is called a `blob`, and it’s just a bunch of bytes. A directory is called a `tree`

# Modeling history

```bash
                      +Feature   
o <------- o <------- o <------- o +Feature
            ^                    ^  +Bug
             \                  /         
              --------- -------o 
                              +Bug
```

- OS Independent
- Unique ID

# Data model, as pseudocode

It may be instructive to see Git’s data model written down in pseudocode:

```bash
// a file is a bunch of bytes
type blob = array<byte>

// a directory contains named files and directories
type tree = map<string, tree | blob>

// a commit has parents, metadata, and the top-level tree
type commit = struct {
    parents: array<commit>
    author: string
    message: string
    snapshot: tree
}
```

---

# Objects and content-addressing

```bash
type object = blob | tree | commit
```

In Git data store, all objects are content-addressed by their SHA-1 hash.

```bash
objects = map<string, object>

def store(object):
    id = sha1(object)
    objects[id] = object

def load(id):
    return objects[id]
```

# SHA-1

```bash
khaled@khaled:~$ echo "Hello" | git hash-object --stdin
e965047ad7c57865823c7d992b1d046ea66edf78
```

# SHA-SUM

```bash
khaled@khaled:~$ echo "Hello" | shasum 
1d229271928d3f9e2bb0375bd6ce5db6c6d348d9  -
```

```bash
khaled@khaled:~$ echo "blob 6\0Hello"
blob 6\0Hello

khaled@khaled:~$ echo -e "blob 6\0Hello"
blob 6Hello

khaled@khaled:~$ echo -e "blob 6\0Hello" | shasum 
e965047ad7c57865823c7d992b1d046ea66edf78  -
```

# References

Now, all snapshots can be identified by their SHA-1 hashes. That’s inconvenient, because humans aren’t good at remembering strings of 40 hexadecimal characters.

---

Git’s solution to this problem is human-readable names for SHA-1 hashes, called “references”. References are pointers to commits. Unlike objects, which are immutable, references are mutable (can be updated to point to a new commit). For example, the master reference usually points to the latest commit in the main branch of development.

```bash
references = map<string, string>

def update_reference(name, id):
    references[name] = id

def read_reference(name):
    return references[name]

def load_reference(name_or_id):
    if name_or_id in references:
        return load(references[name_or_id])
    else:
        return load(name_or_id)
```

- Track History
- No Content Change

# 2-Tree Arch(other version control)

One way you might imagine implementing snapshotting as described above is to have a “create snapshot” command that creates a new snapshot based on the current state of the working directory. Some version control tools work like this, but not Git.

- Working Tree
- Repositories

---

# 3-Tree Arch(GIT)

- Working Tree
- Staging area
- Repositories

![](https://marklodato.github.io/visual-git-guide/conventions.svg)

# Staging area

it’s a part of the interface to create commits.
We want clean snapshots, and it might not always be ideal to make a snapshot from the current state. For example, imagine a scenario where you’ve implemented two separate features, and you want to create two separate commits, where the first introduces the first feature, and the next introduces the second feature. Or imagine a scenario where you have debugging print statements added all over your code, along with a bugfix; you want to commit the bugfix while discarding all the print statements.
![center](https://coderefinery.github.io/git-intro/_images/staging-basics.svg)

---

# Setup Git

```bash
khaled:~$ git config --global user.name ""
khaled:~$ git config --global user.email ""
```

```bash
khaled:~/GIT$ echo "Welcome to Git" >> test.txt
khaled:~/GIT$ cat test.txt 
```

# Change to repo

```bash
khaled:~/GIT$ git init 
Initialized empty Git repository in /home/khaled/GIT/.git/
```

```bash
khaled:~/GIT$ ls -al
total 24
drwxrwxr-x  3 khaled khaled  4096 أبر 18 17:53 .
drwxr-xr-x 38 khaled khaled 12288 أبر 18 17:51 ..
drwxrwxr-x  7 khaled khaled  4096 أبر 18 17:53 .git
-rw-rw-r--  1 khaled khaled    15 أبر 18 17:52 test.txt
```

```bash
khaled:~/GIT$ git status 
On branch master

No commits yet

Untracked files:
  (use "git add <file>..." to include in what will be committed)
    test.txt

nothing added to commit but untracked files present (use "git add" to track)
```

# Check the tracked files in index

```bash
khaled:~/GIT$ git ls-files
khaled:~/GIT$ find .git//objects/ -type f
```

# Adding file to stage area

```bash
khaled:~/GIT$ git add test.txt 
```

---

# Check files in index

```bash
khaled:~/GIT$ git status 
```

```bash
khaled:~/GIT$ find .git//objects/ -type f
.git//objects/6b/7935af7497b535cae9c6a8ca2d31f4092e69be
```

```bash
khaled:~/GIT$ git ls-files
test.txt
```

```bash
khaled:~/GIT$ git ls-files -s
100644 6b7935af7497b535cae9c6a8ca2d31f4092e69be 0   test.txt
```

```bash
khaled:~/GIT$ git cat-file -t 6b7935af7497b535cae9c6a8ca2d31f4092e69be
blob
```

```bash
khaled:~/GIT$ git cat-file -s 6b7935af7497b535cae9c6a8ca2d31f4092e69be
15
```

```bash
khaled:~/GIT$ git cat-file -p 6b7935af7497b535cae9c6a8ca2d31f4092e69be
Welcome to Git
```

# Moving to repo

```bash
khaled:~/GIT$ git commit -m "inital commit"
[master (root-commit) 73271c6] inital commit
 1 file changed, 1 insertion(+)
 create mode 100644 test.txt
```

![](https://marklodato.github.io/visual-git-guide/commit-main.svg)

```bash
khaled:~/GIT(master)$ find .git//objects/ -type f
.git//objects/6b/7935af7497b535cae9c6a8ca2d31f4092e69be
.git//objects/73/271c63fd90b485ba75013c5fed3435417182da
.git//objects/b2/961e1c0566cda4b63c8af7bc10075e752d3174
```

- blob
- tree
- commit(rapper)

```bash
khaled:~/GIT(master)$ git log 
commit 73271c63fd90b485ba75013c5fed3435417182da (HEAD -> master)
Author: Khaled Gabr <khaledgabr77@gmail.com>
Date:   Tue Apr 18 18:18:32 2023 +0200

    inital commit
```

---

```bash
khaled:~/GIT(master)$ git cat-file -t 7327
commit
```

```bash
khaled:~/GIT(master)$ git cat-file -p 7327
tree b2961e1c0566cda4b63c8af7bc10075e752d3174
author Khaled Gabr <khaledgabr77@gmail.com> 1681834712 +0200
committer Khaled Gabr <khaledgabr77@gmail.com> 1681834712 +0200

inital commit
```

```bash
khaled:~/GIT(master)$ git cat-file -p b296
100644 blob 6b7935af7497b535cae9c6a8ca2d31f4092e69be test.txt
```

add a new line int the `test.txt` file.
check the file status.

```bash 
khaled:~/GIT(master)$ git status 
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
    modified:   test.txt

no changes added to commit (use "git add" and/or "git commit -a")
```

```bash
khaled:~/GIT(master)$ git status -s
 M test.txt
```

The red `M` the file not found in the `stage area` or the `repo`.

so you need to add the file:

```bash
khaled:~/GIT(master)$ git add test.txt 
```

```bash
khaled:~/GIT(master)$ git status -s
M  test.txt
```

The green `M` means you add the file to `the stage area`.

the file now in the stage area but we need to sent it to the repo. you need to commit

```bash
khaled:~/GIT(master)$ git commit -m "sec line added"
[master 7a09670] sec line added
 1 file changed, 1 insertion(+)
```

we expect git have 2 `commits` msgs, let's check:

---

```bash
khaled:~/GIT(master)$ git log 
commit 7a0967019a20436ddb595f794553335fa1ecfae6 (HEAD -> master)
Author: Khaled Gabr <khaledgabr77@gmail.com>
Date:   Tue Apr 18 20:40:12 2023 +0200

    sec line added

commit 73271c63fd90b485ba75013c5fed3435417182da
Author: Khaled Gabr <khaledgabr77@gmail.com>
Date:   Tue Apr 18 18:18:32 2023 +0200

    inital commit
```

the log show much info like who did that when which email,...but i don't need to know theses info.

```bash
khaled:~/GIT(master)$ git log --oneline
7a09670 (HEAD -> master) sec line added
73271c6 inital commit
```

okay you added file to repo let's check:

```bash
khaled:~/GIT(master)$ find .git//objects/ -type f
.git//objects/7a/0967019a20436ddb595f794553335fa1ecfae6
.git//objects/6b/7935af7497b535cae9c6a8ca2d31f4092e69be
.git//objects/73/271c63fd90b485ba75013c5fed3435417182da
.git//objects/1e/6441d5b8b3a5bc27d5588e6e1080f7361fada6
.git//objects/b2/961e1c0566cda4b63c8af7bc10075e752d3174
.git//objects/f6/bd9dd095f9fac7299ef1ed39de0c1d761aa0dd
```

```bash
khaled:~/GIT(master)$ git cat-file -p 7a09
tree 1e6441d5b8b3a5bc27d5588e6e1080f7361fada6
parent 73271c63fd90b485ba75013c5fed3435417182da
author Khaled Gabr <khaledgabr77@gmail.com> 1681843212 +0200
committer Khaled Gabr <khaledgabr77@gmail.com> 1681843212 +0200

sec line added
```

let's add a new line to the file `test.txt`

```bash
khaled:~/GIT(master)$ git status -s
 M test.txt
```

The red `M` means the file not found in the `stage area` or the `repo`.

![](https://marklodato.github.io/visual-git-guide/conventions.svg)

## Diff

now let's introduce the `git diff` command this command make compear between the `WD`, `SA` and `repo`.

---

```bash
khaled:~/GIT(master)$ git diff 
diff --git a/test.txt b/test.txt
index f6bd9dd..80e6e52 100644
--- a/test.txt
+++ b/test.txt
@@ -1,2 +1,3 @@
 Welcome to Git
 New Line Added to the file
+Again, add new line!
```

the comperasion here between the `WD` and `SA`.
let's add the change to the stage area by using command:

```bash
khaled:~/GIT(master)$ git add .
```

now try to use the same command `git diff` again, you will found there is no output shown. because the command moved the changes to `SA`.

hmm, you now asking how to check the differnace between the `stage area` and `repo`.

```bash
khaled:~/GIT(master)$ git diff --staged 
diff --git a/test.txt b/test.txt
index f6bd9dd..80e6e52 100644
--- a/test.txt
+++ b/test.txt
@@ -1,2 +1,3 @@
 Welcome to Git
 New Line Added to the file
+Again, add new line!
```

![](https://marklodato.github.io/visual-git-guide/diff.svg)

## Undoing

let's add a new line for our `test.txt` file, for undoing the line you added:

```bash
khaled:~/GIT(master)$ git restore test.txt
```

let's add a new line again for our `test.txt` file, also use `git add` to start tracking the file.

```bash
khaled:~/GIT(master)$ git status 
On branch master
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
    modified:   test.txt
```

the file added to the stage area, What if you want to make the file untracked!!
the command below will do that.

```bash
khaled:~/GIT(master)$ git restore --staged test.txt
```

check the file status now you will find the file become untracking agian!

```bash
khaled:~/GIT(master)$ git status 
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
    modified:   test.txt

no changes added to commit (use "git add" and/or "git commit -a")
```

but wait if you opend the file you will find the line you added still there make sence!, but still have command for discard the line by using

```bash
khaled:~/GIT(master)$ git restore test.txt
```

easy right!, let's check how you could discard from repo

change the recent commit, you could change the last commit by using this command, the text editor will open and you can add the new commit msg.

```bash
khaled:~/GIT(master)$  git commit --amend 
```

![](https://marklodato.github.io/visual-git-guide/commit-amend.svg)


## Reset

now the `WT`, `SA` and `repo` are the same cool!, and the `HEAD` pointing to the last commit. suddelny you decide to move backward to the last commit.
hmm, so you need to using `git reset` and reset the `HEAD`.
move backward one step:

```bash
khaled:~/GIT(master)$ git reset HEAD~1
Unstaged changes after reset:
M   test.txt
```

keep in mind the changes happend in `SA`. and this safe and clear. but what if you want this changes to the `WT`!

```bash
khaled:~/GIT(master)$ git reset --hard HEAD~1
HEAD is now at 08ee610 inital commit
```

please check the `test.txt` file now!!

all commites you added will be saved, but if you tried to show the traffic loged `git log` nothing will show, hmm but don't worry you can use `git reflog`

```bash
khaled:~/GIT(master)$ git reflog 
08ee610 (HEAD -> master) HEAD@{0}: reset: moving to HEAD~1
d772dcc HEAD@{1}: reset: moving to HEAD~1
a6bcd64 HEAD@{2}: commit (amend): new line added here!
22d79fe HEAD@{3}: commit: new line added
d772dcc HEAD@{4}: commit: undoing test
08ee610 (HEAD -> master) HEAD@{5}: commit (initial): inital commit
```

![center](https://marklodato.github.io/visual-git-guide/reset-commit.svg)

if you remmber before using the reset command the `master/HEAD` commit was `a6bcd64 HEAD@{2}: commit (amend): new line added here!`.
now let's try to `FastForward` move forward to the previous `HEAD`

```bash
khaled:~/GIT(master)$ git reset --hard HEAD@{2}
HEAD is now at a6bcd64 new line added here!
```

Amazing, check git log now you will find the `HEAD` backed again to `new line added here`

## Tags

not every commit or change you do in Git is a version!, but it could be some of commits or changes are a version!
you could Bookmark a commit as new version `Tag`.

let's create a new version, let's `add` and `commit` a new line to the `test.txt` file.

```bash
khaled:~/GIT(master)$ git tag -a v1.0 -m "Version 1.0"
```

![](https://git-scm.com/book/en/v2/images/branch-and-history.png)


```bash
khaled:~/GIT(master)$ git show v1.0 
tag v1.0
Tagger: Khaled Gabr <khaledgabr77@gmail.com>
Date:   Wed Apr 19 05:11:53 2023 +0200

Version 1.0

commit 6706ba028dc3832276e42f0b8a4bb06bd5ef6fe3 (HEAD -> master, tag: v1.0)
Author: Khaled Gabr <khaledgabr77@gmail.com>
Date:   Wed Apr 19 05:10:11 2023 +0200

    new verion is coming!

diff --git a/test.txt b/test.txt
index b5402f4..ee0dc0a 100644
--- a/test.txt
+++ b/test.txt
@@ -3,3 +3,5 @@ Welcome to Git
 undoing test 
 
 new line added 
+
+loading a new version!
```

## Branching

![](https://git-scm.com/book/en/v2/images/commit-and-tree.png)

Creating new branch

```bash
khaled:~/GIT(master)$ git branch testing 
```

![](https://git-scm.com/book/en/v2/images/head-to-master.png)

Check branches in Git

```bash
khaled:~/GIT(master)$ git branch 
  testing
* master
```

the `*` means this is the current branch(`HEAD`). so any changes `add` or `commits` will be added to the master branch.

hmm, you have now 2 branches `HEAD` or `master` and `testing` branch. also, you can check which branche you are using for `log` command:

```bash
khaled:~/GIT(master)$ git log --oneline
415af3d (HEAD -> master, testing) fourht line added
5bb5005 Third line added
d5e16df sec line added
97b3b92 initial commit
```

Let's switch to the `testing` branch

```bash
khaled:~/GIT(master)$ git switch testing 
Switched to branch 'testing'
```

![](https://git-scm.com/book/en/v2/images/head-to-testing.png)

```bash
khaled:~/GIT(testing)$ git log --oneline 
415af3d (HEAD -> testing, master) fourht line added
5bb5005 Third line added
d5e16df sec line added
97b3b92 initial commit
```

you can see now the `HEAD` is pointing to the `testing` branch

let's add and commit a new line to `test.txt` file. also, check the `git log` output.

![](https://git-scm.com/book/en/v2/images/advance-testing.png)

```bash
khaled:~/GIT(testing)$ git log --oneline
0b0feb2 (HEAD -> testing) new line added
415af3d (master) fourht line added
5bb5005 Third line added
d5e16df sec line added
97b3b92 initial commit
```

Let’s switch back to the master branch:

```bash
khaled:~/GIT(testing)$ git switch master 
Switched to branch 'master'
```

![](https://git-scm.com/book/en/v2/images/checkout-master.png)

## Merging

simple way to merge `testing` branch into `master`, you need to switch to the `master` branch first then use  git `merge` command:

```bash
khaled:~/GIT(master)$ git merge testing 
Updating 415af3d..0b0feb2
Fast-forward
 test.txt | 1 +
 1 file changed, 1 insertion(+)
```

if you need to check which branches merged into the master you can use this command:

```bash
khaled:~/GIT(master)$ git branch --merged 
  testing
* master
```

from previous command we noticed that `testing` branch merged to `master` branch..so the data file are the same!
let's delete `testing` branch:

```bash
khaled:~/GIT(master)$ git branch -d testing 
Deleted branch testing (was 0b0feb2).
```

let's check if the `testing` branch deleted:

```bash
khaled:~/GIT(master)$ git log --oneline 
0b0feb2 (HEAD -> master) new line added
415af3d fourht line added
5bb5005 Third line added
d5e16df sec line added
97b3b92 initial commit
```

##  Divergent history

You created and switched to a branch, did some work on it, and then switched back to your main branch and did other work. Both of those changes are isolated in separate branches: you can switch back and forth between the branches and merge them together when you’re ready. And you did all that with simple branch, checkout, and commit commands.

![](https://git-scm.com/book/en/v2/images/advance-master.png)

```bash
khaled:~/GIT(master)$ git branch testing 
```

```bash
khaled:~/GIT(master)$ git switch testing 
Switched to branch 'testing'
```

```bash
khaled:~/GIT(testing)$ nano test.txt 
khaled:~/GIT(testing)$ git add .
khaled:~/GIT(testing)$ git commit -m "added new line for merge"
[testing 5b0f71f] added new line for merge
 1 file changed, 1 insertion(+)
```

```bash
khaled:~/GIT(testing)$ git log --oneline 
5b0f71f (HEAD -> testing) added new line for merge
0b0feb2 (master) new line added
415af3d fourht line added
5bb5005 Third line added
d5e16df sec line added
97b3b92 initial commit
```

let's `switch` to `master` branch and create new file `test2.txt`, `add` and `commit` to the `master` branch.

```bash
khaled:~/GIT(master)$ graph 
* 1a343f2 (HEAD -> master) new test file added
| * 5b0f71f (testing) added new line for merge
|/  
* 0b0feb2 new line added
* 415af3d fourht line added
* 5bb5005 Third line added
* d5e16df sec line added
* 97b3b92 initial commit
```

![](https://git-scm.com/book/en/v2/images/advance-master.png)

```bash
khaled:~/GIT(master)$ git merge testing 
Merge made by the 'recursive' strategy.
 test.txt | 1 +
 1 file changed, 1 insertion(+)
```

![](https://git-scm.com/book/en/v2/images/basic-merging-2.png)

```bash
khaled:~/GIT(master)$ git log --oneline --decorate --graph --all 
*   24f880f (HEAD -> master) Merge branch 'testing'
|\  
| * 5b0f71f (testing) added new line for merge
* | 1a343f2 new test file added
|/  
* 0b0feb2 new line added
* 415af3d fourht line added
* 5bb5005 Third line added
* d5e16df sec line added
* 97b3b92 initial commit
```

## Merge Conflicts

let's say your `master` branch has file `test.txt`, and you create a new branch called `testing`. let's switch to the `testing` branch `ls` you will find the `test.txt` file there..let's create a new file called `text2.txt` add some lines then `add` and `commit`..change to `master` branch create a new file with same name `test2.txt` add some lines then `add` and `commit`.

```bash
khaled:~/GIT(master)$ graph
* 17739c2 (HEAD -> master) sec line added
*   24f880f Merge branch 'testing'
|\  
* | 1a343f2 new test file added
| | * ffd55b4 (testing) new file added
| |/  
| * 5b0f71f added new line for merge
|/  
* 0b0feb2 new line added
* 415af3d fourht line added
* 5bb5005 Third line added
* d5e16df sec line added
* 97b3b92 initial commit
```

you did amazing work..Let's `Merge` the `testing` branch into the `master` branch.

```bash
khaled:~/GIT(master)$ git merge testing 
CONFLICT (add/add): Merge conflict in test2.txt
Auto-merging test2.txt
Automatic merge failed; fix conflicts and then commit the result.
```

hmm, it's seems normal `merge` not working fine..let's fix the merge.

you can use `mergetool` command for fixing the issue, but we will another way to let the deverpoer the decision:

open the `test2.txt` file you will found the file some new lines the first is `HEAD` line and second is the `merged` line(`testing`)..normlly you need to read and remove these lines..let's `add` theses changes and complete ypur merge.

```bash
khaled:~/GIT(master)$ git merge --continue 
[master fb06b2c] Merge branch 'testing
```

```bash
khaled:~/GIT(master)$ graph
*   fb06b2c (HEAD -> master) Merge branch 'testing
|\  
| * ffd55b4 (testing) new file added
* | 17739c2 sec line added
* | 24f880f Merge branch 'testing'
|\| 
| * 5b0f71f added new line for merge
* | 1a343f2 new test file added
|/  
* 0b0feb2 new line added
* 415af3d fourht line added
* 5bb5005 Third line added
* d5e16df sec line added
* 97b3b92 initial commit
```

## Rebase

A rebase is an alternative to a merge for combining multiple branches. Whereas a merge creates a single commit with two parents, leaving a non-linear history, a rebase replays the commits from the current branch onto another, leaving a linear history.

### Branch
![](https://git-scm.com/book/en/v2/images/basic-rebase-1.png)

### Merging divergent history
![](https://git-scm.com/book/en/v2/images/basic-rebase-2.png)

### Rebase

```bash
khaled:~/GIT(testing)$ git rebase master 
First, rewinding head to replay your work on top of it...
Fast-forwarded testing to master.
```

![](https://git-scm.com/book/en/v2/images/basic-rebase-3.png)

### fast-forward merge

```bash
khaled:~/GIT(master)$ git merge testing 
```

![](https://git-scm.com/book/en/v2/images/basic-rebase-4.png)

---

# Git command-line interface

`Basics`

- `git help <command>`: get help for a git command
- `git init`: creates a new git repo, with data stored in the .git directory
- `git status`: tells you what’s going on
- `git add <filename>`: adds files to staging area
- `git commit`: creates a new commit Write good commit messages! Even more reasons to write good commit messages!
- `git log`: shows a flattened log of history
- `git log --all --graph --decorate`: visualizes history as a DAG
- `git diff <filename>`: show changes you made relative to the staging area
- `git diff <revision> <filename>`: shows differences in a file between snapshots
- `git checkout <revision>`: updates HEAD and current branch

`Branching and merging`

- `git branch`: shows branches
- `git branch <name>`: creates a branch
- `git checkout -b <name>`: creates a branch and switches to
  - it same as `git branch <name>`; `git checkout <name>`

---

- `git merge <revision>`: merges into current branch
- `git mergetool`: use a fancy tool to help resolve merge conflicts
- `git rebase`: rebase set of patches onto a new base

`Remotes`

- `git remote`: list remotes
- `git remote add <name> <url>`: add a remote
- `git push <remote> <local branch>:<remote branch>`: send objects to remote, and update remote reference
- `git branch --set-upstream-to=<remote>/<remote branch>`: set up correspondence between local and remote branch
- `git fetch`: retrieve objects/references from a remote
- `git pull`: same as git fetch; git merge
- `git clone`: download repository from remote

`Undo`

- `git commit --amend`: edit a commit’s contents/message
- `git reset HEAD <file>`: unstage a file
- `git checkout -- <file>`: discard changes

---

`Advanced Git`

- `git config`: Git is highly customizable
- `git clone --depth=1`: shallow clone, without entire version history
- `git add -p`: interactive staging
- `git rebase -i`: interactive rebasing
- `git blame`: show who last edited which line
- `git stash`: temporarily remove modifications to working directory
- `git bisect`: binary search history (e.g. for regressions)
- `.gitignore`: specify intentionally untracked files to ignore
