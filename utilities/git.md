# Contents

- [concept](#concept)
- [configuration](#configuration)
    - [aliases](#aliases)
    - [folder structure](#folder-structure)
    - [gitignore](#gitignore)
        - [check why file ignored](#check-why-file-ignored)
        - [exclude w/o .gitignore](#exclude-wo-gitignore)
    - [hide utracked files](#hide-utracked-files)
    - [test connection](#test-connection)
    - [aliases](#aliases-2)
- [usage](#usage)
    - [branch](#branch)
        - [change base](#change-base)
        - [delete local branch](#delete-local-branch)
        - [delete remote branch](#delete-remote-branch)
        - [delete file from all commits history](#delete-file-from-all-commits-history)
    - [rename branch](#rename-branch)
        - [set upstream](#set-upstream)
        - [show graph](#show-graph)
    - [checkout](#checkout)
        - [one file](#one-file)
        - [remote branch](#remote-branch)
    - [commit](#commit)
        - [squash two commits](#squash-two-commits)
        - [show files in commit](#show-files-in-commit)
        - [sign](#sign)
        - [commit in wrong branch](#commit-in-wrong-branch)
    - [diff](#diff)
        - [local from remote](#local-from-remote)
        - [remote from local](#remote-from-local)
        - [with branch](#with-branch)
    - [fetch](#fetch)
        - [fetch remote branch w/o checkout](#fetch-remote-branch-wo-checkout)
    - [index](#index)
        - [show files in index](#show-files-in-index)
        - [show ignored files](#show-ignored-files)
        - [remove file from index](#remove-file-from-index)
        - [remove all files from index](#remove-all-files-from-index)
        - [restore deleted file](#restore-deleted-file)
        - [restore file to state 2 commits back](#restore-file-to-state-2-commits-back)
        - [skipping files](#skipping-files)
        - [undo add to stage](#undo-add-to-stage)
    - [remote](#remote)
        - [add remote with non-default ssh port](#add-remote-with-non-default-ssh-port)
        - [pull from remote force](#pull-from-remote-force)
        - [push to remote](#push-to-remote)
        - [push to all remotes](#push-to-all-remotes)
        - [push to remote diff name](#push-to-remote-diff-name)
        - [rename all remotes](#rename-all-remotes)
    - [repo](#repo)
        - [init](#init)
    - [scripting](#scripting)
        - [paths](#paths)
    - [show](#show)
    - [stash](#stash)
        - [apply 1 file](#apply-1-file)
        - [clear](#clear)
        - [show diff](#show-diff)
    - [submodules](#submodules)
        - [update](#update)
    - [urls](#urls)
        - [change remote url](#change-remote-url)
    - [debug](#debug)

# concept
* **workspace** is the directory tree of (source) files that you see and edit
* **index** is a single, large, binary file in `.git/index`, which lists all files in the current branch, their sha1 checksums, time stamps and the file name
* **local repository** is a hidden directory (`.git`) including an objects directory containing all versions of every file in the repo (local branches and copies of remote branches) as a compressed "blob" file

```

      +------commit -a---------->>+
      |                           |
      +------add-->>+---commit-->>+
      |             |             +-----push---->>+
      |             |             |               |
 +----+----+      +-+---+     +---+------+    +---+-------+
 |workspace|      |index|     |local repo|    |remote repo| 
 +----+----+      +-+---+     +-------+--+    +---+-------+
      |             |                 |           |
      +<<------------pull or rebase---------------+
      |             |                 |           |    
      |             |                 +<<--fetch--+
      |             |                 |
      +<<-----checkout HEAD-----------+
      |             |                 |
      +<<-checkout--+                 |
      |             |                 |
      +<------diff HEAD-------------->+
      |             |
      +<---diff---->+   

```                                   

Everything in git is reffered by hashes.

e.g. after commiting 1 file, there will be 3 objects:
- archived file itself, with its hash as a name
- snapshot of working tree, with its hash
- commit itself, also with hash

Each **commit** object contains:
- hash of working tree snapshot, e.g. `tree <hash>`
- comment
- author/creator info
- parent commit hash

`HEAD`, branches and tags are just refs to the appropriate **commit hash**, i.e. deletion of branch doesn't lead to deletion of other objects.

# configuration

## aliases
`git config --global alias.<name> '<command>'`
`<command>`: for shell command use `!` *?*
e.g.
`git config --global alias.pushall '!git remote | xargs -L1 git push --all'`

## folder structure
* `HEAD` contains path to `refs` and the hash of commit
* `branches`
* `config` contains author info, repos urls, etc
* `description` used by `gitweb`
* `hooks` scripts to launch before/after several actions: push/pull/commit, etc
* `info/`
    * `exclude` similar to [[#gitignore|.gitignore]] but won't be included in commit
* `objects` archived files/commits/snapshots with their hashes as names. *NOTE:* first two symbols of hash are the name of the subfolder.
* `refs/`
    * `refs`
    * `tags`

## gitignore
```
  # exclude everything
  *
  # except
  !*.extension
  # even if they are in subfolders
  !*/
```
### check why file ignored
`git check-ignore -v $filename`

### exclude w/o .gitignore
`.git/info/exclude`

## hide utracked files
`git config --local status.showUntrackedFiles no`
## test connection
`ssh -T git@<address>`

## aliases
`git config --global alias.<name> '<command'>`

e.g.   

`git config --global alias.pushall '!git remote | xargs -L1 git push --all'`

# usage
## branch

### change base
`git rebase <new_base>`
`git rebase <new_base> <old_base>`

### delete local branch
`git branch -d $branch`

### delete remote branch
`git push origin --delete $remote_branch`

### delete file from all commits history
`git filter-branch --tree-filter "rm -f <file>" -- --all`

## rename branch
`git branch -m <oldname> <newname>`   

if branch is already checkouted:   

`git branch -M <newname>`

### set upstream
`git branch --set-upstream-to remote/branch-name`

### show graph
`git log --all --decorate --oneline --graph`   

mnemonic: **A**ll **D**ecorate **O**neline **G**raph - **a dog**

## checkout
### one file
`git checkout $commit_hash -- $filename`

if file doesn't exist in current branch   

`git show TREESH:/path/to/file > /path/to/file`
### remote branch
`git checkout -t <remote>/<branch>`   
`git checkout -b <local_branch> <remote>/<branch>`

## commit
### squash two commits
`git rebase --interactive HEAD~2`   

### show files in commit
`git diff-tree --no-commit-id --name-only -r <commit>`

for the latest commit change `pick` to `squash`, e.g.   
```
  pick <hash> commit1
  pick <hash> commit2
```
then edit combined message for new commit
```
  message of commit1
  message of commit2
```
### sign
* `gpg -k --keyid-format long`  
* `git config user.signingkey <keyid>!`  
* `git config commit.gpgsign true`
`!` forces to use particular gpg key  
identities don't have to be the same  

### commit in wrong branch
* `git reset --soft origin/<wrong_branch>` roll-back changes to worktree
* `git checkout -b <correct_branch> && git add ...` follow as usual

git branch <correct_branch>  
## diff
### local from remote
`git log <branch> ^<remote/branch>`
### remote from local
`git log ^<branch> <remote/branch>`
### with branch
`git diff --name-only $branch`

## fetch
fetch downloads remote objects

### fetch remote branch w/o checkout
`git fetch <remote> <remote_branch>:<local_branch>`

## index
### show files in index
`git ls-tree -r master --name-only`
### show ignored files
`git status --ignored`
### remove file from index
`git reset $file`
### remove all files from index
`git reset`
### restore deleted file
`git restore <file>`
### restore file to state 2 commits back
`git restore --source branch~2 <file>`
### skipping files
`git update-index`
1. `--assume-unchanged`   
Git assumes that files with this bit are not changed by user and skips their check 
2. `--skip-worktree`   
Git assumes that working copy of files with this bit is up to date with index and use version from the index. Meanwhile the file in the working tree can be different or even absent. 

in both cases Git:   
- won't commit this file (even if it is added explicitly)
- _can_ update working copy of the file if it's changed on remote side
- will show merge conflict if file is changed on both sides
### undo add to stage
`git restore --staged <file>`

## remote
### add remote with non-default ssh port
`get remote add <name> ssh://git@<address>:<ssh_port>/<username>/<repo>.git`
### pull from remote force
`git reset --hard <remote>/<branch>`
### push to remote
`git push -u $remote $branch`
### push to all remotes
`git remote | xargs -L1 git push --all`
### push to remote diff name
`git push -u $remote HEAD:$remote_branch`
### rename all remotes
`rg --hidden 'remote "<old_name>"' --with-filename -c | sed s/config:[0-9]// | xargs -I{} sh -c 'cd {} && git remote rename <old_name> <new_name> && cd <workdir>'`

## repo
### init
* `git init <name>` creates **working** repository with a **working tree**.
Supposed to be used when files in repository will be modified, i.e. common workflow case.
* `git init --bare <name>` creates **bare** repository without **working tree**.
Supposed to be used as centralized point to store changes from remote repos, i.e. for **sharing**.   

Assume it doesn't have a working tree, there will be no conflicts on pushing files into.   
That's how remote servers, e.g. GitHub, work.

## scripting
### paths
* `git rev-parse --show-toplevel` shows root dir in git working tree
* `git rev-parse --show-superproject-working-tree` if current repo is submodule, shows root dir of main repo
* `git rev-parse --show-prefix` shows relative path from root folder of repo

## show
`git show branch:file` shows file from another branch

## stash
### apply 1 file
`git checkout stash@{0} -- $file`
### clear
`git stash clear`
### show diff
`git stash show` use `-p` option to show diff

## submodules
### update
`git submodule update`
- `--init` initialize all submodules, which haven't been yet (also resets them to state in latest commit)
- `--checkout` checkout on commit, recorded in superproject (detached head)
- `--rebase` rebase on commit, recorded in superproject (no detached head)
- `--recursive` update nested submodules
- `--remote` use the status of the submodule's remote-tracking branch (branch name is taken from .gitmodules)

e.g.   
* checkout all submodules on commits in super-repo `git submodule update --recursive --checkout`
* update all submodules to latest commits in remote repos `git submodule update --remote --recursive --checkout`

## urls
### change remote url
`git remote set-url $remote $new_url`

## debug
add `GIT_TRACE=1` before command
