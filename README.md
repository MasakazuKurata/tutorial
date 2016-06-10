# Basic

## Configuration

### Git Configuration

In your *~/.gitconfig*
```
  [alias]
	st = status
	ci = commit
	co = checkout
	lola = log --graph --decorate=full\n--pretty=oneline --abbrev-commit --all
	logf = "!echo \"Remember to add -S<string>\" ; git log --color -p --source --all"
  [color]
	ui = true
	diff = auto
	status = auto
  [core]
	excludesfile = <YourHomeFolde>/.gitignore_global
	editor = emacs -nw
  [branch]
	autosetuprebase = always
  [push]
	default = matching
```
This defines some useful alias. Turns on color for some commands. It defines a
global ignore file that you will need to create yourself. Sets the default
editor to emacs in `No Window` mode. The `autosetuprebase = always` makes every
pull a rebase operation, which is explained below. "default matching" means
pushes are done to matching branches.

### BashCompletion and GitPrompt

Enable the bash-completion for git and consider installing the "git-prompt", to
see the current branch, and if there are uncommitted changes consider installing
the git prompt

  http://code-worrier.com/blog/git-branch-in-bash-prompt/


E.g.:
```
  HEAD (newFeature)$ git commit --<tab>
  --all               --author=           --dry-run           --file=             --interactive       --no-verify         --quiet             --reset-author      --signoff           --untracked-files=  --verbose
  --amend             --cleanup=          --edit              --include           --message=          --only              --reedit-message=   --reuse-message=    --template=         --untracked-files   --verify
```

  I am on branch `newFeature` and just asked for all options to the `commit`
  command. This simplifies remembering the options a lot and speeds up git usage.


## Github Workflow

### Cloning a repository to work on

Here is an example how to create a local working copy for the iLCUtil
packages. For other packages just change the package name. You can also use the
ssh to clone repositories, see the git(hub) documentation for that.
Replace `<yourUserName>` by your github username

1. Clone the repository to your local workstation
```
     git clone https://github.com/iLCSoft/iLCUtil.git HEAD
```
   This `remote` will be called "origin", and the first branch that is checked
   out is master. The local master branch will automatically track the master
   branch on the origin remote. To see the local branches and what remote
   branch, if any, they track
```
     git branch -vv
```
   This will list the local branches, if they track a remote branch, and if
   there are differences between the local and remote branch (commits ahead or behind).

2. Make a fork in github to your own repository

3. Add a "downstream" remote pointing to your fork on github
```
     git remote add downstream https://<yourUserName>@github.com/<yourUserName>/iLCUtil.git
```
   The name of the remote `downstream` can be whatever you want it to be. To see
   the list of remotes and where they point run
```
     git remote -vv
```

### Working, updating, pushing

To develop new features or fix bugs, always create a `topic` branch from the
latest development of the `origin`. If there are commits added to the `origin`
you can incorporate these changes via a `rebase` later.

1. Make sure you have all the latest changes
```
     git fetch origin
```
   To get all changes from the `origin` remote. Then you can check if the
   `master` or other tracking branch needs updating
```	
     git branch -vv
```
   Assuming you are `behind` on the `master` branch
```
     git checkout master
     git pull origin
```
   If you are using the configuration as described above
   (autosetuprebase=Always), this will `rebase` your local `origin` branch to
   the remote `origin` branch.

   Now create your `topic` branch
```
     git checkout master
     git checkout -b newFeature
```
   This will create a new branch call `newFeature` based from the current branch
   `master`, if you want the branch off from a different branch just it out
   before hand. Now you can develop, commit, commit, and commit. Before making a
   Pull Request you want to have all the other developments that might have
   occurred incorporated into your branch. So
```
     git fetch origin
     git branch -vv
```
   to see if your base branch needs to be updated. If there are changes in the
   base branch, we update our local base branch, assuming it is called `master`
   via git pull, and then `rebase` our topic branch `newFeature` to the base branch
```
     git checkout master
     git pull origin
     git checkout newFeature
     git rebase master
```
   If there are conflicts git will tell you how to resolve them. Also see
   further down in the tutorial about resolving conflicts during rebase.



# Tutorial


## Staging and Committing

### Simple commit

### Selecting code to be committed

### Amending commits

## Re-writing history: Interactive Rebase

### Switching commits

### Squash/Fixup commits

### reword'ing commit logs

## merge/rebase conflict

## git reflog

### undo merge/rebase/whatever

## git stash (save, pop, drop)
