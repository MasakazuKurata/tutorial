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
  	## for newer versions of git, otherwise try "simple"
	default = matching
```
This defines some useful alias. Turns on color for some commands. It defines a
global ignore file that you will need to create yourself. Sets the default
editor to emacs in `No Window` mode. The `autosetuprebase = always` makes every
pull a rebase operation, which is explained below. "default matching" means
pushes are done to matching branches.

### BashCompletion and GitPrompt

Enable the bash-completion for git and consider installing the "git-prompt", to
see the current branch, and if there are uncommitted changes 

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

#### Clone the repository to your local workstation
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
   For example here I am `behind` the remote repository by one commit
   ```
   tutorial (master *)$ git branch -vv
   * master abe579c [origin/master: behind 1] Add some t[...]
   ```

#### Make a fork in github to your own repository
   Click on the "Fork" button at the top right of the page. Next to "(Un)Watch, (Un)Star"


#### Add a "downstream" remote pointing to your fork on github
   ```
     git remote add downstream https://<yourUserName>@github.com/<yourUserName>/iLCUtil.git
   ```
   The name of the remote `downstream` can be whatever you want it to be. To see
   the list of remotes and where they point run
   ```
     git remote -vv
   ```
   For example:
   ```
   HEAD (master)$ git remote -vv
   downstream      https://andresailer@github.com/andresailer/iLCUtil.git (fetch)
   downstream      https://andresailer@github.com/andresailer/iLCUtil.git (push)
   origin  https://github.com/iLCSoft/iLCUtil.git (fetch)
   origin  https://github.com/iLCSoft/iLCUtil.git (push)
   ```

### Working, updating, pushing

To develop new features or fix bugs, always create a `topic` branch from the
latest development of the `origin`. If there are commits added to the `origin`
you can incorporate these changes via a `rebase` later.

#### Make sure you have all the latest changes
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
   (`autosetuprebase=always`), this will `rebase` your local `origin` branch to
   the remote `origin` branch.
   
   If you have local changes that are not commited `pull` (i.e., `rebase`) will not be alllowed.
   Use [git stash](#git-stash) for storing your changes without commiting them

#### Now create your `topic` branch
   ```
   git checkout master
   git checkout -b newFeature
   ```
   This will create a new branch call `newFeature` based from the current branch
   `master`, if you want the branch off from a different branch just it out
   before hand. Now you can develop, commit, commit, and commit (see later about changing commits after the fact, rebase interactive, etc.). Before making a
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

#### Push you local changes to your `downstream` remote
   ```
   git push downstream newFeature
   ```
   See later in the tutorial about force'd pushing, if you had to rebase, or changed local commits after pushing to your remote repository
   
#### Open a Pull Request (PR)
   If you push to a branch on github, github will add a notification about your recently pushed branch, and give you directly a button to create a pull request.
   ![Github open pull request](images/PRQuestion.png?raw=true "OpenPR")
   Click on the button and fill in the fields for the title and description of the pull request. Chose the proper target branch. 
   ![Github describe PR](images/PR_step2.png?raw=true "DEscPR")
And create the Pull Request. If github warns you that there are conflicts, you need to rebase your topic branch to the original base branch, by repeating the steps above.

## Example of rebasing

In this example (displayed using the `git lola` alias introduced above), we are
working in improving this documentation in the `addToDoc` branch, while also
some changes on the remote repository were done. The `origin/master` and the
`addToDoc` branch both start from the commit `abe579c` but are diverging.

```
* f88eaea (HEAD, refs/heads/addToDoc) More description of committing
* a244a72 Add some notes about staging
| * c35566d (refs/remotes/origin/master, refs/remotes/origin/HEAD, refs/heads/master) Indentation, description for PR
| * 33eeb70 Add examples for git commands
| * 4bf5fb1 Github Markdown
|/
* abe579c Add some text for the tutorial for basic git setup and workflow
```
Now we want to rebase the `addToDoc` branch to the `origin/master` branch.
`git branch -vv` also tells us that we are diverging from the `origin/master` (we set addToDoc to track the upstream branch here.)

```
tutorial (addToDoc)$ git branch -vv
* addToDoc f88eaea [origin/master: ahead 2, behind 3] More description of committing
  master   c35566d [origin/master] Indentation, description for PR
