# Git

## Configure

Configure git

```text
git config --global user.email <email>
git config --global user.name "Marc"
git config --global push.default simple
```


## Saving credentials (HTTP)

Edit ~/.netrc

```text
machine git.gitbook.com
	login <username>
	password <password>
```

## Aliases

```text
$ git config --global alias.co checkout
$ git config --global alias.br branch
$ git config --global alias.ci commit
$ git config --global alias.st status
```

## Create a repo

```text
git init
```

## Clone a repo

This will clone a repo in to a new folder.

```text
git clone http://something.git
```	

or 

```text
git clone ssh://(username)@(url)
```

If the remote repo is massive, you can limit the amount of history downloaded using the --depth flag.


## Add changes (aka Staging Changes)

Add a file:

```text
git add 1.txt
```

Add all files:

```text
git add .
```

or 

```text
git add -p
```


## Undo add

Undo staging of all files:

```text
git reset
```

Undo the local copy changes too:

```text
git reset --hard 
```


Undo staging of a particular file:

```text
git reset 3.txt
```


Undo untracked (new) files:  

(-n will list the changes, -f will apply them)  This does not reset all modified files, only new ones that are untracked.

```text
git clean -n
git clean -f
```



## Commit 

```text
git commit -m "Commit message"
```



## Diff

Diff working folder to staged

```text
git diff 
```

Diff staged to committed

```text
git diff --staged
```


Show tree of repo:

```text
git ls-tree master
```



## View history


```text
git log -5 --one-line --name-status 
```

* -5 to show 5 lines
* --one-line to show one line per commit
* --name-status lists the files changed in each commit.


View history for a specific file:

```text
git log -p 1.txt
```

view graph:

```text
git log --graph --oneline
```


## View commited files

View committed files in current folder:

```text
git ls-files
```

The * in the graph indicates which branch each commit is on.

## Revert

Revert changes made by a revision, apply to staging area and working copy.  Then commit.

```text
git revert --no-commit <revision id>
git commit
```

Revert changes made by a revision.  This commits instantly!:

```text
git revert <revision id>
```

## Branching

Make a branch:

```text
git branch <name>
```

Switch to branch:

```text
git checkout <name>
```

Or 

```text
git switch <name>
```

View log for a branch (this will also include revisions prior to the creation of this branch):

```text
git log <name>
```

View log for a branch (and stop when reaching the beginning of the branch):

```text
git log master..<name>
```

## Merging

Checkout the target branch, then merge.

```text
git checkout master
git merge develop
```

Use --no-commit to make sure things are OK before committing.

If we have 2 changes on the same line/position, there's a conflict.  I got merge markers in my file, and a series of other copies called *_BACKUP_3110.*, *_BASE_3110.*, *_LOCAL_3110.*, *_REMOTE_3110.*

To resolve, edit the file and review the merge markers, ideally using a merge tool.   Install kdiff3, then:

```text
git mergetool 1.txt
```

After fixing the conflicts use "git add" to add the merged file, and "git commit".

Show revisions which haven't been merged from another branch.

```text
git cherry -v <other branch>
```

Cherry pick revisions:

```text
git cherry-pick <commit>
```

## Cherry picking

This command copies a revision from another branch.

```text
git cherry-pick af02e0b
```

Or copy it to your working copy only:

```text
git cherry-pick af02e0b --no-commit
```

## Squashing

Reset the head back a few commits and then recommit them as one.

```text
git reset --soft HEAD~3 &&
git commit -m "New message"
```

## Remote

Push to remote

```text
git push
```
