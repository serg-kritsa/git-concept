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