```

Now for rebasing we don't have to update our local master branch, because it is
already up-to-date. We only need to rebase `addToDoc`. We also add the checkout,
just to emphasise, that one needs to be on the branch that is supposed to be
rebased.


```
git checkout addToDoc
git rebase master
```

And this will happen
```
tutorial (addToDoc)$ git rebase master
First, rewinding head to replay your work on top of it...
Applying: Add some notes about staging
Applying: More description of committing
```

We are now no longer behind the `origin/master` branch, but only ahead of it
```
tutorial (addToDoc *)$ git branch -vv
* addToDoc 5aea685 [origin/master: ahead 2] More description of committing
  master   c35566d [origin/master] Indentation, description for PR
```

Or visually:

```
* 5aea685 (HEAD, refs/heads/addToDoc) More description of committing
* fe01d7a Add some notes about staging
* c35566d (refs/remotes/origin/master, refs/remotes/origin/HEAD, refs/heads/master) Indentation, description for PR
* 33eeb70 Add examples for git commands
* 4bf5fb1 Github Markdown
* abe579c Add some text for the tutorial for basic git setup and workflow
```
the `addToDoc` branch is now a simple continuation of the `origin/master` branch



# Tutorial


## Staging and Committing

Commiting in git is split in different steps. First you prepare what you want to
commit. You can pick entire files, or just parts of changes in individual files.
Use `git status` to display the currently tracked files, untracked files, files
with modification, and files with staged changes.

### Simple commit

After modifications of a file

```
tutorial (addToDoc *)$ git status
On branch addToDoc
Your branch is behind 'origin/master' by 3 commits, and can be fast-forwarded.
  (use "git pull" to update your local branch)

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

	modified:   README.md

no changes added to commit (use "git add" and/or "git commit -a")
```

To simply add a file for the next commit do for example
```
git add README.md
```

Then the status will be:
```
tutorial (addToDoc +)$ git status
On branch addToDoc
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

	modified:   README.md
```

Further changes to these file are not automatically staged! If editing the file
after staging you will see
```
tutorial (addToDoc *+)$ git status
On branch addToDoc
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

	modified:   README.md

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

	modified:   README.md
```

Git also tells you how to remove the changes from the staging area `git reset
HEAD <file>`, or how to add more files. etc.

If all the desired changes are staged, we can commit them
```
git commit -m"Short description

Longer description about why we just needed to make this commit
Refering to Issue ID #1224 and the bug it solved"
```

### Selecting code to be committed

You should use

```
git add --patch [-p]  [FileName]
```

to make sure you are only commiting what you want to commit.  This gives you the
possibility to review your code, and avoid commiting debug print outs, changes
that should go to an extra commit or similar problems. `git add -p` will prompt you for every change if it should be staged. And allways tell you about your options.

```
tutorial (addToDoc *+)$ git add -p
diff --git a/README.md b/README.md
index 7738ee0..9821646 100644
--- a/README.md
+++ b/README.md
@@ -150,7 +150,7 @@ with modification, and files with staged changes.

 ### Simple commit

-After modifications of a fil
+After modifications of a file

  Stage this hunk [y,n,q,a,d,/,j,J,g,e,?]?
```

Here we made a typo in a previous commit, and want to add the "e" in "file". By
entering `y` we accept this `hunk` for the next commit. All options are
described when entering "?".


To see the staged changes use `git diff --staged`, `git diff` will only show the
non-staged changes.


### Amending commits

## Re-writing history: Interactive Rebase

### Switching commits

### Squash/Fixup commits

### reword'ing commit logs

## merge/rebase conflict

## git reflog

### undo merge/rebase/whatever

## git stash

`pull`, `rebase`, `checkout` might not be allowed if there are uncommited changes in the working directory. To quickly get rid of the changes there is `git stash`.
I.e.:
* `git stash` to move all changes into a stash at the current commit, equivalent to `git stash save`
* `git stash pop` to apply the latest stash to the current working tree.

There can be more than one stash at a time:

`git stash` shows all stashes, `git stash drop stash@{0}` can be used to remove a stash without applying it

