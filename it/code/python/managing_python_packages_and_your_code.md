
# Managing Python, Packages, and your Code

Tags: python, environment, code, version, pyenv, virtual environment, environment, venv, pip, package, library, sdk, docker, setup, git

Assumes Skills: git, terminal


## Summary

Python is a high level programming language renowned for its ease of use and readability, making it highly maintainable and apt for rapid prototyping.  The goal of this document is to explain and demonstrate a straight forward methodology to make your projects easy and reliable to manage.


## 1. Python Versions with PyEnv

There are many versions of python and no telling which you may need for a project.  Further, incorrectly installing incompatible versions or packages could damage your system and will likely cause unmanagable conflicts for your projects.  It is important to carefully manage your python and package versions *independently* of your system installation that your operating system depends on.

### 1.1  PyEnv

The standard for managing available versions of python is with <ins>PyEnv</ins>
PyEnv works by building and installing specified versions of python into a (typically user) directory.
The user may then call upon that version of python whenever needed.

i.  Install PyEnv.
  * Linux ( [Github / pyenv](https://github.com/pyenv/pyenv) )
  * Windows ( [Github / pyenv-win](https://github.com/pyenv-win/pyenv-win) )
  * Mac `brew install pyenv`

You'll likely want to add the pyenv installation directory to your PATH variable in your environment variables, either via .bashrc, .profile, or within system properties in Windows.
Take note of the pyenv installation directory, you may need to refer to it to access the pyenv command.

ii. PyEnv to list candidate versions of python to install:
`pyenv install --list`

iii. PyEnv to install a specific candidate.
`pyenv install <version>`
 
iv. Access the installed version.
* Linux / Mac `~/.pyenv/versions/version/<version>/bin/python --version`
* Windows ``%USERPROFILE%\.pyenv\pyenv-win\versions\<version>\Lib\python --version``

 🎉 Congrats!

## 2. Python Virtual Environments

Just as multiple versions of python may conflict, so will inevitably multiple packages or dependencies for your various projects.  Now, most python distributions come with a common module, "**venv**", for managing package environments, called "virtual environments".

A python virtual environment or *venv* is just a directory containing a stubbed reference to python where packages may be managed, independently of the system and other projects.  It will contain the packages and scripts for your project.  It is common to name the virtual environment "venv" in the root of your project directory.

### 2.1 Create a Virtual Environment for your project.

a. Navigate to your project directory.
* Linux/Mac `cd ~/projects/<project>`
* Windows `cd C:\Users\\\<you\>\projects\\\<project>\`

b. Configure .gitignore to ignore your virtual environment.
`echo venv >> .gitignore`

c. Create a virtual environment using the earlier installed python version:
* Linux/Mac `~/.pyenv/versions/<version>/bin/python -m venv venv`
* Windows: ``%USERPROFILE%\.pyenv\pyenv-win\versions\<version>\Lib\python -m venv venv``

### 2.2 Using the Virtual Environment.
From within the virtual environment, you may access python functionality as normal:
<ins>From here on, you will need to refer to windows in a venv as **`venv\Lib\python`**</ins>
```
venv/bin/python --version
venv/bin/pip --version
```
Optional:  You may "activate" the virtual environment in your shell such that venv/bin is added to your PATH environment variable, with: `source venv/bin/activate`
You may `deactivate` the virtual environment at any time, or simply exit the shell.

## 3. Installing Packages

Now that we've configured our environment with, we may install the packages we require.  Python's Pip is the official standard for managing python packages for projects.  It is capable of downloading, compiling, and installing python packages and accompanying scripts.

### 3.1 Configure Pip Repository (optional, common for organizations)
The typical method of providing python packages for a community is with a PyPi package repository.  Pip will refer to an internet default, however many organization require strict isolation in production such that you may only install packages from an internal repository.

You may configure your project to use a specific PyPi package repository with a `pip.conf` file:
```
[global]
index-url = https://example.org/
```

### 3.2 Pip Install Packages.

i. Pip install specific packages.
`venv/bin/pip install requests`

ii. Pip install packages from a requirements file. 
`venv/bin/pip install -r requirements.txt`

iii. Pip install from editable project (package) for easy development.
`venv/bin/pip install --editable .`
This will only work if your project is prepared as an installable package.

### 3.3 Pip List Installed Packages.
`venv/bin/pip freeze`

Example output
```
certifi==2022.12.7
charset-normalizer==3.0.1
idna==3.4
requests==2.28.2
urllib3==1.26.14
```

This can also be used to create a requirements.txt file!
`venv/bin/pip freeze > requirements.txt`


## 4. Packaging your Code

### 4.1 Prepare your Project Directory Tree

A common organizational structure is:
```
HelloWorld/
  src/
    HelloWorld/
      __main__.py
  doc/
  .gitignore
  setup.py
```

* #### [Namespace Packages](https://packaging.python.org/en/latest/guides/packaging-namespace-packages/)

    If you intend to release many packages, it is often suggested to use a namespace package, as to not litter the global package namespace.  This means that your package will be accessible as `tag.name`

    So you, as a user might have:  `user.package1`, `user.package2` packages.
If you intend to use namespace packages, I would suggest a directory tree like:
Notice the addition of the user directory.
    ```
    HelloWorld/
      src/
        user/
          HelloWorld/
            __main__.py
      doc/
      .gitignore
      setup.py
    ```
### Prepare the .gitignore file.
This will ignore files and directories to be uploaded to git.
```
.venv
*.pyc
```

### Prepare your setup.py file.

This example file shows how to manage several features:
* Versions for dependencies.
* "Extras" which install extra dependencies for various use cases, like [dev,cli,nvidia,amd,cpu]
* Entry Points, shell commands or script to make available.

```python
from setuptools import setup, find_packages

setup(
    name="user.helloworld",
    version="1.2.3",
    package_dir={"": "src"},
    packages=find_packages("src"),
    url="https://example.com/project",
    license="",
    author="",
    author_email="",
    install_requires=[
        "requests >= 2.19.1, <3.0.0",
    ],
    extras_require={
        "dev": [
            "flake8 >= 5.0.4, <6.0.0",
        ],
        "cli": [
            "click >= 8.1.3, <9.0.0",
        ],
    },
    entry_points={
        "console_scripts": [
            # my_script will run main() from user/helloworld/__main__.py
            "my_script=user.helloworld.__main__:main":
        ],
    },
)
```

## 5. Dockerizing your Project.

### 5.1 Prepare .dockerignore
When you build docker, this prevents these files from being prepared in the docker context.
(they'll only be used if you include them in the final image, but still avoid large files)
```
.venv
*.pyc
```

### 5.2 Prepare Dockerfile

```Dockerfile
FROM python:version
# FROM python@sha:hash

# copy build context to /app
COPY . /app
RUN python -m venv vev
# add venv to PATH
ENV PATH="/app/venv/bin:${PATH}"
# if your code is an installable package
# RUN pip install -e .
# CMD python -m package

# default run command
CMD python -u main.py
```

## 6. Standardization

### 6.1 PEP8 - https://peps.python.org/pep-0008

Python has several defined standards for writing code.

#### Flake 8 to Lint your Code
You can use flake8 to help you maintain sylistic standards and spot code smells early.
```
venv/bin/pip install flake8
venv/bin/flake8 src/
```
