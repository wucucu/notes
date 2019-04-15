TIPS

- [1. git](#1-git)
  - [1.1. username/email](#11-usernameemail)
    - [1.1.1. the user/email showed in the github](#111-the-useremail-showed-in-the-github)
    - [1.1.2. Using different SSH keys for multiple Bitbucket accounts](#112-using-different-ssh-keys-for-multiple-bitbucket-accounts)
    - [1.1.3. specify different username/email for each repo](#113-specify-different-usernameemail-for-each-repo)
    - [1.1.4. force to set username/email manually](#114-force-to-set-usernameemail-manually)
  - [1.2. remote server ssh](#12-remote-server-ssh)
  - [1.3. remote branch](#13-remote-branch)
  - [1.4. untrack and clean ignored files](#14-untrack-and-clean-ignored-files)
- [2. linux shell](#2-linux-shell)
  - [2.1. `find` usage](#21-find-usage)
    - [2.1.1. basics](#211-basics)
    - [2.1.2. remove all .bak files from a project](#212-remove-all-bak-files-from-a-project)
  - [2.2. echo file](#22-echo-file)
    - [2.2.1. `cat` multiple files and print filename as header](#221-cat-multiple-files-and-print-filename-as-header)
- [3. jupyter notebook](#3-jupyter-notebook)
  - [3.1. `autoreload`](#31-autoreload)
- [4. Django](#4-django)
  - [4.1. run python script in Django shell](#41-run-python-script-in-django-shell)
- [5. PostgreSQL](#5-postgresql)
  - [5.1. where date condition](#51-where-date-condition)

# 1. git

## 1.1. username/email

### 1.1.1. the user/email showed in the github

try to config git that be able to push from local terminal and the user/email
was showing as wucucu/~

- ssh staff: https://help.github.com/articles/connecting-to-github-with-ssh/
  - as ~/.ssh/is_rsa.pub already exits, add it to github profile ssh setting
- user/email staff: https://help.github.com/articles/connecting-to-github-with-ssh/
  - verify local computer git config user.email information in the github profile

### 1.1.2. Using different SSH keys for multiple Bitbucket accounts

(https://developer.atlassian.com/blog/2016/04/different-ssh-keys-multiple-bitbucket-accounts/)

### 1.1.3. specify different username/email for each repo

(https://stackoverflow.com/questions/4220416/can-i-specify-multiple-users-for-myself-in-gitconfig)

### 1.1.4. force to set username/email manually

Version ≥ 2.8.0, force to set username and password for a initial git repo

```sh
[user]
    useconfigonly = true
```

or

```sh
git config --global user.userconfigonly true
```

then in first commit it shows

```sh
fatal: user.useConfigOnly set but no name given
```

## 1.2. remote server ssh

create multiple identities for Mac OSX, GitBash, and Linux
(https://stackoverflow.com/questions/21139926/how-to-maintain-multiple-bitbucket-accounts-with-multiple-ssh-keys-in-the-same-s)

This is useful when you want to set up ssh for different accounts from a same
remote service provider, like, bitbucket.org.

Notice, after setup, we need to change the server name in the config setting
remote.origin.url, refer to the last part of the first answer.

## 1.3. remote branch

create a remote branch from a local branch or vice versa
(https://stackoverflow.com/questions/11266478/git-add-remote-branch)

 remove remote/local branch

```sh
$ git push -d origin <branch_name>
$ git branch -d <branch_name>
```

## 1.4. untrack and clean ignored files

Untrack files already added to git repository based on .gitignore
(http://www.codeblocq.com/2016/01/Untrack-files-already-added-to-git-repository-based-on-gitignore/)

- commit all changes
- remove everything from the repo, `git rm -r .`
- readd everything, `git add .`
- commit, `git commit -m "Clean .gitignore untracked files."`

# 2. linux shell

## 2.1. `find` usage

(https://alvinalexander.com/unix/edu/examples/find.shtml)

### 2.1.1. basics

```bash
find . -name "foo.*"
```

### 2.1.2. remove all .bak files from a project

(https://askubuntu.com/questions/377438/how-can-i-recursively-delete-all-files-of-a-specific-extension-in-the-current-di)

```bash
find . -name "*.bak" -type f
find . -name "*.bak" -type f -delete
```

## 2.2. echo file

### 2.2.1. `cat` multiple files and print filename as header

(https://stackoverflow.com/questions/5917413/cat-multiple-files-but-include-filename-as-headers)

# 3. jupyter notebook

## 3.1. `autoreload`

(https://ipython.org/ipython-doc/3/config/extensions/autoreload.html)
IPython extension to reload modules before executing user code.
autoreload reloads modules automatically before entering the execution of code
typed at the IPython prompt.

```python
In [1]: %load_ext autoreload

In [2]: %autoreload 2

In [3]: from foo import some_function

In [4]: some_function()
Out[4]: 42

In [5]: # open foo.py in an editor and change some_function to return 43

In [6]: some_function()
Out[6]: 43
```

# 4. Django

## 4.1. run python script in Django shell

(https://stackoverflow.com/questions/16853649/how-to-execute-a-python-script-from-the-django-shell)

# 5. PostgreSQL

## 5.1. where date condition

```sql
SELECT * FROM t where d BETWEEN '2019-01-01' AND CURRENT_DATE;
```