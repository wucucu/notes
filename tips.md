TIPS
----

# git username and password
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

# git Create multiple identities for Mac OSX, GitBash, and Linux
(https://stackoverflow.com/questions/21139926/how-to-maintain-multiple-bitbucket-accounts-with-multiple-ssh-keys-in-the-same-s)

This is useful when you want to set up ssh for different accounts from a same remote service provider, like, bitbucket.org.
