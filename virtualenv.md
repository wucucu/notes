Create Isolated Python Environments for Package Installation
====

author: wcc

## Use `pip` for Installing Python Packages to Different Python Interpreter
There are 3 different python interpreters in my system. Python 2.7 is the default python interpreter linked to `python` in the terminal. I forget whether it was builtin the system or installed along with Xcode. Notice that Python 2.6 is also there. Meanwhile Python 3 is linked to `python3`. 

```shell
$ python --version
Python 2.7.10
$ python2.6 --version
Python 2.6.9
$ python3 --version
Python 3.5.1
```

`pip` and `pip3` command is used to install packages for default Python and Python 3 respectively. It is set automatically when I used the python [installer][refInstaller] for Mac OS X.

```shell
$ which pip
/usr/local/bin/pip
$ which pip3
/usr/local/bin/pip3
$ ls -l /usr/local/bin/pip*
-rwxr-xr-x  1 root  wheel  281 Jun  6 10:34 /usr/local/bin/pip
-rwxr-xr-x  1 root  wheel  283 Jun  6 10:34 /usr/local/bin/pip2
-rwxr-xr-x  1 root  wheel  287 Jun  6 10:34 /usr/local/bin/pip2.7
lrwxrwxr-x  1 root  admin   66 Feb 14 06:29 /usr/local/bin/pip3 -> ../../../Library/Frameworks/Python.framework/Versions/3.5/bin/pip3
lrwxrwxr-x  1 root  admin   68 Feb 14 06:29 /usr/local/bin/pip3.5 -> ../../../Library/Frameworks/Python.framework/Versions/3.5/bin/pip3.5
```

## Use the Package Installed to Python 3 by `pip3 install`
I tried to solve the problem raising this article is to install the virtual python environment builder [Virtualenv][refVirtualenv]. I use `pip` to install the `virtualenv` package to Python 3.

```shell
$ pip3 install virtualenv
$ vertualenv
-bash: vertualenv: command not found
```

We need to add the original bin directory of `python3.5` to the environment variable `PATH` which contains the `vertualenv` program. I did this by editing the `~/.profile` which involves the user setting of the bash terminal.

```shell
$ which python3
/usr/local/bin/python3
$ ls -l /usr/local/bin/python3
lrwxr-xr-x  1 root  wheel  69 Feb 14 06:29 /usr/local/bin/python3 -> ../../../Library/Frameworks/Python.framework/Versions/3.5/bin/python3
```

Add following scripts to `~/.profile` and restart the terminal. Notice the path directory is same with the shown one after `->` in the above script.
```shell
# Setting PATH for Python 3.5
PATH="${PATH}:/Library/Frameworks/Python.framework/Versions/3.5/bin"
export PATH
```
## Raised Problem
[MySQLdb][refMySQL-python](`mysql-python`) dose not support Python 3 and above at the moment(12062016).
```shell
$ pip install mysql-python
Requirement already satisfied (use --upgrade to upgrade): mysql-python in /Library/Python/2.7/site-packages

$ pip3 install mysql-python
Collecting mysql-python
  Using cached MySQL-python-1.2.5.zip
    Complete output from command python setup.py egg_info:
    Traceback (most recent call last):
      File "<string>", line 1, in <module>
      File "/private/var/folders/n8/0fz1g82j7js5d5dnfq9gpr_40000gn/T/pip-build-3xksmiqo/mysql-python/setup.py", line 13, in <module>
        from setup_posix import get_config
      File "/private/var/folders/n8/0fz1g82j7js5d5dnfq9gpr_40000gn/T/pip-build-3xksmiqo/mysql-python/setup_posix.py", line 2, in <module>
        from ConfigParser import SafeConfigParser
    ImportError: No module named 'ConfigParser'
    
    ----------------------------------------
Command "python setup.py egg_info" failed with error code 1 in /private/var/folders/n8/0fz1g82j7js5d5dnfq9gpr_40000gn/T/pip-build-3xksmiqo/mysql-python/
```

