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
                                                       (index)  >   remote 
                                                                  repository
            <<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<< changes
### git files lifecycle
    untracked   >   
                    unmodified    >   modified    >   staged    >   unmodified

        git add >>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
                        git add >>>>>>>>>>>>>>>>>>>>>>>>>
                                                        git push >>>>>>>>
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

