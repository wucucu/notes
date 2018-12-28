TIPS
----

# git

  
## username/email 

### the user/email showed in the github
try to config git that be able to push from local terminal and the user/email was showing as wucucu/~
- ssh staff: https://help.github.com/articles/connecting-to-github-with-ssh/
    - as ~/.ssh/is_rsa.pub already exits, add it to github profile ssh setting
- user/email staff: https://help.github.com/articles/connecting-to-github-with-ssh/
    - verify local computer git config user.email information in the github profile
    
### Using different SSH keys for multiple Bitbucket accounts
(https://developer.atlassian.com/blog/2016/04/different-ssh-keys-multiple-bitbucket-accounts/)

### specify different username/email for each repo
(https://stackoverflow.com/questions/4220416/can-i-specify-multiple-users-for-myself-in-gitconfig)

### force to set username/email manually
Version ≥ 2.8.0, force to set username and password for a initial git repo
```
[user]
    useconfigonly = true
```
or
```sh
git config --global user.userconfigonly true
```

then in first commit it shows
```
fatal: user.useConfigOnly set but no name given
```

## remote server ssh 
create multiple identities for Mac OSX, GitBash, and Linux
(https://stackoverflow.com/questions/21139926/how-to-maintain-multiple-bitbucket-accounts-with-multiple-ssh-keys-in-the-same-s)

This is useful when you want to set up ssh for different accounts from a same remote service provider, like, bitbucket.org.

Notice, after setup, we need to change the server name in the config setting remote.origin.url, refer to the last part of the first anwser.

## remote branch
 create a remote branch from a local branch or vice versa
(https://stackoverflow.com/questions/11266478/git-add-remote-branch)

 remove remote/local branch
 ```
 $ git push -d origin <branch_name>
 $ git branch -d <branch_name>
 ```


# linux shell
## `find` usage
(https://alvinalexander.com/unix/edu/examples/find.shtml)


### basics
```bash
find . -name "foo.*" 
```

### remove all .bak files from a project
(https://askubuntu.com/questions/377438/how-can-i-recursively-delete-all-files-of-a-specific-extension-in-the-current-di)
```bash
find . -name "*.bak" -type f
find . -name "*.bak" -type f -delete
```

## echo file
### cat multiple files and print filename as header
(https://stackoverflow.com/questions/5917413/cat-multiple-files-but-include-filename-as-headers)




