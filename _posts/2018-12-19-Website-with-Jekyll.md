---
layout: post
title: Website with Jekyll
---
This website is genereted using Jekyll. The [Step-By-Step-How-Two](https://jekyllrb.com/docs/step-by-step/01-setup/) is great but I missed some information which I documented here.

## Preperation

### Ruby

At first install [Ruby](https://www.ruby-lang.org/en/). On Arch-Linux: 

``` bash
sudo pacman -S ruby
```

### Jekyll

The following command will install [Jekyll](https://jekyllrb.com/) in your user directory (*~/.gem/ruby/*).

``` bash
gem install --user-install jekyll bundler
```

You probably want to add the ruby bin folder to your search path by adding following line to your "run commands" file of choice. (*~/.bashrc*, *~/.zshrc*, ...).

``` bash
export PATH=$PATH:~/.gem/ruby/2.5.0/bin
```
Adapt the version of ruby accordingly to your installed version.

## Create a new Website

Now you are ready to start working on your website. I would suggest to instruct Jekyll to create a dummy website for you with:

``` bash
jekyll new <path_to_project_folder>
```
This has the advantage that jekyll will create a base directory structure and most importand create some config files for you.

Unfortunately this call fails on my system, because Jekyll tries to install some additional ruby gems and complains about missing super user rights. I decided to install it in the user directory and not on the system level. Therefore I installed the missing gems by hand:

``` bash
gem install minima jekyll-feed
```

The following command builds the webiste under *\<path_to_project_folder>/_site*:

``` bash
cd <path_to_project_folder>
jekyll build
```

Jekyll has an build in webserver which can be started with:

``` bash
cd <path_to_project_folder>
jekyll serve
```

The website can be accessed via [http://localhost:4000](http://localhost:4000).
If one of the source files in *\<path_to_project_folder>* changes, then the webserver immediately updates the website in *\<path_to_project_folder>/_site*:

## Writing and Configuring a Website

A good step by step tutorial to write a website is provided on the [Jekyll Website](https://jekyllrb.com/docs/step-by-step/01-setup/). The next sections cover some additional topics not described in the step by step tutorial.

### Front Matter

Allways inlude a Front Matter even if you don't have to declare anything. Otherwise the Liquid statements in the documents will not be evaluated.

### Syntax Highlighting

In Markdown you can highlight code by fence it with three back-ticks.

~~~
```c
int main(int argc, char **argv);
```

```bash
export HELLO="Hello world!";
echo $HELLO;
```
~~~

The default highlighter of Jekyll is [Rouge](http://rouge.jneen.net/) which is compatible with [Pygments](http://pygments.org/). Rouge enrich the code with additional HTML tags and corresponding CSS classes. [Pygmentize](http://pygments.org/docs/cmdline/) can be used to generate according CSS definitions.
```bash
sudo pacman -S pygmentize
```
Use Pygmantize to create a style which is called "vim":

```bash
pygmentize -S vim -f html > syntax.css 
```

Alternatively you can download the [css files](https://github.com/richleland/pygments-css).

### Automatic Deployment using Git
The idea is, that a *git push* to a remote automaticly deploys the website to the http server. [Git hooks](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks) allows to execute custom skripts on events. I use the *post-receive* hook which is executed after a push to the remote.
Therefore i create a non-bare git repository on the webserver.
```bash
cd /path/to/local/repository/
git init
``` 
To prevent inconsistencies, it is not allowed to push commits to a non-bare repository by default. A git push only updates the git index and not the checked out files. Since I don't intent to develop within this repository, the checked out files shall be automatically synced with updated index. 

To allow pushing to this non-bare remote, follwing lines have to be added to */path/to/local/repository/.git/config*:
```
[receive]
	denyCurrentBranch = ignore
```
The *post-receive* hook is created by creating the the a script-file */path/to/local/repository/.git/hooks/post-receive* and make it executable.
```bash
touch /path/to/local/repository/.git/hooks/post-receive
chmod u+x /path/to/local/repository/.git/hooks/post-receive
```
My hook skript has following content:
```bash
#!/bin/bash
unset GIT_DIR
DEPLOY_DIR=/path/to/deploy/directory/
LOCAL=/path/to/local/repository/

export PATH=$PATH:~/.gem/ruby/2.5.0/bin

git -C ${LOCAL} reset --hard
jekyll build -s ${LOCAL} -d ${DEPLOY_DIR}
```
Git sets the environment valriable GIT_DIR before executeing hooks. This leads to a strange behavior of git, which i don't understand and couldn't fix. Thereforefore i unset it first.
The checked out files will by synced with *git reset \-\-hard*. After that jekyll is called to update the website in */path/to/deploy/direcoty/*. Since jekyll is installed in the user directory, it is necessary to update the search path with *export ...*
