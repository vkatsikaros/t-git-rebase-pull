# Test `git reset` on a `-no-ff` and "clean log" workflow

Playing with a git workflow where the branches merged are rebased in order to keep the log history clean (I ain't arguing if this is good or bad, just playing with it).

User A creates a branch. In the meantime user B adds a change and pushes in remote. Then, when user A merges hs branch and tries to push, it obviously gets rejected. How to quickly solve the situation?

Reset the master branch to the local origin/master ref (ie before the merge).
```
git reset --hard origin/master
```
Then pull/fetch and rebase the feature branch again. If the new commit pushed adds conflicting changes, more manual work might be involved. If not the previous rebase effort is not wasted.

Simulate 2 users each with their repo. Create repo, add a file, push for A and clone for B.
```
 $ rm -rf repoA repoB
 $ mkdir repoA
 $ mkdir repoB
 $ cd repoA/
/repoA $ git init
Initialized empty Git repository in blah...blah...blah/repoA/.git/
/repoA $ cat > hello.txt
Hello!
/repoA $ git remote add origin git@github.com:vkatsikaros/t-git-rebase-pull.git
/repoA $ git add hello.txt
/repoA $ git commit -m"Start"
[master (root-commit) add14a0] Start
 1 file changed, 1 insertion(+)
 create mode 100644 hello.txt
/repoA (master)$ git push -u origin master
Counting objects: 3, done.
Writing objects: 100% (3/3), 222 bytes | 0 bytes/s, done.
Total 3 (delta 0), reused 0 (delta 0)
To git@github.com:vkatsikaros/t-git-rebase-pull.git
 * [new branch]      master -> master
Branch master set up to track remote branch master from origin.
```

Create branch for user A and push it
```
/repoA (master)$ git checkout -b lala
Switched to a new branch 'lala'
/repoA (lala)$ vim hello.txt
/repoA (lala)$ git add hello.txt
/repoA (lala)$ git commit -m"Changes"
[lala f7e59b9] Changes
 1 file changed, 1 insertion(+), 1 deletion(-)
/repoA (lala)$ git push origin lala
Counting objects: 5, done.
Writing objects: 100% (3/3), 260 bytes | 0 bytes/s, done.
Total 3 (delta 0), reused 0 (delta 0)
To git@github.com:vkatsikaros/t-git-rebase-pull.git
 * [new branch]      lala -> lala
/repoA (lala)$ git log -p
commit f7e59b91ce180971756fbf473d22ab9f61a3ac0e
Author: Vangelis Katsikaros <vkatsikaros@gmail.com>
Date:   Thu Apr 21 12:40:45 2016 +0300

    Changes

diff --git a/hello.txt b/hello.txt
index 10ddd6d..c2679e4 100644
--- a/hello.txt
+++ b/hello.txt
@@ -1 +1 @@
-Hello!
+Hello There!

commit add14a0413950f23044bd698b9dbc75f2aa8a934
Author: Vangelis Katsikaros <vkatsikaros@gmail.com>
Date:   Thu Apr 21 12:39:46 2016 +0300

    Start

diff --git a/hello.txt b/hello.txt
new file mode 100644
index 0000000..10ddd6d
--- /dev/null
+++ b/hello.txt
@@ -0,0 +1 @@
+Hello!
/repoA (lala)$ cat > hello2.txt
Hello 2222!
/repoA (lala)$ git add hello2.txt
/repoA (lala)$ git commit -m"New file"
[lala debe83b] New file
 1 file changed, 1 insertion(+)
 create mode 100644 hello2.txt
/repoA (lala)$ git push origin lala
Counting objects: 4, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 284 bytes | 0 bytes/s, done.
Total 3 (delta 0), reused 0 (delta 0)
To git@github.com:vkatsikaros/t-git-rebase-pull.git
   f7e59b9..debe83b  lala -> lala
```

User B now adds a conflicting change and pushes.
```
/repoA (lala)$ cd ../repoB/
/repoB $ git clone git@github.com:vkatsikaros/t-git-rebase-pull.git .
Cloning into '.'...
remote: Counting objects: 6, done.
remote: Compressing objects: 100% (2/2), done.
remote: Total 6 (delta 0), reused 6 (delta 0), pack-reused 0
Receiving objects: 100% (6/6), done.
Checking connectivity... done.
/repoB (master)$ vim hello.txt
/repoB (master)$ git add hello.txt
/repoB (master)$ git commit -m"Changes"
[master c15d445] Changes
 1 file changed, 1 insertion(+), 1 deletion(-)
/repoB (master)$ git push origin master
Counting objects: 7, done.
Writing objects: 100% (3/3), 257 bytes | 0 bytes/s, done.
Total 3 (delta 0), reused 0 (delta 0)
To git@github.com:vkatsikaros/t-git-rebase-pull.git
   add14a0..c15d445  master -> master
/repoB (master)$ git log -p
commit c15d445584aa232fb9028deae4c7f719549b852b
Author: Vangelis Katsikaros <vkatsikaros@gmail.com>
Date:   Thu Apr 21 12:42:12 2016 +0300

    Changes

diff --git a/hello.txt b/hello.txt
index 10ddd6d..980a0d5 100644
--- a/hello.txt
+++ b/hello.txt
@@ -1 +1 @@
-Hello!
+Hello World!

commit add14a0413950f23044bd698b9dbc75f2aa8a934
Author: Vangelis Katsikaros <vkatsikaros@gmail.com>
Date:   Thu Apr 21 12:39:46 2016 +0300

    Start

diff --git a/hello.txt b/hello.txt
new file mode 100644
index 0000000..10ddd6d
--- /dev/null
+++ b/hello.txt
@@ -0,0 +1 @@
+Hello!
```

User A now merges (hasn't pulled to receive B's commit) and tries to push.
```
/repoB (master)$ cd ../repoA/
/repoA (lala)$ git checkout master
Switched to branch 'master'
Your branch is up-to-date with 'origin/master'.
/repoA (master)$ git merge lala
Updating add14a0..debe83b
Fast-forward
 hello.txt  | 2 +-
 hello2.txt | 1 +
 2 files changed, 2 insertions(+), 1 deletion(-)
 create mode 100644 hello2.txt
/repoA (master)$ gi^C
/repoA (master)$ git log --oneline --graph --decorate
* debe83b (HEAD, origin/lala, master, lala) New file
* f7e59b9 Changes
* add14a0 (origin/master) Start
/repoA (master)$ git reset --hard origin/master
HEAD is now at add14a0 Start
/repoA (master)$ git merge --no-ff lala
Merge made by the 'recursive' strategy.
 hello.txt  | 2 +-
 hello2.txt | 1 +
 2 files changed, 2 insertions(+), 1 deletion(-)
 create mode 100644 hello2.txt
/repoA (master)$ git log --oneline --graph --decorate
*   e195d93 (HEAD, master) Merge branch 'lala'
|\
| * debe83b (origin/lala, lala) New file
| * f7e59b9 Changes
|/
* add14a0 (origin/master) Start
/repoA (master)$ git push origin master
To git@github.com:vkatsikaros/t-git-rebase-pull.git
 ! [rejected]        master -> master (fetch first)
error: failed to push some refs to 'git@github.com:vkatsikaros/t-git-rebase-pull.git'
hint: Updates were rejected because the remote contains work that you do
hint: not have locally. This is usually caused by another repository pushing
hint: to the same ref. You may want to first integrate the remote changes
hint: (e.g., 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
```
Time to reset the master before the merge commit.
```
/repoA (master)$ git reset --hard origin/master
HEAD is now at add14a0 Start
```

Now pull to get the change and will rebase the feature branch again.
```
/repoA (master)$ git pull
remote: Counting objects: 3, done.
remote: Total 3 (delta 0), reused 3 (delta 0), pack-reused 0
Unpacking objects: 100% (3/3), done.
From github.com:vkatsikaros/t-git-rebase-pull
   add14a0..c15d445  master     -> origin/master
Updating add14a0..c15d445
Fast-forward
 hello.txt | 2 +-
 1 file changed, 1 insertion(+), 1 deletion(-)
/repoA (master)$ git checkout lala
Switched to branch 'lala'
```
To rebase we also need to resolve a conflict (this was done on purpose).
```
/repoA (lala)$ git rebase master
First, rewinding head to replay your work on top of it...
Applying: Changes
Using index info to reconstruct a base tree...
M	hello.txt
Falling back to patching base and 3-way merge...
Auto-merging hello.txt
CONFLICT (content): Merge conflict in hello.txt
Failed to merge in the changes.
Patch failed at 0001 Changes
The copy of the patch that failed is found in:
   blah...blah...blah/repoA/.git/rebase-apply/patch

When you have resolved this problem, run "git rebase --continue".
If you prefer to skip this patch, run "git rebase --skip" instead.
To check out the original branch and stop rebasing, run "git rebase --abort".

/repoA ((no branch, rebasing lala))$ git diff
diff --cc hello.txt
index 980a0d5,c2679e4..0000000
--- a/hello.txt
+++ b/hello.txt
@@@ -1,1 -1,1 +1,5 @@@
++<<<<<<< HEAD
 +Hello World!
++=======
+ Hello There!
++>>>>>>> Changes
/repoA ((no branch, rebasing lala))$ git checkout --theirs hello.txt
/repoA ((no branch, rebasing lala))$ git add hello.txt
/repoA ((no branch, rebasing lala))$ git rebase --continue
Applying: Changes
Applying: New file
/repoA (lala)$ git checkout master
Switched to branch 'master'
Your branch is up-to-date with 'origin/master'.
/repoA (master)$ git merge --no-ff lala
Merge made by the 'recursive' strategy.
 hello.txt  | 2 +-
 hello2.txt | 1 +
 2 files changed, 2 insertions(+), 1 deletion(-)
 create mode 100644 hello2.txt
/repoA (master)$ git log --oneline --graph --decorate
*   7fb4871 (HEAD, master) Merge branch 'lala'
|\
| * f9cebc3 (lala) New file
| * a97e94e Changes
|/
* c15d445 (origin/master) Changes
* add14a0 Start
/repoA (master)$ git push origin master
Counting objects: 6, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (3/3), done.
Writing objects: 100% (4/4), 456 bytes | 0 bytes/s, done.
Total 4 (delta 2), reused 0 (delta 0)
To git@github.com:vkatsikaros/t-git-rebase-pull.git
   c15d445..7fb4871  master -> master
```
