TIPS
----

# git username and email
specify different username/email for each repo
(https://stackoverflow.com/questions/4220416/can-i-specify-multiple-users-for-myself-in-gitconfig)

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

# git remote server ssh 
create multiple identities for Mac OSX, GitBash, and Linux
(https://stackoverflow.com/questions/21139926/how-to-maintain-multiple-bitbucket-accounts-with-multiple-ssh-keys-in-the-same-s)

This is useful when you want to set up ssh for different accounts from a same remote service provider, like, bitbucket.org.

Notice, after setup, we need to change the server name in the config setting remote.origin.url, refer to the last part of the first anwser.

# git remote branch
 create a remote branch from a local branch or vice versa
(https://stackoverflow.com/questions/11266478/git-add-remote-branch)
