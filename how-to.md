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



## what changed in feature in comparison w/ master
git diff master <branch-name>                         shown chunk 
git diff master..<branch-name>                        alisas. output the same 
git diff master...<branch-name>                       differences from common commit 

git diff <commit-hash-1> <commit-hash-2>              changes between specified commits
git diff <commit-hash-1>..<commit-hash-2>             alias

git diff                                              changes in working directory in comparison w/ stage  
git diff HEAD                                         changes in working directory in comparison w/ local repository  
git diff @                                            alias

### EXAMPLE #1
- change index.html
git diff                                              changes in working directory in comparison w/ stage  
git add index.html
git diff                                              no changes. all staged
- change index.html
git diff                                              NEW changes in working directory in comparison w/ stage  
### EXAMPLE #2
git diff --staged                                     changes in stage in comparison w/ local repository. HEAD by default  
git diff --cached                                     alias
                                                      in `git commit -v`  added output from `git diff --staged`
                                                      `git config --global commit.verbose true`
git diff --staged <commit-hash>                     changes in stage in comparison w/ <commit-hash>  
### EXAMPLE #3
git diff <filename>
    git diff -- <filename>                           add `--` if it's used master, HEAD, etc for filename
git diff HEAD <filename>
git diff <branch-name-1> <branch-name-2> <filename>
git diff <branch-name-1> <branch-name-2> <filename-1> <filename-2>
git diff --name-only <branch-name-1> <branch-name-2>



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


## remove remote branch tracking
- created <new-remote-branch>
git branch -a                                       <new-remote-branch> not shown        
git fetch                                           `* [new branch]     `<new-remote-branch>`  -> origin/`<new-remote-branch>
git branch -a                                       <new-remote-branch> not shown locally, shown remotely        
git checkout <new-remote-branch>                    local branch tracks remote one
    git branch -a                                   <new-remote-branch> shown locally & remotely        
    git branch -vv                                  local <new-remote-branch> corresponds remote ones         
- deleted <new-remote-branch> on github
    git branch -a                                   <new-remote-branch> shown locally & remotely BUT remote deleted       
    git branch -vv                                  <new-remote-branch> shown BUT remote deleted
git fetch -v                                        not shown deleted remote branch
    git branch -vv                                  <new-remote-branch> STILL tracks BUT remote deleted
git remote update origin                            `Fetching origin`
    git branch -vv                                  <new-remote-branch> STILL tracks BUT remote deleted
git remote update origin --prune                    `* [deleted]     (none)  -> origin/`<new-remote-branch>
    git branch -vv                                  <new-remote-branch> marked as `gone`
To delete locally
    git checkout master
    git branch -d <new-remote-branch>
    git branch -vv                                  not shown
    git remote show origin                          not shown    
To delete locally forced
    git branch -D <new-remote-branch>
    git branch -vv                                  not shown
    git remote show origin                          not shown    

## remove remote branch from local terminal
git checkout -b <new-local-branch>
git branch -a                                       <new-local-branch> not in remote repository
git push -u origin <new-local-branch>               <new-local-branch> tracks (corresponding) remote
git push -d origin <new-local-branch>               `- [deleted]         `<new-local-branch>
git branch -a                                       <new-local-branch> not in remote repository (just removed)
git checkout master
git branch -d <new-local-branch>

## show refs from .git/refs/
git show-ref                                        see all branch refs (SHORTCUT)
    ls .git/refs/heads
    ls .git/refs/remote/origin
git show-ref master                                 see local & remote master refs (SHORTCUT)
git log

## format git log
git help log

git log --pretty=oneline                            <commit-hash-full> <title>
git log --pretty=oneline --abbrev-commit            <commit-hash-######> <title>
git log --oneline                                   alias
git log --oneline -no-decorate                      + without refs

### format line
git log --pretty=format:'%h'                        <commit-hash-######>
git log --pretty=format:'%ci'                       <date-ISO 8601>         YYYY-MM-DD hh:mm:ss +-time-zone-dif
git log --pretty=format:'%cI'                       <date-ISO 8601>         YYYY-MM-DDThh:mm:ss+-time-zone-dif
git log --pretty=format:'%cs'                       YYYY-MM-DD
git log --pretty=format:'[%an]'                     <author> 
git log --pretty=format:'%d'                        <branch-head-ref> 
git log --pretty=format:'%s'                        <commit-title>
git log --pretty=format:'%h %ci | %s%d [%an]'       <commit-hash-######> <yyyy-m-d hh:mm> <title> HEAD <author> 
git log --pretty=format:'%cd' --date=format:'%F %R' <YYYY-MM-DD hh:mm>
git help config | grep 'Basic colors'
git log --pretty=format:'%C(yellow) %h'                                                 print in yellow           
git log --pretty=format:'%C(bold yellow) %h'                                            print in bold yellow           
git log --pretty=format:'%C(green) %d %C(reset) %s'                                     print param in green then by default                
git log --pretty=format:'%C(bold #225522) %h'                                           print in rgb-color

git config --global pretty.<my> format:'%C(yellow) %h %C(bold #225522) %d %C(reset) %s'   save format as <my>
git log --pretty=<my>                                                                     use saved format

git config --global format.pretty <my>              set format by default                                        
git log                                             used as default <my> in output 
git log --pretty=medium                             use format by default  

git config --global log.date short
git config --global log.date relative
git config --global log.date format:'%F %R'         UTC date as <YYYY-MM-DD hh:mm>
git config --global log.date format-local:'%F %R'   local date in <YYYY-MM-DD hh:mm>

