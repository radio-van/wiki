# Contents

- [concept](#concept)
- [configuration](#configuration)
    - [aliases](#configuration#aliases)
    - [folder structure](#configuration#folder structure)
    - [gitignore](#configuration#gitignore)
        - [check why file ignored](#configuration#gitignore#check why file ignored)
        - [exclude w/o .gitignore](#configuration#gitignore#exclude w/o .gitignore)
    - [hide utracked files](#configuration#hide utracked files)
    - [test connection](#configuration#test connection)
    - [protocols](#configuration#protocols)
- [usage](#usage)
    - [branch](#usage#branch)
        - [reflog](#usage#branch#reflog)
        - [rebase](#usage#branch#rebase)
            - [basic usage](#usage#branch#rebase#basic usage)
            - [interactive usage](#usage#branch#rebase#interactive usage)
            - [revert rebase](#usage#branch#rebase#revert rebase)
        - [delete local branch](#usage#branch#delete local branch)
        - [delete remote branch](#usage#branch#delete remote branch)
        - [delete file from all commits history](#usage#branch#delete file from all commits history)
        - [rename branch](#usage#branch#rename branch)
        - [rename remote branch](#usage#branch#rename remote branch)
        - [set upstream](#usage#branch#set upstream)
        - [show graph](#usage#branch#show graph)
    - [checkout](#usage#checkout)
        - [last used branch](#usage#checkout#last used branch)
        - [one file](#usage#checkout#one file)
        - [remote branch](#usage#checkout#remote branch)
        - [separate worktree](#usage#checkout#separate worktree)
    - [clone](#usage#clone)
        - [clone to repo under specific directory](#usage#clone#clone to repo under specific directory)
    - [commit](#usage#commit)
        - [search for string in whole git history](#usage#commit#search for string in whole git history)
        - [squash commits](#usage#commit#squash commits)
        - [show files in commit](#usage#commit#show files in commit)
        - [undo amend](#usage#commit#undo amend)
        - [commit in wrong branch](#usage#commit#commit in wrong branch)
            - [one file](#usage#commit#commit in wrong branch#one file)
            - [more commits](#usage#commit#commit in wrong branch#more commits)
        - [commit part of the file](#usage#commit#commit part of the file)
        - [split commit](#usage#commit#split commit)
    - [sign](#usage#sign)
    - [diff](#usage#diff)
        - [local from remote](#usage#diff#local from remote)
        - [remote from local](#usage#diff#remote from local)
        - [with branch](#usage#diff#with branch)
        - [calculate difference](#usage#diff#calculate difference)
    - [fetch](#usage#fetch)
        - [clean local cache](#usage#fetch#clean local cache)
        - [fetch remote branch w/o checkout](#usage#fetch#fetch remote branch w/o checkout)
    - [index](#usage#index)
        - [show files in index](#usage#index#show files in index)
        - [show ignored files](#usage#index#show ignored files)
        - [show repo size](#usage#index#show repo size)
        - [show biggest files in index](#usage#index#show biggest files in index)
        - [remove file from index](#usage#index#remove file from index)
        - [remove all files from index](#usage#index#remove all files from index)
        - [remove file from entire git history](#usage#index#remove file from entire git history)
        - [restore deleted file](#usage#index#restore deleted file)
        - [restore file to state 2 commits back](#usage#index#restore file to state 2 commits back)
        - [skipping files](#usage#index#skipping files)
        - [undo add to stage](#usage#index#undo add to stage)
    - [merge](#usage#merge)
        - [force to pick files from particular branch](#usage#merge#force to pick files from particular branch)
    - [remote](#usage#remote)
        - [add remote with non-default ssh port](#usage#remote#add remote with non-default ssh port)
        - [pull from remote force](#usage#remote#pull from remote force)
        - [pull with submodules](#usage#remote#pull with submodules)
        - [push to remote](#usage#remote#push to remote)
        - [push to all remotes](#usage#remote#push to all remotes)
        - [push to remote diff name](#usage#remote#push to remote diff name)
        - [push even if up-to-date](#usage#remote#push even if up-to-date)
        - [rename all remotes](#usage#remote#rename all remotes)
    - [repo](#usage#repo)
        - [init](#usage#repo#init)
    - [scripting](#usage#scripting)
        - [paths](#usage#scripting#paths)
    - [show](#usage#show)
    - [stash](#usage#stash)
        - [apply 1 file](#usage#stash#apply 1 file)
        - [apply and clear](#usage#stash#apply and clear)
        - [clear](#usage#stash#clear)
        - [show diff](#usage#stash#show diff)
    - [submodules](#usage#submodules)
        - [add](#usage#submodules#add)
        - [sync](#usage#submodules#sync)
        - [update](#usage#submodules#update)
        - [recursive push](#usage#submodules#recursive push)
    - [tags](#usage#tags)
        - [push local tags](#usage#tags#push local tags)
    - [urls](#usage#urls)
        - [change remote url](#usage#urls#change remote url)
    - [debug](#usage#debug)

# concept
Git is a graph of commits with connections.

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

## protocols
By default `http, https, git, ssh, file` are allowed if no config specified.  
`git config --global --add protocol.<name>.allow = [always, never, user]`

**NOTE**: if any protocol is specified, default ones are not allowed automatically.

# usage
## branch
Branches are just references on particular commits. When a new commit is made, branch pointer advances on that commit.  
Local branches are stored in `.git/refs/heads/`.  
Remote branches are stored in `.git/refs/remotes/`.

### reflog
Log of all `HEAD` movements.

Despite the fact that *remote* branches are stored locally, they can't be changed directly, only as side effect of `git-push` or `git-fetch`.

### rebase
**rebase** changes parent commit of the branch to the head commit of given branch.
```
C0 <- C1 <- C2 (master)
       \               
        C4 (feature)
```
`git checkout feature && git rebase master`
```
C0 <- C1 <- C2 (master)
             \               
              C4' (feature)
```
if `C2` and `C4` have conflicts, then after resolving them, `C4` commit will be changed (i.e. `C4'`)

#### basic usage
`git rebase <new_base>`
`git rebase <new_base> <old_base>`

#### interactive usage
  - `git rebase --interactive <hash>` from the given commit
  - `git rebase --interactive --root` from the very beginning
it will call text editor with all commits and gives opportunities to:
* `pick` save commit as is
* `reword` change commit's description
* `squash` with previous commit (previous commit is on the bottom line)
* `drop` delete commit
* `edit` gives the same state as right after staging changes in work dir (i.e. it's possible to modify files, add more commits, etc.)

`git rebase --abort` cancels rebase process

to separate changes in one file `git rebase --interactive` + `git add --patch` can be used

#### revert rebase
* `git reflog` -> search for rebase hunk
* `git reset --hard HEAD@{<hunk>}`

### delete local branch
`git branch -d $branch`

### delete remote branch
`git push origin --delete $remote_branch`

### delete file from all commits history
`git filter-branch --tree-filter "rm -f <file>" -- --all`

### rename branch
`git branch -m <oldname> <newname>`   

if branch is already checkouted:   

`git branch -M <newname>`

### rename remote branch
* [delete it](#delete remote branch)
* push new one

### set upstream
`git branch --set-upstream-to remote/branch-name`

### show graph
`git log --all --decorate --oneline --graph`   

mnemonic: **A**ll **D**ecorate **O**neline **G**raph - **a dog**

## checkout
### last used branch
`git checkout -`

### one file
`git checkout $commit_hash -- $filename`

if file doesn't exist in current branch   

`git show TREESH:/path/to/file > /path/to/file`

### remote branch
`git checkout -t <remote>/<branch>`   
`git checkout -b <local_branch> <remote>/<branch>`

### separate worktree
Creates second worktree, independend from the main.
Useful when checkout is difficult (e.g. a lot of temp files)
and cloning is time consumable.
`git worktree add -b <branch> <path> <source-branch>`

to remove:
`git worktree remove <path>`

## clone
### clone to repo under specific directory
`git -C <dir> clone <repo>`

## commit
### search for string in whole git history
`git log -S <string>`

### squash commits
- `git reset --hard <initial commit>` resets work dir to state of choosen commit
- `git merge --squash HEAD@{1}` merges commits into one from current (initial) to last (`HEAD@{1}` refers to previous `HEAD` position)
- `git commit`

or use  
`git rebase --interactive HEAD~X`   

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

### undo amend
`git reset --soft HEAD@{1}`  
*NOTE*: `HEAD@{1}` = commit that `HEAD` pointed at before it was moved to anywhere it is pointed now

### commit in wrong branch
#### one file
* `git reset --soft origin/<wrong_branch>` roll-back changes to worktree
* `git checkout -b <correct_branch> && git add ...` follow as usual
#### more commits
* remember **hash** of current commit
* `git reset --hard <origin/branch | HEAD~X>`
* `git checkout -b <new branch>`
* `git reset --hard <remembered hash>`

### commit part of the file
`git add --patch <file>`

### split commit
* `rebase` and mark commit for editing `e`
* `git reset HEAD^`
* `git add ...`
* `git commit -c ORIG_HEAD` (to keep original message)

## sign
* `gpg -k --keyid-format long`  
* `git config user.signingkey <keyid>!`  
* `git config commit.gpgsign true`
`!` forces to use particular gpg key  
identities don't have to be the same  

git branch <correct_branch>  
## diff
### local from remote
`git log <branch> ^<remote/branch>`
### remote from local
`git log ^<branch> <remote/branch>`
### with branch
`git diff --name-only $branch`
### calculate difference
`git diff --stat|--shortstat $branch`

## fetch
fetch downloads remote objects

### clean local cache
`git fetch --prune <remote>`

### fetch remote branch w/o checkout
`git fetch <remote> <remote_branch>:<local_branch>`


## index

### show files in index
`git ls-tree -r master --name-only`

### show ignored files
`git status --ignored`

### show repo size
```
git gc
git count-objects -vH
```

### show biggest files in index
`git ls-tree -r -t -l --full-name HEAD | sort -n -k 4 | tail -n 10`  
where:  
- `-r` for recursive
- `-t` show recurse items
- `-l` show size
- `sort -n -k 4` sorts by 4th column (size) and assumes the column is numeric 

### remove file from index
unstage file aka undo `git add`  
`git reset $file`

### remove all files from index
`git reset`

### remove file from entire git history
**WARNING**: it's dangerous and performance-cost operation. All commits will be changed.  
`git filter-branch --index-filter "git rm -rf --cached --ignore-unmatch <path_to_file>" HEAD`

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

## merge
### force to pick files from particular branch
assume that there is a `newbranch` that can't be merged into `oldbranch`  
because of conflicts but it's okay to sacrifice changes in `oldbranch`  
i.e. force merge and override.
```
git checkout newbranch
git merge -s ours oldbranch  # merges in the old branch, but keeps all files
git checkout oldbranch
get merge newbranch
```

## remote
### add remote with non-default ssh port
`get remote add <name> ssh://git@<address>:<ssh_port>/<username>/<repo>.git`
### pull from remote force
`git reset --hard <remote>/<branch>`
### pull with submodules
`git pull --recurse-submodules` pulls main repo and all submodules (i.e. submodules will be on their lates commits)
### push to remote
`git push -u $remote $branch`
* `-f` rewrites remote history with local
* `--force-with-lease` does the same but fails if remote repo contains another participant's commits
**NOTE**: commits deleted by `--force` are not _deleted_, to clean repo from 'ghosts' use `git gc --prune`

### push to all remotes
`git remote | xargs -L1 git push --all`
### push to remote diff name
`git push -u $remote HEAD:$remote_branch`
### push even if up-to-date
`git commit --allow-empty -m 'push to execute post-receive'`

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
### apply and clear
`git stash pop`
### clear
`git stash clear`
### show diff
`git stash show` use `-p` option to show diff

## submodules
### add
`git submodule add -b <branch> <git url> <local path>`

### sync
`git submodule sync -- <submodule>`
sets upstream URL of current remote to the one specified in `.gitmodules`.
**NOTE**: it overrides URL of current remote, not adds the new one

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

### recursive push
`git push --recurse-submodules=on-demand` will push all submodules (if submodule's state behind remote).  
**NOTE**: This will not automatically create new commits.

## tags
Tags are technically the same as [branches](#branch), i.e. just pointers to particular commits, but are not movable (*branches* are movable).

### push local tags
`git push --tags`

## urls
### change remote url
`git remote set-url $remote $new_url`

## debug
add `GIT_TRACE=1` before command
