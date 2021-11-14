## initialize new repository
git init
ls -la
### inner git structure (used by git only)
ls .git
    config  description  HEAD  hooks/  info/  objects/  refs/
cat config
cat description
cat HEAD

## keep empty folder in git
mkdir empty-folder && cd empty-folder && touch .gitkeep && cd ..

## create blob object
git hash-object
echo Hello Git | git hash-object --stdin            returns hash of passed string
echo Hello Git | git hash-object --stdin -w         -w      to write it in `.git/objects/##` ## - 1st two letters as folder name   

## setup user info
git config --global user.name <name>
git config --global user.email <email>
git config --list

### git layers
    working directory                             >   staging 
                                                        area 
                                                       (index)  >   local 
                                                                  repository
            <<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<< changes
### git files lifecycle
    untracked   >   
                    unmodified    >   modified    >   staged    >   unmodified

        git add >>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
                        git add >>>>>>>>>>>>>>>>>>>>>>>>>
                                                        git commit >>>>>>
                                        <<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<< changes
                                        git add >>>>>>>>>


git status
## add to stage specified file
git add <filename>
### add to stage all untracked/modified files
git add .
### see changes in status
git status
### see git files
    find .git/objects -type f                           files w/ relative path
    git ls-files                                        see staged area as list
    git ls-files -s                                     see staged area as table
## unstage not commited file
git reset HEAD <filename>
## untrack not commited file using git rm
git rm --cached <filename>
    git ls-files -s                                     not in staged area
    git status                                          is untracked

## commit staged changes
git commit -m "<description>"
    git creates commit object, blob object for file and tree object
### see commit history
git log
### explore commit object
git cat-file -p <commit-hash-####>                  hash of tree object,     hash of parent commit (if NOT first),    author,     committer,     description
git cat-file -s <commit-hash-####>                  object size
git cat-file -t <commit-hash-####>                  commit

git cat-file -p <tree-hash-####>                    list of blob in tree object

## go to other commit from unmodified state
git checkout <commit-hash-######>                   move to specified commit
git checkout <branchname>                           return on the top of branch
### EXAMPLE: go to the prev commit
cat .git/HEAD                                       ref: refs/heads/master
cat .git/refs/heads/master                          sha1-hash
git cat-file -p <commit-hash-####>                  copy hash of parent commit
git checkout <commit-hash-######>                   move HEAD pointer to parent commit 
                                                    cat .git/HEAD returns sha1-hash instead of ref to branch >>> detached HEAD warning
                                                    cat .git/refs/heads/master remains the same
                                                    ls -l returns files at that moment
                                                    git ls-files -s shows files in staged area
git checkout master                                 returns on the top of branch

## manage branch 
git branch                                       list all local branches (current marked w/ '* ')
git branch <branchname>                          creates new local branch
git checkout <branchname>                        move to specified branch
git branch -d <branchname>                       delete specified branch
git branch -m <old> <new>                        rename specified branch

git checkout -b <branchname>                     move to just created branch (SHORTCUT)
    git branch <branchname>                          creates new local branch
    git checkout <branchname>                        move to specified branch

### EXAMPLE: creation
git branch temp                                  creates temp branch
    ls .git/refs/heads                           master temp
    cat .git/refs/heads/master                   sha1
    cat .git/refs/heads/temp                     sha1 (the same means that points to the same commit)
    cat .git/HEAD                                ref: refs/heads/master
git checkout temp                                Switched to branch 'temp'
    cat .git/HEAD                                ref: refs/heads/temp
    cat .git/refs/heads/temp                     sha1 (the same means that points to the same commit)
git checkout master                              Switched to branch 'master'
    cat .git/HEAD                                ref: refs/heads/master
git branch -m temp new-temp                      
git branch                                       * master
                                                 new-temp
