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
