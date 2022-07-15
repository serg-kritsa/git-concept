
https://github.com/git-user/git-repo/tree/master/folder-in-repo




26

Whoever is working on specific folder he needs to clone that particular folder itself, to do so please follow below steps by using sparse checkout.

Create a directory.

Initialize a Git repository. (git init)

Enable Sparse Checkouts. (git config core.sparsecheckout true)

Tell Git which directories you want (echo 2015/brand/May( refer to folder you want to work on) >> .git/info/sparse-checkout)

Add the remote (git remote add -f origin https://jafartke.com/mkt-imdev/DVM.git)

Fetch the files (git pull origin master