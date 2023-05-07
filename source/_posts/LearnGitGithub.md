---
title: Learn Git & GitHub
date: 2023-01-07 10:39:49
categories:
- Tools
tags: 
- Git
- Tools
---

A note about learning Git.

<!--more-->

# Git

Git is a popular version control system.
     
## Get started

### Install 

```
add-apt-repository ppa:git-core/ppa
sudo apt-get install git
```

### Check the Git version

```
git --version
```

### Configure git

```
git config --global user.name "Your Name"
git config --global user.email "email@example.com"
```

> Note: `--global` will set the name and email for every repository on this computer.
> You can remove `--global` to set the information just for this repository.

### Create a git folder

```
mkdir myproject
cd myproject
```

### Initialize git

You must navigate to the correct folder.

```
git init
```

> Note: Git creates a hidden folder called .git in the root directory of the repository to keep track of changes.

## Add new files to git

* First, create a new file.

```
git status
```

Now git is aware of the file, but has not added it to our repository.

* Tracked: files have been added to your repository.
* Untracked: files that are in your working directory, but not added to your repository.

`git status --short` can display the changes in a more compact way.

> `??` Untracked files.
> `A` Files add to stage.
> `M` Modified files.
> `D` Deleted files.

## Git staging environment

Staged files are ready to be commited to the repository you are working in.

Now add the new file to the staging environment.

```
git add newfile.cpp
```

Now you can use `git status` to check the status.

### Git add more than one file

Please create some files and change the `newfile`.

```
git add --all
```

Using `--all` instead of individual filenames will stage all changes(new, modified, deleted).

> The shorthand command for git add `--all` is git add `-a`.

## Git commit

When you finish your work, you are ready to move from `stage` to `commit` for your repository.

Git considers each commit change point or "save point", which you can go back to make some changes.
When we commit, you should always include a message to indicate what you changed.

```
git commit -m "First release"
```

### Git commit without stage

```
git commit -a -m "Second release"
```

We don't recommend this way, because it can sometimes cause you to include unwanted changes.
And this command is not suitable for committing untracked files.

### Git commit log

To view the history of commits for a repository, use the `log` command.
```
git log
```

## Git help

* `git commit -help` -- See all the available options for the specific command.
    > Use `--help` to open the relevant Git manual page.
* `git help --all` -- See all possible commands.
    * `SHIFT + g` to the end of the list, `q` to exit.

## Git branch

A branch is a new version of the main repository's codebase.

To add some new features to the `firstfile.cpp`, create a new branch.

```
git branch newbranch
```

`git branch` can display all the branches.

`git checkout newbranch` can move our current workspace from the main branch to the new branch.

> `git checkout -b newbranch` will create a new branch and move to it if it doesn't exist.

Make some changes to a file and add a new file.

`git status` -- To check the status of the current branch.

`git add --all` -- Add files to the staging environment.

Now, `git checkout main` and `ls`, you'll find that the file created just now is no longer exist.

## Emergency branch

Imagine you are not yet done with the new branch, but we need to fix an error on main.

We can create a new branch called `bugFix` to deal with it without messing the two branches.

Make some changes, `git add` and `git commit`, now we should merge two branches.

## Git merge

### Merge branches

Now `git checkout main` and `git merge bugFix`.

```
git merge bugFix
```

After merging, this branch is useless, so we want to delete it.

```
git branch -d bugFix
```

### Merge conflicts

Now we return to our newbranch to finish our work, make some changes to the `firstfile.cpp`.

Then `git checkout main` and `git merge newbranch` to apply new changes.

We failed, then use `git status` to check the status.

```
Unmerged paths:
  (use "git add <file>..." to mark resolution)
        both modified:   firstfile.cpp
```

Open the `firstfile.cpp` you'll find there's something new.

Adjust the file to what you want, then git add and git commit -m "merged changes".

Delete the `newbranch`, then finish this part of work.

# Git and Github

## Get started

Go to Github to sign up for an account and creat a new repository.

> Remember to use the same e-mail adress you used in the git config.

### Push local repository to GitHub

Copy the link of the repository paste the following command.

Search for how to configure ssh.

```
git remote add origin git@github.com:lzlcs/learngit.git
```

