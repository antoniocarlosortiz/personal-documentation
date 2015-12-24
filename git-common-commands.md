#Git Common Commands

```
git init
```
to initialize a git repository


```
git status
```
to see the current status of the project


```
git add <filename>
```
to start tracking changes made to specified file. Wildcards can be added to the filename.


```
git commit -m <message>
```
to store staged changes from ```git add```

```
git log
```
list of all the commits in the order that it was they were committed.

```
git remote add origin <url_of_repo>.git
```
To add a remote repository

```
git push -u origin master
```
The push command tells Git where to put our commits. The name of our remote is ```origin``` and the default local branch name is ```master```. The ```-u``` tells Git to remember the parameters so next time we can simply run ```git push``` and Git will know what to do.

```
git pull origin master
```
Check for any changes in our git repository and pull down any changes.

Reference for what ```git pull``` actually does:

[origin master vs origin/master](http://stackoverflow.com/questions/18137175/in-git-what-is-the-difference-between-origin-master-vs-origin-master)

```
git diff HEAD
```
Get the difference from our most recent commit.

```
git diff --staged
```
To see the changes we just staged.


```
git reset <filename>
```
Used to unstage a file.

```
git checkout -- <filename>
```
to change back a file to how they were at the last commit.

```
git branch branch_name
```
to create a branch

```
git rm <filename>
```
wildcards can be used on the filename to remove more than one file. ```git rm``` should be used as opposed to normally deleting files so git can track the deleted files. If you happen to delete a file without ```git```, use ```git commit -am <message>``` to auto remove deleted files with the commit.

```
git checkout <branchname>
```
to switch to a branch.

```
git merge <branchname>
```
to merge the specifed branch with to the current branch. Conflicts can emerge.

```
git merge -d <branchname>
```
to delete a branch that has already been merged with the master branch.

```
git merge -d -f <branchname>
```
or
```
git merge -D <branchname>
```
to delete branches that havent been merged with the master branch.