At the first glance, I thought `mysql-python` installation failed due to serveral different Python versions existing in my system at the same time. The script `setup.py` was executed by `python`(my default python command linked to Python 2.7), while I thought it should be `python3`(the command used to run my Python 3.5). Furthermore, considering the `ImportError` of lacking module `ConfigParser`(`import` of python is case sensitive, [`ConfigParser` is Python 2.7 builtin module and has been changed to `configparser` since Python 3.0][refConfigParser]), I tried to find solutions for installing package for different Python Interpreter in one system.

## Use `virtualenv` to Sovle the Package Installing Problem in Multi-Installation Python Enviroment

http://stackoverflow.com/a/7237949
As same as `pip`, `virtualenv` is recommended for Python installation in [Python Packaging User Guide][refToolRecommendations].

```shell
$ virtualenv temp-python
Using base prefix '/Library/Frameworks/Python.framework/Versions/3.5'
New python executable in /Users/CKL/Documents/MATLAB/temp-python/bin/python3.5
Also creating executable in /Users/CKL/Documents/MATLAB/temp-python/bin/python
Installing setuptools, pip, wheel...done.
$ source temp-python/bin/activate
(temp-python)$ python --version
3.5.1
(temp-python)$ which python
temp-python/bin/python
(temp-python)$ ls temp-python/bin
activate    easy_install-3.5  python-config
activate.csh    pip     python3
activate.fish   pip3      python3.5
activate_this.py  pip3.5      wheel
easy_install    python
(temp-python)$ pip install mysql-python
Collecting mysql-python
  Using cached MySQL-python-1.2.5.zip
    Complete output from command python setup.py egg_info:
    Traceback (most recent call last):
      File "<string>", line 1, in <module>
      File "/private/var/folders/n8/0fz1g82j7js5d5dnfq9gpr_40000gn/T/pip-build-qckpikbn/mysql-python/setup.py", line 13, in <module>
        from setup_posix import get_config
      File "/private/var/folders/n8/0fz1g82j7js5d5dnfq9gpr_40000gn/T/pip-build-qckpikbn/mysql-python/setup_posix.py", line 2, in <module>
        from ConfigParser import SafeConfigParser
    ImportError: No module named 'ConfigParser'
    
    ----------------------------------------
Command "python setup.py egg_info" failed with error code 1 in /private/var/folders/n8/0fz1g82j7js5d5dnfq9gpr_40000gn/T/pip-build-qckpikbn/mysql-python/
(temp-python)$ deactivate
$
```
`virtualenv temp-python` will make a folder `temp-python` in the current directory. The folder will involve the virtual enviroment for the specific Python Interpreter. The default interpreter is the one `virtualenv` installed with. Use `virtualenv -h` to check details.

## Conclusion
some achievements
* bash terminal setting executed order (/etc/profle ~/.profile /etc/bashrc)
http://www.cyberciti.biz/faq/unix-linux-adding-path/
* same command names found in several $PATH directories(the very first one will be executed)
http://superuser.com/a/141445
* Python 2 tools to connect mysql: mysql-python torndb pymysql 
* Need to find useful tools of Python 3 to connect mysql


 [refConfigParser]: https://docs.python.org/2.7/library/configparser.html "ConfigParser Python 2.7 Documentation"
 [refMySQL-python]: https://pypi.python.org/pypi/MySQL-python/1.2.5 "MySQL-python 1.2.5 PyPI"
 [refPip]: https://pip.pypa.io/en/stable/ "pip Documentation"
 [refInstaller]: https://www.python.org/downloads/mac-osx/ "Python Releases for Mac OS X"
 [refVirtualenv]: https://virtualenv.pypa.io/en/stable/ "Virtualenv Documentation"
 [refToolRecommendations]: https://packaging.python.org/en/latest/current/ "Python Packaging and Installation Tool Recommendations"