Now, push out main branch to the origin url, and set it as the default remote branch.

```
git push --set-upstream origin main
```

> `-u` is the same as `--set-upstream`.

## Edit files in Github

It's simple, follow the instructions.

## Git pull from github

Use `pull` to get the most recent changes to your local copy.
`pull` is a combination of 2 different commands: `fetch` and `merge`.

### Git fetch
`fetch` gets all the change history of a tracked branch.
Now check our `status`.

```
On branch main
Your branch is behind 'origin/main' by 1 commit, and can be fast-forwarded.
  (use "git pull" to update your local branch)

nothing to commit, working tree clean
```

We are behind the `origin/main` by 1 `commit`, let's double check by viewing the `log`.
Also use `git diff` to show the differences between our local `main` and `origin/main`.

So we can safely `merge`.

### Git merge

```
git merge origin/main
git status
```
Everything is done.

### Git pull

Make another change to the `README.md`.

```
git pull origin
```

## Push to Github

Try making some changes to our local git and pushing them to Github.

```
git commit -m "xxxx"
git push origin
git status
```

Go to Github to confirm that the repository has a new commmit.

## Github branch

Create a branch `beta` on Github and make some changes.

```
git pull origin
git status
git branch -a
```

> Use `git branch -a` to show all local and remote branches.
> Use `git branch -r` to show remote branches only.

`git checkout beta` and `git pull` to check if it is all up to date.

Now you can see the changes you made recently on branch `beta`.

## Push a branch to Github

```
git checkout -b branch2
```

Make some changes.

```
git status
git add 
git commit -m "..."
git push origin branch2
```

In Github, we can see the changes and merge them into the branch `master` if we approve it.

## Github flow 

1. Create a new branch
2. Make changes and add commits
3. Open a pull request
4. Review
5. Deploy
6. Merge

### Create a new branch

If you want to try something new, you create a new branch.
Branching gives you an environment where you can make changes without affecting the main branch.

When you make a new branch, you will always want to make it from the main branch.

> Using descriptive names for new branches, so everyone can understand what is happening.

### Make changes and add commits

When you reach a small milestone, add the changes into your branch by commit.

> Commit message are very important.

### Open a pull request

A Pull Request notifies people you have changes ready for them to consider or review.

### Review 

When a Pull Request is made, it can be reviewed by whoever has the proper access to the branch.

### Deploy 

GitHub allows you to deploy from a branch for final testing in production before merging with the master branch.

### Merge

After exhaustive testing, you can merge the code into the master branch!

# Git contribute

## Github fork

### Fork a repository

Click the `fork`.

## Git clone

### Clone a fork from Github

** Move back to the origin repository. **

```
git clone ....
cd ....
git status
git log
git remote -v 
```
Raname: `git remote rename origin upstream`.

Then fetch the URL of our own fork: `git remote add origin ...`

* origin: we can read and write.
* upstream wo can read only.

## Github send pull request

Make some changes.

```
git add .
git commit -m ""
git push origin
```

Now go to Github and pull request.

## Git ignore

When sharing your code with others, there are often files or parts of your project, you do not want to share.

### Create .gitignore

`touch .gitignore`

> It is also possible to have additional .gitignore files in subdirectories. 
> These only apply to files or folders within that directory.

## Git security

```
ssh-keygen -t rsa -b 4096 -C "...@...com"
ssh-add ~/.ssh/id_rsa
clip < ~/.ssh/id_rsa.pub
```
Add ssh to Github.
```
ssh -T git@github.com
git remote set-url remote-name git@github.com:username/repository.git
```

# Git undo

## Git revert

When we want to take a previous commit and add it as a new commit, keeping the log intact.

```
git log --oneline
git revert HEAD --no-edit
```
Use `--no-edit` to skip the commit message editor (getting the default revert message).

## Git reset

Move the repository back to a previous commit, discarding any changes made after that commit.

```
git reset ...
```

## Git amend

commit --amend is used to modify the most recent commit.

```
git commit -m "Adding plines to reddme"
git commit --amend -m "Added lines to README.md"
```

> You should avoid making changes that rewrite history to remote repositories,
> especially if others are working with them.
