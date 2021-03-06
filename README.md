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


##### Make sure your master branch is not _ahead_ of the origin

Make sure you don't have any commits on your `master` branch that are not part of the upstream `master`.
This means
```
git branch -vv
master             abcd1234 [origin/master: behind 9] useful commit message
```
Here you can `git pull` to obtain the commits from the origin without any problems.

It should never say `ahead` for the master branch. I.e., this shouldn't be the case
```
git branch -vv
master             absc1234  [origin/master: ahead 12] foobar
```
If that is the case you need to figure out if the commits on your master are worth keeping, then rename the branch and get a new fresh master from origin.
```
git checkout master
git branch -m backupMaster ## rename the branch
```
or reset the branch to the `origin/master`.

#### Now create your `topic` branch
   ```
   git checkout master
   git checkout -b newFeature
   ```
   This will create a new branch call `newFeature` based from the current branch
   `master`, if you want the branch off from a different branch just it out
   before hand. Now you can develop, commit, commit, and commit (see later about changing commits after the fact, rebase interactive, etc.). 
   
##### Incorporating upstream changes after creating the branch (rebasing)
   Before making a Pull Request you want to have all the other developments that might have
   occurred incorporated into your branch. This is also possible to do after the Pull Request was created, when pushing the branch downstream the Pull Request is automatically updated. So
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
   Make sure you have set the `autosetuprebase=always` in the config as shown [above](#configuration)

#### Push you local changes to your `downstream` remote
   ```
   git push downstream newFeature
   ```
   See later in the tutorial about force'd pushing, if you had to rebase, or changed local commits after pushing to your remote repository. if you have updated your history, for example with rebasing, you have to `force` push
   ```
   git push -f downstream newFeature
   ```
   
   
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

You 
use

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

During rebasing conflicts between changes in your branch and the other branch can occur.
(As mentioned in the very beggining of this tutorial you should install the "git prompt" as it will tell you about the different stages of rebasing and eliminate the confusion about what is going on.)

So lets say we have to rebase our feature branch to an updated master:
```
(otherBranch) $ git checkout featureGranch
(featureBranch) $ git rebase master
First, rewinding head to replay your work on top of it...
Applying: Implementation of anti-DID XYZ field-map reader
Using index info to reconstruct a base tree...
M       detector/other/FieldMapXYZ.cpp
Auto-merging detector/other/FieldMapXYZ.cpp
CONFLICT (content): Merge conflict in detector/other/FieldMapXYZ.cpp
error: Failed to merge in the changes.
Patch failed at 0001 Implementation of anti-DID XYZ field-map reader
The copy of the patch that failed is found in: .git/rebase-apply/patch

When you have resolved this problem, run "git rebase --continue".
If you prefer to skip this patch, run "git rebase --skip" instead.
To check out the original branch and stop rebasing, run "git rebase --abort".

```
Git tells you here how to resolve the conflict, or how to end the rebase attempt right now with `git rebase --abort`
If you do not abort, `git-prompt` tells you that your are currently rebasing
```
(featureBranch *+|REBASE 1/10) $
```

Git status will tell you about which files have a conflict
```
(AntiDIDFieldMap *+|REBASE 1/10)$ git st
rebase in progress; onto cd32419
You are currently rebasing branch 'AntiDIDFieldMap' on 'cd32419'.
  (fix conflicts and then run "git rebase --continue")
  (use "git rebase --skip" to skip this patch)
  (use "git rebase --abort" to check out the original branch)

Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        modified:   detector/include/FieldMapXYZ.h

Unmerged paths:
  (use "git reset HEAD <file>..." to unstage)
  (use "git add <file>..." to mark resolution)

        both modified:   detector/other/FieldMapXYZ.cpp

```
Here the file `FieldMapXYZ.cpp` has conflicts, in the same commit the file `FieldMapXYZ.h` was changed without giving conflicts.
We will now resolve the conflict in `FieldMAPXYZ.cpp` and then continue our rebase. During the rebase the commit message and other changes in the code are pre-served. We just need to solve the conflicts.

In the file, where the conflict occurs, the two different versions lines are marked in merge markers:
```
++<<<<<<< cd32419b622beb5f22de23aa6c9b0ec0e5ab5d4b
 +  if (not ( 0.0 <= x && x <= rhoMax &&
 +          0.0 <= y && y <= zMax ) ) {
 +
++=======
+   if (not ( x >= xMin && x <= xMax &&
+             y >= yMin && y <= yMax &&
+             z >= zMin && z <= zMax )
+      ) {
+     globalField[0] = globalField[1] = globalField[2] = 0.0;
++>>>>>>> Implementation of anti-DID XYZ field-map reader
      return;
    }
```

Now use your favourite editor (i.e., emacs) to open the file and pick one, or the other solution, or merge the two things into one that is different from both. (Emacs offers the `smerge-ediff` mode to give a nice graphical interface to resolve merge conflicts.)

After we resolved the conflict git status will show
```
(AntiDIDFieldMap +|REBASE 1/10)$ git st
rebase in progress; onto cd32419
You are currently rebasing branch 'AntiDIDFieldMap' on 'cd32419'.
  (all conflicts fixed: run "git rebase --continue")

Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        modified:   detector/include/FieldMapXYZ.h
        modified:   detector/other/FieldMapXYZ.cpp
```

(Note that this was done with git version 2.9.2, in older versions of git an additional `git add detector/other/FieldMapXYZ.cpp` might be necessary)

In this case we had to change the `FieldMapXYZ.cpp` to incorporate the changes from master with ours. So now we continue with the rebase
```
(AntiDIDFieldMap +|REBASE 1/10)$ git rebase --continue
Applying: Implementation of anti-DID XYZ field-map reader
Applying: Use of anti-DID XYZ field-map reader
Applying: New solenoid and anti-DID field maps (Feb. 23th 2017)
Applying: Little but fix in specification of solenoid and anti-DID field maps
Applying: Little but fix in specification of solenoid and anti-DID field maps
Applying: Little bug fix in FieldMapXYZ
Applying: Giving different names to Solenoid and antiDID field-maps
Applying: Added some print-outs in case of errors
Applying: Added some print-outs in case of errors
Using index info to reconstruct a base tree...
M       detector/other/FieldMapBrBz.cpp
Falling back to patching base and 3-way merge...
Auto-merging detector/other/FieldMapBrBz.cpp
Applying: Let constant B-field as defaut and leave field maps, both solenoid and anti-DID, commented out
2017-03-31 13:22 sailer@pclcd17:/data/sailer/software/ILCSOFT/HEAD-2016-12-05/lcgeo/HEAD (AntiDIDFieldMap)$
```
And it finishes successful until the end. If there are more conflicts they need to be resolved as shown before.
Note now that "git-prompt" no longer shows the "REBASE" comment as we finished our rebase.

It can also happen that we only want to pick the changes from master, and our own commit has become useless.
In that case the commit is empty and instead of `git rebase --continue` we need to use `git rebase --skip` to advance to the next step.







## git reflog

### undo merge/rebase/whatever

## git stash

`pull`, `rebase`, `checkout` might not be allowed if there are uncommited changes in the working directory. To quickly get rid of the changes there is `git stash`.
I.e.:
* `git stash` to move all changes into a stash at the current commit, equivalent to `git stash save`
* `git stash pop` to apply the latest stash to the current working tree.

There can be more than one stash at a time:

`git stash` shows all stashes, `git stash drop stash@{0}` can be used to remove a stash without applying it