git checkout new-temp                            Switched to branch 'new-temp'
git checkout master                              Switched to branch 'master'
git branch -d new-temp                           Deleted branch new-temp (was #######)

### EXAMPLE: commit
git checkout -b BR-1                            move to just created branch
echo Hello, Git from BR-1 > file.txt
git add file.txt
git status                                      shown branchname on top
git commit -m "to BR-1"
    git status                                  shown branchname on top
    ls                                          files in working dir
    git ls-files -s                             files in stage (the same number of files)
git log                                         all commits
    git cat-file -p <commit-hash-######>
        git cat-file -p <parent-commit-hash-######>     commit where branch was created

## clone repository
git clone https://github.com/<username>/<reponame>.git
    cd reponame
    git log
    ls .git/objects/                             info pack                         
### unpack cloned repository
    cd .git/objects/pack                         
        ls                                      *.idx pack-<hash>.pack (archived)
        cat pack-<hash>.pack | git unpack-objects (unzip archive in .git/objects/)
        rm -f pack-<hash>.pack
    
## show what is modified (but NOT staged)
git diff                                        to see changes not added to stage 
    index <sha1-hash>..<provisional-hash-for-modified-file> 100755
    @@ -3,6 +3,8 @@                             inserted
        ^ old-chunk-stats-here          |
          ^ old-chunk-ends-here         |
             ^ new-chunk-stats-here     |
               ^ new-chunk-ends-here    |   means added 2 lines
    then
    @@ -11,7 +13,7 @@                                           edited
        ^ old-chunk-stats-here                          |
          ^ old-chunk-ends-in-this-number-of-lines      |
             ^ new-chunk-stats-here                     |
               ^ new-chunk-in-this-number-of-lines      |   means line edited

## merge branches
### OVERVIEW: fast-forward merge branches 
BEFORE
                HEAD
                master
commit  commit  commit  
                    |-  commit  commit  commit  
                    |                   feature1

AFTER
commit  commit  commit  
                    |                   HEAD
                    |                   master
                    |-  commit  commit  commit  

### Example: fast-forward merge branches
- created <new-branch-name> from master 
- created commit in <new-branch-name>
- ```IMPORTANT!``` no commits were made in master since creation <new-branch-name>
git checkout master
git merge <new-branch-name>                         actually moving pointer
git branch -d <new-branch-name>                     delete as not needed anymore

### OVERVIEW: 3-way merge branches 
BEFORE
                base            HEAD
                v               master
commit  commit  commit  commit  commit  
                    |
                    |-  commit  commit  commit  
                    |                   feature1

AFTER
                base            HEAD
                v               master                 new merge
commit  commit  commit  commit  commit --------------- commit  
                    |                                    |
                    |-  commit  commit  commit ----------- 
                    |                   feature1

### Example: 3-way merge branches 
- created <new-branch-name> from master 
- created commits in <new-branch-name>
- created commits in master
git checkout master
git merge <new-branch-name>                         enter message for new merge commit
    git log                                         <new-branch-name> pointer is shown
    git branch -d <new-branch-name>                 delete as not needed anymore (at `git log` <new-branch-name> pointer will disappear)

### Example: resolving conflict
- created <new-branch-name> from master 
- committed changes in some files in <new-branch-name>
- committed changes in the same files in master
git checkout master
git merge <new-branch-name>                         merge conflict state
    git status                                      #1: shown files where conflict is
    git ls-files -s                                 #2: conflicted file printed 3 times (used by git indexes 
                                                        1->base_version 
                                                        2->from_current_branch 
                                                        3->from_merged ). for not in conflict used 0
    nano filename                                   printed conflicted area w/ <<<<<<<< ============ >>>>>>>> . 
        HINT: Ctrl+K to toremove whole line if needed                                                   
        edit file to resolve conflict and continue merging
    git add <filname>
    git ls-files -s                                 #2: resolved file should receive index 0 
    git status                                      All conflicts fixed but you are still merging
    git commit
                                                    INFO: .git/MERGE_HEAD keeps the last commit-hash in feature branch         
        HINT: if vim editor
            i to enter INSERT mode > edit commit message if needed > Esc to exit INSERT mode
            :wq to write changes and quit
    git status                                      nothing to commit, working tree clean


### git layers w/ remote repository
    working directory                             >   staging 
                                                        area 
                                                       (index)  >   local 
                                                                  repository            >   remote
                                                                                          repository

        <<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<< changes
### git files lifecycle
    untracked   >   
                    unmodified    >   modified    >   staged    >   unmodified

        git add >>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
                        git add >>>>>>>>>>>>>>>>>>>>>>>>>
                                                        git commit >>>>
                                        <<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<< changes
                                        git add >>>>>>>>>

                                                                        git push >>>>>>>>>>>>>
                                                                        <<<<<<<<<<<< git fetch
        <<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<< git pull

# remote branch management
## show remote repositories (cloned)
git remote                                          `origin`    
git remote -v                                       `origin https://github.com/`<username>`/`<reponame>`.git (fetch)`
                                                    `origin https://github.com/`<username>`/`<reponame>`.git (push)`  
git remote show origin                              detailed branch config info    

## show branches 
git branch                                          local 
git branch -vv                                      show local branches corrisponding remotes
                                                    after cloning git has master branch only that correspond remote 
git branch -r                                       remote (also created in github UI )
git branch -a                                       local & remote (maybe not equal)

### add not existing branch in local repository
git checkout <remote-branch-from-cloned-repository>
git branch -vv                                      <remote-branch-from-cloned-repository> will be shown

### delete branch in local repository
git checkout master
git branch -d <remote-branch-from-cloned-repository>
git branch

## add new branch in remote repository to local one
git fetch 
## remove deleted in remote repository branch from local one
git remote show origin                              deleted remote branch marked as stale
git remote prune origin                             `* [pruned] origin/`<deleted-branch-remote-repository>
    git branch -vv                                  deleted remote branch marked as gone
    git checkout master
    git branch -d <deleted-branch-remote-repository>
    git branch -vv                                  deleted remote branch not shown
    git remote show origin                          deleted remote branch not shown

## git pull overview
git branch -vv                                      to make sure that remote branches corresponding local ones
git fetch                                           updating FETCH_HEAD w/ sha1 of last commits in remote repository for all tracking branches
          -v                                        `= [up to date]  master  origin/master` ...  
    cat .git/FETCH_HEAD                             some branch marked as `not-for-merge` 
git merge FETCH_HEAD

## EXAMPLE: git pull
- added commit in <remote-branch-name>
git checkout <remote-branch-name>
git pull -v
    git ls-files -s                                 see staged files
    git log

## set upstream branch
- created <new-local-branch>
- added commit to it
git branch -vv                                      no corrisponding branch for <new-local-branch>
git push -v                                         `fatal: The current branch `<new-local-branch>` has no upstream branch.`
git push -v --set-upstream origin <new-local-branch>
    git push -v -u origin <new-local-branch>        shortcut. the same             
    git branch -vv                                  corrisponding branch for <new-local-branch>