### EXAMPLES #1: git log 
git log <branch-name>
git log <branch-name-1> <branch-name-2>             commits from both branches together
git log <branch-name-1> <branch-name-2> --graph     draw branches
git log --all --graph                               draw all branches BUT better to use GUI tool

git log <branch-name-1> ^<branch-name-2>            commits from <branch-name-1> after base commit
git log <branch-name-2>..<branch-name-1>            alias
git log HEAD..<branch-name>                         commits from <branch-name> not presented in HEAD-branch
git log ..<branch-name>                             alias
git log <branch-name>..                             commits from HEAD-branch not presented in <branch-name> 
git log <branch-name>.. --boundary                  + base commit 
git log <branch-name-1>...<branch-name-2> --boundary --graph     draw commits in specified branches after base commit that included

NOTE: .. ... in `git diff` have different meaning   

### EXAMPLES #2: git log 
git log <filename>                                  commits where <filename> was changed

git log <branch-name-2>..<branch-name-1> <filename>                 commits from <branch-name-1> after base commit WHERE <filename> was changed   
git log <branch-name-2>..<branch-name-1> <dirname-1>  <dirname-2>   commits from <branch-name-1> after base commit WHERE <dirname-1>  <dirname-2> was changed

### EXAMPLES #3: git log w/ regular expression
git log --grep <word>                               search by <word> in current branch
git log --grep <word-1>  --grep <word-2>            search by <word-1> OR <word-2> in current branch
git log --grep <word-1>  --grep <word-2> --all-match           search by <word-1> AND <word-2> in one commit title in current branch
git log --grep <word> <branch-name>                 search by <word> in <branch-name>

git log --grep '<reg-exp>'                          search by <reg-exp> (default old RegExpr pattern) in current branch
git log --grep '<reg-exp>' -P                       search by <reg-exp> (Perl pattern RegExpr) in current branch
git config  --global grep.patternType perl          set Perl pattern RegExpr by default

git log -G<reg-exp>                                 search by <reg-exp> in changes
git log -G<reg-exp> -p                              search by <reg-exp> in changes AND print found
git log -G'<reg-exp>' -p                            search by <reg-exp> in changes AND print found
    example: regexp using special symbols with \ before              git log -G'function handle\(' -p  
git log -G<reg-exp> -p <filename>                               search by <reg-exp> in changes in <filename> AND print found
git log -G<reg-exp> -p                              search by <reg-exp> in changes in <filename> AND print found


### EXAMPLES #3: git log
git log -L <start-line>,<end-line> <filename>       search from <start-line> to <end-line> in <filename> 
git log -L <start-re>,<end-re> <filename>           search from <start-line> to <end-line> in <filename> 
    example: regexp using special symbols with \ before              git log -L '<head>','<\/head>' index.html  

git log --author=<word>    
git log --committer=<word>    
git log --before '<YYYY-MM-DD>'    
git log --after '<YYYY-MM-DD>'    

### add alias
git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr)%C(bold blue)<%an>%Creset' --abbrev-commit"
git lg

## git blame
git blame <filename>
git blame <filename> --date=short
git blame <filename> --date=short -L 5,8

## git tags
### lightweignt tags
git tag <tagname>                                   lightweignt is stored in .git/refs/tags

git tag                                             show tags        
git show <tagname>                                  show detailed tag info        
ls .git/refs/tags
cat .git/refs/tags/<tagname>
git tag -v <tagname>                                NOTE: lightweignt has no additional tag info such as author and date       

### annotated tags
git tag -a <tagname> -m "<annotated-tag-message>"   stored also in .git/objects
cat .git/refs/tags/<tagname>                        hash-id
git cat-file -t <hash>                              'tag'
ls .git/objects
git tag -v <tagname>                                tag info     
    OR git cat-file -p <hash>                       the same

git push -v --tags                                  tags will be in Release section > Tags tab
git push -v origin <tagname>                        push only specified tag       

## cherry-pick
git checkout "<branchname-2>"
make changes
git commit -a -m "<commit-message>"
git log
copy commit-hash-####
git checkout "<branchname-1>"
git cherry-pick <commit-hash-####>                  brand new commit

git cherry-pick --no-commit <commit-hash-####>      move changes only to staged area
git status -v                                       show changes

## stash
uncommitted changes in <work-branch>
git stash
    cat .git/refs/stash
    git cat-file -p <hash-###>                      temp commit details
git checkout <branchname>                           allowed w/o loosing data
git checkout <work-branch>                          go back
git stash pop                                       applied stored changes in  stash
                                                    git will delete temporarily stored commit

## rebase
git checkout <rebased-branchname>
git rebase <destination-branchname>                 rebased on the top of <destination-branchname>
git checkout <destination-branchname>
git merge <rebased-branchname>                      actually fast-forward merge

### rebase + squash (interactive)
git log                                             copy <last-commit-hash> that will be starting point for squash
git checkout -b <branchname>
touch file1.txt
    git add .
    git commit -m "file1 in  <branchname>"
touch file2.txt
    git add .
    git commit -m "file2 in  <branchname>"
git log
git rebase -i <last-commit-hash>                    output like this
                                                        pick <hash-####> file1 in <branchname>
                                                        pick <hash-####> file2 in <branchname>
    i                                               replace `pick` w/ `s` for squashing
        s #### file2 in <branchname>
        ESC + :wq
    git log                                         there will be single commit instead of two
git checkout master
    git merge -v <branchname>                       = rebase + squash                    